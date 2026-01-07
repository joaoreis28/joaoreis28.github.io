---
title: Aproximação de imagens usando triangulações e interpolação
date: 2025-11-03 09:53:00 -0300
categories: [Geometria Computacional]
tags: [geometria, otimização]     # TAG names should always be lowercase
description: Descrição do trabalho de divisão planar .
math: true
image: /assets/images/trab2.png
---

## Introdução 

Neste projeto da disciplina de Geometria Computacional, explorei uma técnica  de processamento de imagens: a representação e aproximação de imagens digitais através de divisões planares (malhas de triângulos).

Normalmente, pensamos em imagens como matrizes densas de pixels (raster), onde uma imagem de 512×512 contém mais de 260 mil pontos de informação. O desafio proposto neste trabalho foi subverter essa lógica: é possível reconstruir essa mesma imagem visualmente utilizando apenas uma fração desses pontos?

O objetivo principal foi desenvolver um algoritmo capaz de receber uma imagem de entrada e um número máximo de pontos , e gerar como saída uma imagem aproximada formada pela triangulação desses pontos. Para medir o sucesso da aproximação, utilizei a métrica RMSE (Root Mean Square Error), que quantifica a diferença entre a imagem original e a versão reconstruída.

### Conceito

A ideia central baseia-se em divisões planares. Dado uma imagem de $ n \times m $ pixels, podemos escolher um conjunto estratégico de pixels $$ P = (p_1, p_2, \dots, p_l) $$, $ l  \leq mn $,  da imagem original para servirem como vértices, dessa forma  criamos uma malha triangular. Cada vértice armazena não apenas sua posição $(x,y) $, mas também a intensidade de cor do pixel original.


Para preencher os espaços vazios (o interior dos triângulos), a cor de cada pixel é calculada através de uma interpolação linear das cores dos três vértices que compõem aquele triângulo. Isso permite criar gradientes suaves que simulam a imagem original, economizando drasticamente a quantidade de dados armazenados.


### Tecnologias Utilizadas

Para implementar essa solução utilizei:
- CGAL (Computational Geometry Algorithms Library), especificamente a estrutura de dados Halfedge (via classe Mesh), essencial para navegar na topologia da malha e realizar operações complexas de triangulação.

 - OpenCV para leitura e manipulação básica dos arquivos de imagem.





## Metodologia

### Seleção de Pontos: Operador de Sobel

O primeiro desafio do projeto foi decidir quais pontos da imagem seriam escolhidos como vértices da triangulação. Ao invés de uma seleção aleatória ou uniforme, optei por uma abordagem probabilística baseada em gradientes de intensidade usando o operador de Sobel.

A intuição por trás dessa escolha é simples: regiões da imagem com mudanças bruscas de cor (bordas, contornos) carregam mais informação visual e devem ser priorizadas. Por outro lado, áreas homogêneas (como céus ou fundos uniformes) podem ser representadas com poucos pontos sem perda significativa de qualidade.

#### Implementação

Primeiro, converto a imagem para escala de cinza e aplico o operador de Sobel nas direções $x$ e $y$ para calcular os gradientes:

```c++
cv::Mat gray, grad_x, grad_y, grad_magnitude;
cv::cvtColor(img_src, gray, cv::COLOR_BGR2GRAY);
cv::Sobel(gray, grad_x, CV_32F, 1, 0, 3);
cv::Sobel(gray, grad_y, CV_32F, 0, 1, 3);
cv::magnitude(grad_x, grad_y, grad_magnitude);
cv::normalize(grad_magnitude, grad_magnitude, 0.0, 1.0, cv::NORM_MINMAX);
```

O operador de Sobel calcula a magnitude do gradiente em cada pixel através da fórmula:

$\nabla I = \sqrt{G_x^2 + G_y^2}$

onde $G_x$ e $G_y$ são as derivadas parciais nas direções horizontal e vertical.

#### Amostragem Probabilística

Diferentemente de uma abordagem determinística que simplesmente seleciona os $l$ pixels com maiores gradientes, implementei uma estratégia probabilística que combina importância local com aleatoriedade:

```cpp
float base_prob = 0.02f; 

while (count < n_points) {
    int r = std::rand() % img_src.rows;
    int c = std::rand() % img_src.cols;
    
    float importance = grad_magnitude.at<float>(r, c);
    float probability = importance + base_prob;
    float dice = (float)std::rand() / RAND_MAX;

    if (dice < probability) {
        d.add_point(Point(r, c));
        count++;
    }
}
```

Cada pixel tem uma probabilidade de ser selecionado proporcional ao seu gradiente mais uma probabilidade base. Isso garante que:
- Pixels em regiões de alta variação (bordas) têm maior chance de serem selecionados
- Regiões homogêneas ainda podem ter alguns pontos representativos
- A distribuição resultante é mais natural e menos concentrada

Como ponto de comparação, também implementei um modo de seleção puramente aleatória (`mode = 0`), que permite avaliar o impacto da estratégia baseada em gradientes na qualidade final.

### Triangulação de Delaunay Incremental

Com os pontos selecionados, o próximo passo foi construir a triangulação. Escolhi o algoritmo incremental de Delaunay, que adiciona os pontos um a um à triangulação, mantendo sempre a propriedade de Delaunay.

A triangulação de Delaunay possui uma propriedade importante: ela maximiza o menor ângulo de todos os triângulos, evitando triângulos muito "achatados". Isso resulta em interpolações mais suaves e naturais na imagem final.

#### Inicialização da Malha

O algoritmo começa com uma triangulação inicial que cobre toda a área da imagem. Criei dois triângulos formando um retângulo com os quatro cantos da imagem:

```cpp
Delaunay::Delaunay(int img_w, int img_h) {
    Vertex p1 = mesh.add_vertex(Point(0, 0));
    Vertex p2 = mesh.add_vertex(Point(0, img_h - 1));
    Vertex p3 = mesh.add_vertex(Point(img_w - 1, img_h - 1));
    Vertex p4 = mesh.add_vertex(Point(img_w - 1, 0));
    mesh.add_face(p1, p2, p3);
    mesh.add_face(p1, p3, p4);
}
```

Essa inicialização garante que todos os pontos subsequentes estarão dentro da triangulação existente, simplificando a lógica de inserção.

#### Algoritmo de Inserção

Para cada novo ponto, o algoritmo executa três etapas principais:

**1. Localização:** Determino onde o ponto cai na triangulação atual (dentro de uma face, sobre uma aresta, ou coincidente com um vértice)

**2. Inserção:** Modifico a topologia da malha para incluir o novo ponto usando operações de Euler da CGAL:
   - Se o ponto está **dentro de uma face**: divido o triângulo em três usando `CGAL::Euler::add_center_vertex()`
   - Se o ponto está **sobre uma aresta**: divido a aresta e os dois triângulos adjacentes usando `CGAL::Euler::split_edge()` e `CGAL::Euler::split_face()`

**3. Restauração da propriedade de Delaunay:** Verifico e corrijo violações através do algoritmo de flip de arestas

```cpp
void Delaunay::add_point(Point p) {
    Location l = locate(p);
    switch (l.t) {
        case Location::Type::FACE:
            add_point_to_face(p, l.face);
            break;
        case Location::Type::LINE:
            add_point_to_edge(p, l.halfedge);
            break;
        case Location::Type::VERTEX:
            return; // Ponto já existe
        case Location::Type::OUTSIDE:
            return; // Ponto fora da malha
    }
}
```

#### Flip de Arestas e Propriedade de Delaunay

A propriedade de Delaunay é verificada através do **teste do círculo circunscrito**: para cada aresta compartilhada por dois triângulos, o quarto vértice (oposto à aresta) não deve estar dentro do círculo que passa pelos três vértices do primeiro triângulo.

```cpp
void Delaunay::flip(Halfedge he) {
    if (mesh.is_border(he) || mesh.is_border(mesh.opposite(he))) return;
    
    Point p1 = mesh.point(mesh.target(he));
    Point p2 = mesh.point(mesh.target(mesh.next(he)));
    Point p3 = mesh.point(mesh.source(he));
    Point p4 = mesh.point(mesh.target(mesh.next(mesh.opposite(he))));

    if (!is_convex_quadrilateral(p1, p2, p3, p4)) return;
    
    if (CGAL::side_of_oriented_circle(p1, p3, p4, p2) == CGAL::ON_NEGATIVE_SIDE) {
        CGAL::Euler::flip_edge(he, mesh);
        // Recursivamente verifica as arestas afetadas
        flip(h1); flip(h2); flip(h3); flip(h4);
    }
}
```

O CGAL fornece predicados geométricos robustos como `side_of_oriented_circle()` que garantem decisões corretas mesmo em casos degenerados. Quando uma violação é detectada, a aresta é "flipada" (invertida no quadrilátero) usando `CGAL::Euler::flip_edge()`, e o processo continua recursivamente nas arestas afetadas.

### Localização de Pontos

Um aspecto crucial do algoritmo incremental é determinar rapidamente em qual triângulo cada novo ponto se encontra. Implementei uma **busca por caminhada (walking search)** que navega pela malha até encontrar a face correta.

#### Algoritmo de Caminhada

O algoritmo começa de uma halfedge qualquer e "caminha" pela malha seguindo uma regra simples: se o ponto está à esquerda de uma aresta (fora da face atual), move-se para a face vizinha através daquela aresta.

```cpp
Location Delaunay::locate(Point p) {
    Halfedge h = *mesh.halfedges_begin();
    if (mesh.is_border(h)) {
        h = mesh.opposite(h);
    }

    int path_size = 0;
    while (path_size < mesh.num_halfedges()) {
        path_size++;
        Halfedge h_start = h;
        bool found_face = true;

        // Verifica as 3 arestas da face atual
        for (int i = 0; i < 3; i++) {
            Point p1 = mesh.point(mesh.source(h));
            Point p2 = mesh.point(mesh.target(h));

            if (p == p1) return Location(mesh.source(h));
            if (p == p2) return Location(mesh.target(h));
            if (is_in_line(p1, p2, p)) return Location(h);

            // Faces são orientadas no sentido horário
            // Ponto interno está à DIREITA das arestas
            // Cruza para face oposta quando ponto está à ESQUERDA
            if (CGAL::left_turn(p1, p2, p)) {
                Halfedge opp = mesh.opposite(h);
                if (mesh.is_border(opp)) {
                    return Location();  // Fora da malha
                }
                h = mesh.next(opp);
                found_face = false;
                break;
            }

            h = mesh.next(h);
        }

        if (found_face) {
            return Location(mesh.face(h_start));
        }
    }

    return Location();
}
```

#### Estrutura de Localização

A função retorna uma estrutura `Location` que encapsula quatro possíveis resultados:

```cpp
struct Location {
    enum Type { FACE, LINE, VERTEX, OUTSIDE };
    Type t;
    Face face;
    Halfedge halfedge;
    Vertex vertex;
};
```

Essa estrutura permite que o algoritmo de inserção saiba exatamente como proceder em cada caso.

#### Testes Geométricos

Para determinar a posição relativa do ponto, uso predicados geométricos robustos da CGAL:

- **`CGAL::left_turn(p1, p2, p)`**: Verifica se `p` está à esquerda do segmento orientado de `p1` para `p2`
- **`CGAL::collinear(p1, p2, p)`**: Verifica se os três pontos são colineares

A função auxiliar `is_in_line()` combina colinearidade com um teste de intervalo para verificar se o ponto está sobre o segmento:

```cpp
bool is_in_line(Point p1, Point p2, Point p) {
    if (!CGAL::collinear(p1, p2, p)) return false;
    
    auto between = [](int a, int b, int c) -> bool {
        return c > std::min(a, b) && c < std::max(a, b);
    };

    if (p1.x() != p2.x())
        return between(p1.x(), p2.x(), p.x());
    else
        return between(p1.y(), p2.y(), p.y());
}
```


## Geração da Imagem Aproximada

### Interpolação Linear das Cores

Com a triangulação construída, cada pixel da imagem final precisa ter sua cor determinada. Para isso, implementei interpolação linear usando **coordenadas baricêntricas**.

A função `interpolate()` determina a cor de qualquer ponto $(x, y)$ localizando-o na triangulação e aplicando a interpolação apropriada:

```c++
cv::Scalar ImageCompression::interpolate(Point p) {
    triangulation::Location l = d.locate(p);
    
    switch (l.t) {
        case triangulation::Location::Type::VERTEX: {
            // Ponto coincide com um vértice - retorna a cor exata
            int x = std::min((int)p.x(), img_src.rows - 1);
            int y = std::min((int)p.y(), img_src.cols - 1);
            cv::Vec3b c = img_src.at<cv::Vec3b>(x, y);
            return cv::Scalar(c[0], c[1], c[2]);
        }
        case triangulation::Location::Type::LINE: {
            // Ponto sobre uma aresta - interpolação linear 1D
            auto [p1, p2] = d.points(l.halfedge);
            auto [w1, w2] = linear_interp_line(p1, p2, p);
            
            cv::Vec3b c1 = img_src.at<cv::Vec3b>((int)p1.x(), (int)p1.y());
            cv::Vec3b c2 = img_src.at<cv::Vec3b>((int)p2.x(), (int)p2.y());
            
            return cv::Scalar(w1*c1[0] + w2*c2[0],
                              w1*c1[1] + w2*c2[1],
                              w1*c1[2] + w2*c2[2]);
        }
        case triangulation::Location::Type::FACE: {
            // Ponto dentro de um triângulo - interpolação baricêntrica
            auto [p1, p2, p3] = d.points(l.face);
            auto [w1, w2, w3] = barycentric_coords(p1, p2, p3, p);
            
            cv::Vec3b c1 = img_src.at<cv::Vec3b>((int)p1.x(), (int)p1.y());
            cv::Vec3b c2 = img_src.at<cv::Vec3b>((int)p2.x(), (int)p2.y());
            cv::Vec3b c3 = img_src.at<cv::Vec3b>((int)p3.x(), (int)p3.y());
            
            return cv::Scalar(w1*c1[0] + w2*c2[0] + w3*c3[0],
                              w1*c1[1] + w2*c2[1] + w3*c3[1],
                              w1*c1[2] + w2*c2[2] + w3*c3[2]);
        }
        default: 
            return cv::Scalar(0, 0, 0);
    }
}
```

#### Coordenadas Baricêntricas

Para um ponto $p$ dentro do triângulo com vértices $v_1, v_2, v_3$ de cores $c_1, c_2, c_3$, a cor interpolada é:

$c(p) = \alpha \cdot c_1 + \beta \cdot c_2 + \gamma \cdot c_3$

onde $(\alpha, \beta, \gamma)$ são as coordenadas baricêntricas calculadas através de áreas:

$\alpha = \frac{\text{área}(p, v_2, v_3)}{\text{área}(v_1, v_2, v_3)}$

e similarmente para $\beta$ e $\gamma$. A implementação usa o produto vetorial para calcular áreas com sinal:

```c++
double triangle_double_area(Point p1, Point p2, Point p3) {
    return (p1.x() - p3.x()) * (p2.y() - p3.y()) - 
           (p2.x() - p3.x()) * (p1.y() - p3.y());
}

std::tuple<float, float, float> barycentric_coords(Point p1, Point p2, Point p3, Point p) {
    double area_total = triangle_double_area(p1, p2, p3);
    
    if (std::abs(area_total) < 1e-9) return {0.33f, 0.33f, 0.33f};

    double area1 = triangle_double_area(p, p2, p3);
    double w1 = area1 / area_total;
    
    double area2 = triangle_double_area(p1, p, p3);
    double w2 = area2 / area_total;
    
    double w3 = 1.0 - w1 - w2;
    
    return {(float)w1, (float)w2, (float)w3};
}
```

#### Interpolação Linear para Arestas

Para pontos sobre arestas, uso interpolação linear 1D baseada na distância:

```cpp
std::pair<float, float> linear_interp_line(Point p1, Point p2, Point p) {
    double dist_total = std::sqrt(std::pow(p1.x()-p2.x(), 2) + 
                                   std::pow(p1.y()-p2.y(), 2));
    double dist_p1 = std::sqrt(std::pow(p.x()-p1.x(), 2) + 
                                std::pow(p.y()-p1.y(), 2));
    
    if (dist_total < 1e-9) return {0.5f, 0.5f};
    
    float w2 = (float)(dist_p1 / dist_total);
    float w1 = 1.0f - w2;
    return {w1, w2};
}
```

A interpolação baricêntrica garante transições suaves de cor dentro de cada triângulo, criando gradientes naturais que aproximam a imagem original.

### Rasterização e Cálculo do RMSE

Para construir a imagem final, implementei uma abordagem direta: percorro cada pixel da imagem de saída, localizo-o na triangulação e aplico a interpolação correspondente.

```c++
void ImageCompression::run() {
    img_compression = cv::Mat(img_src.rows, img_src.cols, CV_8UC3, cv::Scalar(0, 0, 0));
    
    double sum_sq_diff = 0.0;
    long total_pixels = img_src.rows * img_src.cols;

    for (int i = 0; i < img_compression.rows; i++) {
        for (int j = 0; j < img_compression.cols; j++) {
            Point p(i, j);
            cv::Scalar color = interpolate(p);
            
            cv::Vec3b &pixel_out = img_compression.at<cv::Vec3b>(i, j);
            pixel_out = cv::Vec3b((uchar)color[0], (uchar)color[1], (uchar)color[2]);

            // Calcula diferença para RMSE
            cv::Vec3b pixel_in = img_src.at<cv::Vec3b>(i, j);
            double diffB = pixel_in[0] - pixel_out[0];
            double diffG = pixel_in[1] - pixel_out[1];
            double diffR = pixel_in[2] - pixel_out[2];
            
            sum_sq_diff += (diffB*diffB + diffG*diffG + diffR*diffR);
        }
    }

    double mse = sum_sq_diff / (total_pixels * 3.0);
    double rmse = std::sqrt(mse);

    std::cout << "\nRMSE: " << rmse << std::endl;
}
```

#### Abordagem de Rasterização

Diferentemente de algoritmos tradicionais de rasterização que percorrem apenas os pixels dentro de cada triângulo, optei por uma **abordagem de força bruta**: para cada pixel da imagem, localizo em qual triângulo ele está e aplico a interpolação.

Embora essa abordagem possa parecer ineficiente, ela possui vantagens práticas:

1. **Simplicidade de implementação**: Não preciso lidar com casos especiais de varredura de triângulos
2. **Garantia de cobertura completa**: Todo pixel é preenchido exatamente uma vez
3. **Facilita o cálculo do RMSE**: Posso comparar cada pixel durante a mesma iteração
4. **Robustez**: Evita problemas de buracos ou sobreposições comuns em rasterizadores tradicionais


#### Métrica RMSE

O RMSE (Root Mean Square Error) quantifica a diferença entre a imagem original e a aproximação:

$\text{RMSE} = \sqrt{\frac{1}{3nm} \sum_{i=1}^{n} \sum_{j=1}^{m} \sum_{c \in \{R,G,B\}} (I_{original}(i,j,c) - I_{aproximada}(i,j,c))^2}$

onde $n \times m$ é o tamanho da imagem e consideramos os três canais de cor (RGB). O fator 3 no denominador normaliza o erro pelos três canais.

Valores menores de RMSE indicam melhor aproximação. Na prática:
- **RMSE < 10**: Aproximação excelente, diferenças imperceptíveis
- **RMSE 10-20**: Boa aproximação, estruturas principais preservadas
- **RMSE 20-40**: Aproximação razoável, alguns detalhes perdidos
- **RMSE > 40**: Aproximação grosseira, perda significativa de qualidade

## Resultados

### Qualidade Visual


Testei o algoritmo com diferentes quantidades de pontos para avaliar a relação entre compressão e qualidade visual:

- **20000 pontos** : Captura apenas as estruturas mais grosseiras
- **50000 pontos** : Começa a revelar detalhes importantes
- **1500000 pontos** : Boa aproximação visual

-  Primeira Imagem 640 pixels X 480 pixels, total de pixels = 307200

![Desktop View](assets/images/hippo.jpg){: width="500" height="300" }
*Imagem Original.*

![Desktop View](assets/images/hippo20.png){: width="500" height="300" }
*Imagem Aproximada com $20.000$ pontos, aproximadamente $6.51 \% $ do total, RMSE = 14.2783.*

![Desktop View](assets/images/hippo50.png){: width="500" height="300" }
*Imagem Aproximada com 60.000 pontos, aproximadamente $19.53 \% $ do total, RMSE = 9.7359.*



![Desktop View](assets/images/hippo150.png){: width="500" height="300" }
*Imagem Aproximada com 150.000 pontos, aproximadamente $48.83 \% $ do total, RMSE = 6.6924.*


-  Segunda Imagem 500 pixels X 480 pixels, total de pixels = 240000


![Desktop View](assets/images/baboon.bmp){: width="500" height="300" }
*Imagem Original.*



![Desktop View](assets/images/babo20.png){: width="500" height="300" }
*Imagem Aproximada com 20000 pontos, aproximadamente $8.83 \% $ do total, RMSE = 27.8388.*


![Desktop View](assets/images/babo50.png){: width="500" height="300" }
*Imagem Aproximada com 50000 pontos, aproximadamente $20.83 \% $ do total, RMSE = 23.1637.*


![Desktop View](assets/images/babo150.png){: width="500" height="300" }
*Imagem Aproximada com 150000 pontos, aproximadamente $62.5 \% $ do total, RMSE = 16.9123.*



### Análise de Performance

Informações sobre o tempo de execução para diferentes números de pontos e a imagem do hipopótamo:
-   20.000 pontos : 138.864 segundos
-   50.000 pontos : 268.943 segundos
-   150.000 pontos : 429.391 segundos


Foi observado que o tempo de execução é relativamente longo devido ao método que a rasterização está sendo feita.
## Discussão

### Comparação: Seleção por Gradiente vs. Aleatória

A implementação permite comparar duas estratégias de seleção de pontos através do parâmetro `mode`:
- **Mode 0**: Seleção uniformemente aleatória
- **Mode 1**: Seleção probabilística baseada em gradientes de Sobel

A abordagem baseada em gradientes demonstrou ser significativamente mais eficaz para capturar os elementos visuais importantes da imagem. Observei que:

- **Imagens com bordas bem definidas** (como o baboon do exemplo) são aproximadas com alta qualidade mesmo com poucos pontos quando se usa detecção de gradientes
- A estratégia aleatória requer consideravelmente mais pontos para atingir qualidade visual equivalente
- **Regiões de gradiente suave** (céus, fundos uniformes) são naturalmente bem representadas pela interpolação linear, independente da estratégia de seleção

### Vantagens da Triangulação de Delaunay

A escolha da triangulação de Delaunay trouxe benefícios visíveis na qualidade final:

- Triângulos mais "equilibrados" (sem ângulos muito agudos) resultam em interpolações mais suaves
- Transições de cor são mais naturais que em triangulações arbitrárias
- A propriedade de maximizar ângulos mínimos evita artefatos visuais em forma de "raios"

### Estrutura Halfedge e Performance

A estrutura de dados Halfedge (Surface_mesh da CGAL) foi fundamental para a eficiência do algoritmo:

- **Navegação topológica**: Operações como `next()`, `opposite()`, `prev()` permitem percorrer a malha em tempo constante
- **Operações de Euler**: Modificações topológicas (split, flip) mantêm a consistência automática da estrutura
- **Memória eficiente**: Embora mais complexa que uma lista simples de triângulos, a Halfedge evita redundância e facilita consultas de vizinhança

### Limitações Observadas

- **Texturas complexas**: Em regiões com texturas muito detalhadas (como pelo do baboon), a interpolação linear não consegue capturar padrões de alta frequência
- **Performance da localização**: Para triangulações muito grandes (>50k pontos), a busca por caminhada pode se tornar um gargalo
- **Inicialização**: A estratégia de começar com os 4 cantos funciona bem, mas triângulos iniciais muito grandes podem afetar a performance das primeiras inserções

### Possíveis Melhorias

**Otimização da seleção de pontos:**
- Implementar seleção adaptativa que adiciona pontos dinamicamente em regiões com maior erro
- Usar detecção de características mais sofisticadas (Harris corners, SIFT)
- Considerar informação de cor, não apenas intensidade

**Otimização da localização:**
- Implementar estrutura hierárquica (DAG) para localização em $O(\log n)$
- Usar spatial hashing para acelerar a busca inicial

**Interpolação avançada:**
- Experimentar com interpolação de Hermite ou splines para capturar variações não-lineares
- Adicionar informação de gradiente nos vértices

**Qualidade vs. Performance:**
- Implementar rasterização tradicional por triângulo para ganho de velocidade, no momento a estratégia de rasterização utilizada é o maior gargalo da aplicação, levando a um tempo de execução elevado
- Usar paralelização (OpenMP) para processar múltiplos pixels simultaneamente

## Conclusão e Código Completo

Este projeto demonstrou que é possível aproximar imagens digitais com qualidade visual satisfatória usando apenas uma pequena fração dos pixels originais. A combinação de seleção baseada em gradientes, triangulação de Delaunay e interpolação linear resultou em um sistema eficiente e eficaz.

Além do resultado prático, o trabalho proporcionou uma compreensão profunda de estruturas de dados geométricas (Halfedge), algoritmos de triangulação e técnicas de processamento de imagens - conhecimentos fundamentais em geometria computacional.

Código Completo : [Ver Código no GitHub](https://github.com/joaoreis28/Trabalho-2-Geometria-Computacional){: target="_blank" }


## Referências

- CGAL Documentation: [Halfedge Data Structures](https://doc.cgal.org/latest/HalfedgeDS/index.html)
- CGAL Documentation: [Euler Operations](https://doc.cgal.org/latest/BGL/group__PkgBGLEulerOperations.html)
- Sobel, I., & Feldman, G. (1968). "A 3x3 isotropic gradient operator for image processing"
- De Berg, M., et al. (2008). "Computational Geometry: Algorithms and Applications"
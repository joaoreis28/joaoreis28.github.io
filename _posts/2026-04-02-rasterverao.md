---
title: "Rasterização de Imagens: Do Triângulo ao Anti-Aliasing"
date: 2026-02-06 10:00:00 -0300
categories: [IMPA, Computação Visual]
tags: [rasterizacao, computacao-grafica, impa, python, aliasing, triangulos]
math: true
image: /assets/images/rasterization.png
---

## Contexto

Este post documenta a solução do exercício de **Rasterização de Imagens** da disciplina Temas de Computação Visual (IMPA, verão 2026). O objetivo é implementar algoritmos de rasterização de primitivas geométricas, investigar problemas de **aliasing** em diferentes resoluções e implementar filtros anti-aliasing.

## O Problema

O exercício propõe uma série de tarefas progressivas envolvendo rasterização:

1. Implementar teste de ponto-em-triângulo
2. Investigar efeitos de resolução na rasterização
3. Analisar problemas em fronteiras de triângulos
4. Rasterizar cenas complexas (lion_scene)
5. Criar cenas com funções implícitas
6. Implementar filtros anti-aliasing
7. Desafios: rotação e fractais

**Repositório do código**: [https://github.com/ganacim/tcv_raster_2026](https://github.com/ganacim/tcv_raster_2026)

## Fundamento Teórico

### Rasterização

**Rasterização** é o processo de converter descrições vetoriais de objetos geométricos (como triângulos) em uma grade discreta de pixels. Este é um dos problemas fundamentais da computação gráfica, essencial para renderização em tempo real.

### Teste Ponto-em-Triângulo

Para determinar se um ponto está dentro de um triângulo, existem várias abordagens. A implementação utilizada neste exercício baseia-se no **teste de orientação** usando produto vetorial.

#### Produto Vetorial 2D

Dados três pontos $\mathbf{v_1} = (x_1, y_1)$, $\mathbf{v_2} = (x_2, y_2)$ e $\mathbf{v_3} = (x_3, y_3)$, o produto vetorial 2D (na verdade, a componente z do produto vetorial 3D) é:

$$
\text{cross}(\mathbf{v_1}, \mathbf{v_2}, \mathbf{v_3}) = (x_2 - x_1)(y_3 - y_1) - (y_2 - y_1)(x_3 - x_1)
$$

Este valor indica a orientação:
- Se $> 0$: $\mathbf{v_3}$ está à **esquerda** da aresta $\overrightarrow{\mathbf{v_1v_2}}$
- Se $< 0$: $\mathbf{v_3}$ está à **direita** da aresta $\overrightarrow{\mathbf{v_1v_2}}$
- Se $= 0$: $\mathbf{v_3}$ é **colinear** com $\mathbf{v_1}$ e $\mathbf{v_2}$

#### Algoritmo de Contenção

Um ponto $\mathbf{p}$ está dentro do triângulo $\triangle ABC$ se e somente se ele está do **mesmo lado** de todas as três arestas. Assumindo vértices ordenados no sentido anti-horário:

$$
\text{inside}(\mathbf{p}) = \text{left}(A, B, \mathbf{p}) \land \text{left}(B, C, \mathbf{p}) \land \text{left}(C, A, \mathbf{p})
$$

onde $\text{left}(v_1, v_2, v_3)$ retorna verdadeiro se o produto vetorial é não-negativo.

## Solução Implementada

### Tarefa 1.1: Implementação do Método `in_out`

A classe `Triangle` recebeu dois métodos:

#### Método `left`

Implementa o teste de orientação usando produto vetorial 2D:
```python
def left(self, v1, v2, v3):
    """
    Testa se v3 está à esquerda da aresta direcionada v1->v2
    Retorna True se o produto vetorial >= 0
    """
    return (v2[0] - v1[0]) * (v3[1] - v1[1]) - (v2[1] - v1[1]) * (v3[0] - v1[0]) >= 0
```

**Análise da implementação:**
O principal ponto de atenção aqui é o operador `>=` que inclui pontos na fronteira do triângulo, fato que será destacado em itens seguintes.

#### Método `in_out`

Aplica o teste de orientação para as três arestas do triângulo:
```python
def in_out(self, point):
    """
    Determina se um ponto está dentro do triângulo
    Retorna True se o ponto está do mesmo lado de todas as três arestas
    """
    a = self.left(self.v1, self.v2, point)
    b = self.left(self.v2, self.v3, point)
    c = self.left(self.v3, self.v1, point)
    return a and b and c
```

**Propriedades importantes:**
- Complexidade temporal: $O(1)$ - apenas três produtos vetoriais
- Robusto a erros numéricos para casos típicos
- Inclusivo nas bordas (devido ao `>=`)

#### Teste com `triangle_scene`

Após a implementação, a cena `triangle_scene` foi rasterizada com sucesso:
```bash
python3 raster.py --s triangle_scene
# Rasterizing triangle_scene at 400x300...
# Output saved to output.png
```

![Rasterização da cena triangle_scene](assets/images/raster/output.png){: width="500" height="300" }
_Exemplo original da cena triangle_scene rasterizada._

### Tarefa 1.2: Análise de Resolução

A segunda etapa do exercício consistiu em rasterizar a cena `triangle_scene` variando a resolução de saída. O objetivo é observar como a discretização da geometria afeta a qualidade visual da imagem final.

Executei o rasterizador com três configurações distintas de resolução, mantendo a janela de visualização fixa:
```bash
# Baixa Resolução (20x20)
python3 raster.py -s triangle_scene -r 20 20 -o triangle_low.png

# Média Resolução (200x200)
python3 raster.py -s triangle_scene -r 200 200 -o triangle_mid.png

# Alta Resolução (800x800)
python3 raster.py -s triangle_scene -r 800 800 -o triangle_high.png
```

![Triângulo em baixa resolução 20x20](assets/images/raster/triangulo20_20.png){: width="500" height="300" }
_Baixa resolução (20x20) - serrilhamento evidente._

![Triângulo em média resolução 200x200](assets/images/raster/triangulo200_200.png){: width="500" height="300" }
_Média resolução (200x200) - serrilhamento reduzido._

![Triângulo em alta resolução 800x800](assets/images/raster/triangulo800_800.png){: width="500" height="300" }
_Alta resolução (800x800) - bordas mais suaves._

![Detalhe do serrilhamento em alta resolução](assets/images/raster/serri.png){: width="500" height="300" }
_Destaque do serrilhamento persistente mesmo em alta resolução._

#### Análise do Fenômeno

O que observamos aqui é o fenômeno conhecido como **aliasing** (ou serrilhamento). Isso ocorre devido à natureza discreta do buffer de imagem em contraste com a natureza contínua da geometria vetorial.

Na imagem de baixa resolução (20x20), os "degraus" são evidentes, pois a grade de amostragem é grosseira. Ao aumentar a resolução para 800x800, os degraus diminuem de tamanho, tornando a imagem visualmente mais suave.

No entanto, matematicamente e visualmente (ao aplicar zoom), o erro persiste. Estamos tentando representar uma borda inclinada de frequência infinita (uma descontinuidade perfeita) usando uma grade de amostragem de frequência finita.

**Conclusão**: Aumentar a resolução apenas empurra o problema para frequências mais altas, mas não o elimina. A solução definitiva exigirá técnicas de **anti-aliasing**.

### Tarefa 1.3: Triângulos Adjacentes

A terceira tarefa propôs um cenário clássico de teste de robustez: rasterizar dois triângulos que compartilham uma aresta comum (e dois vértices). O objetivo é verificar a consistência matemática do teste de inclusão na fronteira entre os objetos.

Criei uma cena específica onde dois triângulos compartilham a aresta vertical definida por $x=3$:
```python
# Triângulo vermelho (esquerda)
self.add(Triangle((1.0, 1.0), (3.0, 1.0), (3.0, 3.0)), Color(1.0, 0.0, 0.0))

# Triângulo azul (direita)
self.add(Triangle((3.0, 1.0), (5.0, 1.0), (3.0, 3.0)), Color(0.0, 0.0, 1.0))
```

#### O Problema Teórico: Frestas

Em implementações ingênuas de rasterização, é comum utilizar desigualdades estritas (`> 0`) para o teste de orientação (função `left`).

Se um pixel cair exatamente sobre a aresta compartilhada, o resultado do produto vetorial (ou função de aresta) será zero.

- Para o triângulo da esquerda: `0 > 0` é **Falso**
- Para o triângulo da direita: `0 > 0` é **Falso**

Nesse cenário, nenhum dos triângulos reivindica o pixel, resultando em uma linha de pixels não pintados (**frestas**).

#### Análise da Minha Solução

Ao executar o meu rasterizador, **não observei frestas**. A junção entre os triângulos foi renderizada de forma contínua. Isso ocorreu porque, na minha implementação da classe `Triangle` em `src/shapes.py`, optei pelo uso do operador **maior ou igual** (`>=`):

![Triângulos adjacentes sem frestas](assets/images/raster/triangulocompartilhado300_300No.png){: width="500" height="300" }
_Triângulos adjacentes compartilhando aresta - sem frestas visíveis._

**Trade-off**: Embora essa escolha elimine frestas, pode causar **double-counting** em alguns casos especiais. Uma solução robusta para produção seria implementar **tie-breaking rules** baseadas na orientação das arestas.

### Tarefa 1.4: Triângulos Separados

A quarta tarefa explora o cenário oposto ao anterior: o que acontece quando existe um espaço vazio muito pequeno entre dois objetos, potencialmente menor que o tamanho de um pixel?

Criei uma cena de teste (`separated_scene.py`) contendo dois triângulos muito próximos, separados por uma distância horizontal de apenas **0.1 unidade**:
```python
# Triângulo 1 (Vermelho): termina em x=3.0
self.add(Triangle((1.0, 1.0), (3.0, 1.0), (3.0, 3.0)), Color(1.0, 0.0, 0.0))

# Triângulo 2 (Azul): começa em x=3.1 (gap de 0.1)
self.add(Triangle((3.1, 1.0), (5.1, 1.0), (3.1, 3.0)), Color(0.0, 0.0, 1.0))
```

Realizei testes variando a resolução para alterar o tamanho relativo do pixel em relação a esse gap de 0.1.

#### Resultados e Análise

O comportamento observado ilustra perfeitamente os problemas de **amostragem de sinais**:

**Baixa Resolução (Pixel > Gap)**: Quando a resolução é baixa (ex: 40x30), a largura de um pixel no espaço do mundo é maior que 0.1. Desse modo, o espaço vazio desapareceu completamente em várias linhas.

![Triângulos separados em baixa resolução](assets/images/raster/trianguloDistante40_30.png){: width="500" height="300" }
_Resolução 40x30 - gap invisível, triângulos parecem conectados._

**Alta Resolução (Pixel < Gap)**: Ao aumentar a resolução (ex: 800x600), o pixel se torna significativamente menor que 0.1. Assim, o gap vertical apareceu claramente separando os dois triângulos.

![Triângulos separados em alta resolução](assets/images/raster/trianguloDistante800_600.png){: width="500" height="300" }
_Resolução 800x600 - gap visível, triângulos claramente separados._

**Implicação prática**: Detalhes geométricos menores que o tamanho do pixel são **perdidos** ou **distorcidos** no processo de amostragem. Este é outro aspecto do problema de aliasing.

### Tarefa 1.5: Cena Complexa (Lion)

A quinta tarefa aumenta a complexidade do teste: rasterizar uma cena composta por milhares de triângulos. Diferente das formas geométricas simples anteriores, este modelo exige atenção ao sistema de coordenadas e à orientação das primitivas.

#### O Problema da Imagem em Branco

Ao executar o rasterizador pela primeira vez com a cena do leão, o resultado foi uma imagem completamente branca, contendo apenas a cor de fundo. O problema residia na **orientação dos vértices**.

O arquivo original define o leão de cabeça para baixo. O script `lion_scene.py` corrige isso aplicando uma reflexão no eixo Y:
```python
y1 = -y1 + 381
y2 = -y2 + 381
y3 = -y3 + 381
```

**O problema**: Geometricamente, multiplicar um eixo por −1 inverte a **quiralidade** do sistema. Triângulos que eram definidos no sentido **anti-horário** (CCW) passaram a ficar no sentido **horário** (CW).

Como meu método `in_out` (baseado no produto vetorial) verifica se o ponto está à esquerda das arestas (esperando ordem CCW), todos os testes retornavam `False` para os pontos internos dos triângulos, pois as arestas estavam "do avesso".

**A solução**: Inverter a ordem de declaração dos vértices ao instanciar o triângulo na cena, trocando `v2` por `v3`:
```python
# Antes (ordem errada após reflexão)
Triangle(v1, v2, v3)

# Depois (corrigindo a quiralidade)
Triangle(v1, v3, v2)
```

#### Ajuste de Janela e Resultados

Outro desafio foi o enquadramento. As coordenadas do leão variam aproximadamente entre (0,0) e (240,380), muito além da janela padrão (0,8). Foi necessário ajustar os parâmetros de `Window` (`-w`) para enquadrar a bounding box do modelo:
```bash
python3 raster.py -s lion_scene -r 100 160 -w 0 240 0 380 -o lion_100x160.png
python3 raster.py -s lion_scene -r 300 480 -w 0 240 0 380 -o lion_300x480.png
python3 raster.py -s lion_scene -r 600 960 -w 0 240 0 380 -o lion_600x960.png
```

![Leão em resolução 100x160](assets/images/raster/lion_100x160.png){: width="100" height="160" }
_Resolução 100x160 - aliasing severo._

![Leão em resolução 300x480](assets/images/raster/lion_300x480.png){: width="300" height="480" }
_Resolução 300x480 - qualidade intermediária._

![Leão em resolução 600x960](assets/images/raster/lion_600x960.png){: width="600" height="960" }
_Resolução 600x960 - maior fidelidade visual._

**Observação**: Mesmo em alta resolução, o serrilhamento permanece visível nas bordas. A próxima etapa (anti-aliasing) será crucial para melhorar a qualidade visual.

## Tarefa 2: Funções Implícitas

Após a rasterização de triângulos (primitivas definidas explicitamente por vértices), o exercício avança para uma classe de formas completamente diferente: as **funções implícitas**.

Diferente de um triângulo, onde parametrizamos a superfície ou definimos suas fronteiras por retas, uma forma implícita é definida por uma equação $f(x,y)$, onde o interior da forma é o conjunto de pontos que satisfazem $f(x,y) \leq 0$.

A tarefa exigia renderizar a região definida pela seguinte equação polinomial de quarto grau:

$$
\begin{align}
f(x, y) = &\ 0.004 + 0.110x - 0.177y - 0.174x^2 + 0.224xy - 0.303y^2 \\
          &- 0.168x^3 + 0.327x^2y - 0.087xy^2 - 0.013y^3 \\
          &+ 0.235x^4 - 0.667x^3y + 0.745x^2y^2 - 0.029xy^3 + 0.072y^4
\end{align}
$$

### Implementação

A classe `ImplicitFunction` implementa o método `in_out` avaliando a função para cada pixel:
```python
def in_out(self, point):
    """
    Retorna True se f(x,y) <= 0
    """
    x, y = point
    result = self.evaluate(x, y)
    return result <= 0
```

### Resultado

![Função implícita rasterizada](assets/images/raster/implicit.png){: width="500" height="300" }
_Região definida pela função implícita polinomial de quarto grau._

**Observação**: Assim como nos triângulos, o aliasing permanece evidente nas bordas curvas da forma implícita. A implementação de anti-aliasing será fundamental para suavizar essas transições.


## Tarefa 3: Implementação de Anti-Aliasing

A terceira tarefa consiste em implementar **filtros de anti-aliasing** para suavizar o efeito de serrilhamento observado nas tarefas anteriores. O anti-aliasing funciona tomando múltiplas amostras por pixel e combinando-as com diferentes estratégias de ponderação.

### Refatoração do Código Base

O código original utilizava uma abordagem de **amostragem única** por pixel, tomando apenas o centro do pixel:
```python
# Código original - 1 amostra por pixel
x_coords = [xmin + (xmax - xmin) * (i + 0.5) / width for i in range(width)]
y_coords = [ymin + (ymax - ymin) * (j + 0.5) / height for j in range(height)]

for j, i in product(range(height), range(width)):
    point = (x_coords[i], y_coords[j])
    # Testa apenas 1 ponto por pixel
```

Para implementar anti-aliasing, foi necessário **refatorar completamente** o loop de rasterização para suportar:
1. **Múltiplas amostras por pixel** (grid NxN)
2. **Diferentes estratégias de posicionamento** das amostras
3. **Ponderação variável** das amostras

### Novos Parâmetros de Linha de Comando

Adicionei dois parâmetros ao script `raster.py`:
```bash
-n, --samples    # Número de amostras por dimensão (ex: 3 = 3x3 = 9 amostras/pixel)
-f, --filter     # Tipo de filtro: box, jitter, random, hat, gaussian
```

**Exemplos de uso:**
```bash
# Box filter com 4x4 amostras
python3 raster.py -s triangle_scene -n 4 -f box -o triangle_box_4x4.png

# Gaussian filter com 5x5 amostras
python3 raster.py -s lion_scene -n 5 -f gaussian -o lion_gaussian_5x5.png

# Random sampling com 64 amostras
python3 raster.py -s implicit_scene -n 8 -f random -o implicit_random_64.png
```

### Algoritmos Implementados

#### 1. Box Filter (Uniform Sampling)

O filtro mais simples. Divide o pixel em uma grade regular NxN e toma amostras no **centro** de cada sub-região, atribuindo **peso uniforme** (1.0) a todas.
```python
if args.filter == 'box':
    step = 1.0 / args.samples
    for sy in range(args.samples):
        for sx in range(args.samples):
            # Centro da célula do grid
            offset_x = (sx + 0.5) * step
            offset_y = (sy + 0.5) * step
            weight = 1.0  # Peso uniforme
            
            sample_x = px_base + offset_x * pixel_w
            sample_y = py_base + offset_y * pixel_h
            # ... teste de colisão ...
```

**Propriedades:**
-  Simples e determinístico
-  Bom para texturas e padrões regulares
-  Pode criar padrões de moiré em bordas diagonais

**Complexidade:** $O(N^2)$ amostras por pixel

#### 2. Jitter Filter (Stratified Sampling)

Similar ao box, mas adiciona **perturbação aleatória** dentro de cada sub-célula do grid. Mantém peso uniforme mas quebra a regularidade do padrão de amostragem.
```python
elif args.filter == 'jitter':
    step = 1.0 / args.samples
    for sy in range(args.samples):
        for sx in range(args.samples):
            # Posição aleatória dentro da célula
            offset_x = (sx + random.random()) * step
            offset_y = (sy + random.random()) * step
            weight = 1.0  # Peso uniforme
```

**Propriedades:**
-  Quebra padrões regulares (reduz moiré)
-  Mantém distribuição estratificada (melhor cobertura que random puro)
-  Bom compromisso qualidade/custo

**Complexidade:** $O(N^2)$ amostras por pixel

#### 3. Random Filter (Pure Random Sampling)

Posiciona amostras de forma **completamente aleatória** dentro do pixel, sem estrutura de grid. Implementado como loop linear para $N^2$ iterações.
```python
elif args.filter == 'random':
    num_iterations = args.samples * args.samples
    for _ in range(num_iterations):
        # Posição completamente aleatória no pixel
        offset_x = random.random()
        offset_y = random.random()
        weight = 1.0
```

**Propriedades:**
-  Elimina completamente padrões de aliasing
-  Converge para solução correta com muitas amostras (Monte Carlo)
-  Requer mais amostras que stratified para mesma qualidade
-  Resultado varia entre execuções

**Complexidade:** $O(N^2)$ amostras por pixel

#### 4. Hat Filter (Triangular/Tent Filter)

Também conhecido como **tent filter** ou **bilinear filter**. Usa grid estratificado (jitter) mas pondera as amostras por sua **distância ao centro** do pixel usando uma função linear (cone).
```python
elif args.filter == 'hat':
    # Grid com jitter
    offset_x = (sx + random.random()) * step
    offset_y = (sy + random.random()) * step
    
    # Peso baseado na distância ao centro (0.5, 0.5)
    dx = offset_x - 0.5
    dy = offset_y - 0.5
    dist = math.sqrt(dx*dx + dy*dy)
    
    # Cone: 1.0 no centro, 0.0 na borda (raio=0.5)
    weight = max(0.0, 1.0 - dist * 2.0)
```

**Função de peso:**

$$
w(d) = \max(0, 1 - 2d)
$$

onde $d$ é a distância euclidiana ao centro do pixel.

**Propriedades:**
-  Suavização mais natural que box
-  Prioriza informação próxima ao centro
-  Pode causar leve desfoque

**Complexidade:** $O(N^2)$ amostras + cálculo de distância por amostra

#### 5. Gaussian Filter

O filtro de mais alta qualidade. Usa grid estratificado e pondera as amostras por uma **distribuição gaussiana** centrada no pixel.
```python
elif args.filter == 'gaussian':
    offset_x = (sx + random.random()) * step
    offset_y = (sy + random.random()) * step
    
    # Peso gaussiano
    dx = offset_x - 0.5
    dy = offset_y - 0.5
    dist_sq = dx*dx + dy*dy
    
    sigma = 0.35  # Controla a largura da gaussiana
    weight = math.exp(-dist_sq / (2 * sigma * sigma))
```

**Função de peso:**

$$
w(x,y) = e^{-\frac{(x-0.5)^2 + (y-0.5)^2}{2\sigma^2}}
$$

com $\sigma = 0.35$ (escolha empírica para equilíbrio entre suavidade e nitidez).

**Propriedades:**
-  Melhor qualidade visual (transição mais suave)
-  Fundamentação matemática (teoria de sinais)
-  Usado em produção (ray tracing, motion blur)
-  Custo computacional ligeiramente maior (exponencial)

**Complexidade:** $O(N^2)$ amostras + cálculo de exponencial por amostra

### Estrutura do Loop de Rasterização

O código refatorado segue esta estrutura para cada pixel:
```python
for j in range(height):
    for i in range(width):
        accumulated_color = np.array([0.0, 0.0, 0.0])
        total_weight = 0.0
        
        # Para cada amostra no pixel
        for sample in generate_samples(args.filter, args.samples):
            # 1. Calcular posição da amostra
            sample_x, sample_y = compute_sample_position(...)
            
            # 2. Calcular peso da amostra
            weight = compute_weight(args.filter, sample_position)
            
            # 3. Teste de colisão (reversed scene list para early exit)
            hit_color = test_scene_collision(sample_x, sample_y)
            
            # 4. Acumular cor ponderada
            accumulated_color += hit_color * weight
            total_weight += weight
        
        # 5. Normalização pelo total de pesos
        final_color = accumulated_color / total_weight
        image_data[j, i] = (final_color * 255).astype(np.uint8)
```

**Otimização importante:** A lista de objetos da cena é **invertida** antes do loop para permitir **early exit** no teste de colisão (objetos da frente para trás).

### Resultados Comparativos

#### Cena: Triangle Scene (800x800)



![Comparação de filtros - Triangle Scene](assets/images/raster/tri.png){: width="800" }
_Cena triangle_scene sem anti-aliasing_

![Comparação de filtros - Triangle Scene](assets/images/raster/triGauss.png){: width="800" }
_Cena triangle_scene com filtro Gaussiano._


#### Cena: Lion (600x960)

A cena do leão, com suas milhares de bordas de triângulos, é um teste excelente para avaliar a eficácia do anti-aliasing:

![Leão sem anti-aliasing](assets/images/raster/lion_semaa.png){: width="300" height="480" }
_Sem anti-aliasing (1x1) - aliasing severo nas bordas._

![Leão com Gaussian 4x4](assets/images/raster/lion_aa4.png){: width="300" height="480" }
_Gaussian filter 4x4 - bordas suavizadas._

![Leão com Gaussian 8x8](assets/images/raster/lion_gaussian_8x8.png){: width="300" height="480" }
_Gaussian filter 8x8 - qualidade próxima ao ideal._





## Desafios

### Desafio 1: Rotação de Cenas

O primeiro desafio consiste em rasterizar cenas onde o conteúdo está **rotacionado em torno de um ponto central**. Este problema é comum em aplicações gráficas e exige o uso de **transformações geométricas**.

#### Abordagem: Wrapper de Transformação

Em vez de modificar cada primitiva individual para suportar rotação, implementei uma solução  usando o **padrão Decorator**: a classe `RotatedShape` que encapsula qualquer forma e aplica uma transformação de rotação inversa aos pontos de teste.

#### Matemática da Rotação

Para testar se um ponto $\mathbf{p} = (x, y)$ está dentro de uma forma rotacionada por um ângulo $\theta$ em torno de um centro $\mathbf{c} = (c_x, c_y)$, precisamos aplicar a **transformação inversa** ao ponto antes de testá-lo contra a forma original.

**Passos da transformação:**

1. **Transladar** para o sistema de coordenadas local (centro na origem):

$$
\mathbf{t} = \mathbf{p} - \mathbf{c} = (x - c_x, y - c_y)
$$

2. **Rotacionar** no sentido inverso ($-\theta$):

$$
\begin{bmatrix} x' \\ y' \end{bmatrix} = \begin{bmatrix} \cos\theta & \sin\theta \\ -\sin\theta & \cos\theta \end{bmatrix} \begin{bmatrix} t_x \\ t_y \end{bmatrix}
$$

Expandindo:

$$
\begin{align}
x' &= t_x \cos\theta + t_y \sin\theta \\
y' &= -t_x \sin\theta + t_y \cos\theta
\end{align}
$$

3. **Transladar de volta** ao sistema original:

$$
\mathbf{p'} = (x' + c_x, y' + c_y)
$$

**Intuição**: Em vez de rotacionar a forma no sentido horário, rotacionamos o ponto de teste no sentido anti-horário e testamos contra a forma original. É matematicamente equivalente mas muito mais simples de implementar.

#### Implementação da Classe `RotatedShape`
```python
class RotatedShape(Shape):
    def __init__(self, shape, angle_degrees, center=(0,0)):
        super().__init__("rotated_shape")
        self.shape = shape
        self.angle = math.radians(angle_degrees)
        self.center = center
        
        # Pré-calcula seno e cosseno da rotação INVERSA
        # cos(-θ) = cos(θ)
        # sin(-θ) = -sin(θ)
        self.c = math.cos(self.angle)
        self.s = math.sin(self.angle)

    def in_out(self, point):
        # 1. Transladar para sistema local
        tx = point[0] - self.center[0]
        ty = point[1] - self.center[1]

        # 2. Aplicar rotação inversa
        rx = tx * self.c + ty * self.s
        ry = -tx * self.s + ty * self.c

        # 3. Transladar de volta
        final_x = rx + self.center[0]
        final_y = ry + self.center[1]

        # 4. Testar contra forma original
        return self.shape.in_out((final_x, final_y))
```

**Otimização importante**: Os valores de seno e cosseno são **pré-calculados** no construtor, evitando chamadas repetidas a funções trigonométricas no loop de rasterização.

#### Cena de Teste: Função Implícita Rotacionada

Criei uma cena (`rotacao.py`) que demonstra a rotação aplicada à função implícita do exercício anterior:
```python
class Scene(BaseScene):
    def __init__(self):
        super().__init__("Rotated Scene")
        self.background = Color(1, 1, 1)

        # Forma original
        original_shape = ImplicitFunction(my_function)
        
        # Adiciona forma original (azul claro) para referência
        self.add(original_shape, Color(0.8, 0.8, 1.0))

        # Adiciona cópia rotacionada em 45° (azul escuro)
        rotated_45 = RotatedShape(original_shape, 45, center=(0,0))
        self.add(rotated_45, Color(0.0, 0.0, 0.5))
```

Esta cena renderiza **duas versões** da mesma forma implícita:
- **Azul claro**: forma original (sem rotação)
- **Azul escuro**: forma rotacionada 45° no sentido anti-horário

#### Resultados

![Função implícita rotacionada sem AA](assets/images/raster/challenge_rotation.png){: width="600" }
_Função implícita original (azul claro) e rotacionada 45° (azul escuro)._


#### Vantagens da Abordagem

1. **Generalidade**: Funciona com **qualquer** primitiva (triângulos, círculos, funções implícitas, Mandelbrot)
2. **Composição**: Pode-se rotacionar formas já rotacionadas (rotações compostas)
3. **Eficiência**: Apenas 4 multiplicações + 4 somas por ponto testado
4. **Simplicidade**: Não modifica as classes de primitivas existentes

#### Complexidade

**Temporal**: $O(1)$ overhead por teste `in_out`
- 4 multiplicações (para rotação)
- 4 adições (translações)
- 1 chamada ao `in_out` da forma original

**Espacial**: $O(1)$ - apenas armazena ângulo, centro e referência à forma


**Exemplo de transformação completa:**
```python
class TransformedShape(Shape):
    def __init__(self, shape, matrix):
        self.shape = shape
        self.inv_matrix = inverse(matrix)  # Pré-calcula inversa
    
    def in_out(self, point):
        transformed = self.inv_matrix @ point
        return self.shape.in_out(transformed)
```

### Desafio 2: Conjunto de Mandelbrot

O segundo desafio consiste em visualizar o **Conjunto de Mandelbrot**, um dos fractais mais famosos da matemática, definido no plano complexo.

#### Definição Matemática

O conjunto de Mandelbrot $\mathcal{M}$ é o conjunto de números complexos $c$ para os quais a sequência recursiva:

$$
\begin{align}
z_0 &= 0 \\
z_{n+1} &= z_n^2 + c
\end{align}
$$

permanece **limitada** (não diverge para infinito).


**Critério de escape**: Se em algum momento $\vert z_n \vert > 2$, sabemos que a sequência divergirá.
Como $\vert z \vert^2 = \text{real}^2 + \text{imag}^2$, testamos se $\vert z \vert^2 > 4$.

#### Implementação da Classe `Mandelbrot`
```python
class Mandelbrot(Shape):
    def __init__(self, max_iter=100):
        super().__init__("mandelbrot")
        self.max_iter = max_iter

    def in_out(self, point):
        # Mapeia o pixel (x, y) para o plano complexo c
        c = complex(point[0], point[1])
        z = 0j
        
        for _ in range(self.max_iter):
            # Equação fundamental: z = z² + c
            z = z * z + c
            
            # Teste de escape: |z|² > 4
            if (z.real * z.real + z.imag * z.imag) > 4:
                return False  # Escapou → fora do conjunto
        
        # Não escapou após max_iter → dentro do conjunto
        return True
```

**Parâmetro `max_iter`**: Controla a precisão da aproximação. Valores maiores revelam mais detalhes nas fronteiras, mas custam mais computacionalmente.

- `max_iter = 50`: Visualização básica, rápida
- `max_iter = 100`: Boa qualidade geral (**escolha padrão**)
- `max_iter = 256`: Alta qualidade, revela detalhes finos
- `max_iter = 1000`: Zoom extremo em regiões complexas

#### Cena de Teste

A cena `mandelbrot.py` implementa uma visualização clássica em preto e branco:
```python
from src.base import BaseScene, Color
from src.shapes import Mandelbrot

class Scene(BaseScene):
    def __init__(self):
        super().__init__("Mandelbrot Set")
        self.background = Color(1, 1, 1)  # Fundo branco
        
        # Conjunto de Mandelbrot (preto)
        # max_iter=100 é suficiente para uma boa visualização básica
        self.add(Mandelbrot(max_iter=100), Color(0.0, 0.0, 0.0))
```

**Escolha de cores**: Preto para pontos **dentro** do conjunto (não escapam), branco para pontos **fora** (escapam). Esta é a visualização clássica que destaca a estrutura característica do conjunto.

#### Janelas de Visualização

O conjunto de Mandelbrot está centrado aproximadamente em $(-0.5, 0)$ no plano complexo. Para visualizá-lo, precisamos escolher janelas apropriadas:

**Visão completa (overview):**
```bash
python3 raster.py -s mandelbrot -w -2.5 1.0 -1.25 1.25 -r 1400 1000 -o mandelbrot_full.png
```
- Janela: Real $\in [-2.5, 1.0]$, Imaginário $\in [-1.25, 1.25]$
- Mostra toda a estrutura principal: corpo central, "cabeça", antenas

**Zoom moderado (região interessante):**
```bash
python3 raster.py -s mandelbrot -w -0.8 -0.4 -0.2 0.2 -r 1200 1200 -n 4 -f gaussian -o mandelbrot_mid.png
```
- Foca na região próxima ao "pescoço" do Mandelbrot
- Revela mini-Mandelbrots e estruturas espirais

**Zoom na região "Seahorse Valley":**
```bash
python3 raster.py -s mandelbrot -w -0.75 -0.73 0.1 0.12 -r 1000 1000 -n 8 -f gaussian -o mandelbrot_seahorse.png
```
- Uma das regiões mais icônicas do conjunto
- Estruturas que lembram cavalos-marinhos

**Zoom extremo (auto-similaridade):**
```bash
python3 raster.py -s mandelbrot -w -0.7436 -0.7426 0.1318 0.1328 -r 2000 2000 -n 8 -f gaussian -o mandelbrot_zoom.png
```
- Demonstra a propriedade fractal de auto-similaridade infinita
- Requer `max_iter` maior (256 ou mais) para detalhes precisos

#### Resultados

![Conjunto de Mandelbrot - visão completa](assets/images/raster/mandelbrot_full.png){: width="700" }
_Visão completa do conjunto de Mandelbrot (janela: -2.5 a 1.0, -1.25 a 1.25) com max_iter=100._

![Mandelbrot - zoom moderado](assets/images/raster/mandelbrot_mid.png){: width="600" }
_Zoom na região central - estruturas fractais começam a aparecer._

![Mandelbrot - Seahorse Valley](assets/images/raster/mandelbrot_seahorse.png){: width="600" }
_Zoom na região "Seahorse Valley" - estruturas fractais auto-similares detalhadas._

![Mandelbrot - zoom extremo](assets/images/raster/mandelbrot_zoom.png){: width="600" }
_Zoom extremo demonstrando auto-similaridade infinita - mini-Mandelbrots aparecem._

#### A Importância do Anti-Aliasing em Fractais

O conjunto de Mandelbrot tem **detalhes infinitos** em todas as escalas, o que torna o anti-aliasing **absolutamente crítico** para renderização de qualidade:

**Sem anti-aliasing (1x1):**
- Aliasing severo nas fronteiras irregulares
- Perda completa de detalhes finos (filamentos, antenas)
- Artefatos de "escada" evidentes
- Estruturas finas desaparecem completamente

**Com Gaussian 4x4:**
- Fronteiras suavizadas
- Detalhes finos começam a aparecer
- Transição mais natural entre interior/exterior

**Com Gaussian 8x8:**
- Fronteiras suaves e precisas
- Detalhes finos preservados (filamentos visíveis)
- Transição gradual e matematicamente correta
- Estruturas complexas renderizadas fielmente





#### Análise de Performance

**Complexidade por pixel:**

$$
T_{\text{pixel}} = O(N^2 \cdot I_{\text{avg}})
$$

onde:
- $N^2$ é o número de amostras de anti-aliasing (ex: 64 para 8x8)
- $I_{\text{avg}}$ é o número médio de iterações até escape

**Observação importante sobre $I_{\text{avg}}$**:
- Pontos **dentro** do conjunto: sempre `max_iter` iterações (pior caso)
- Pontos **fora** próximos à fronteira: $\sim$ 20-50 iterações
- Pontos **fora** longe da fronteira: < 10 iterações (escapam rápido)

**Distribuição típica** (para janela completa):
- ~20% dos pixels estão dentro (custo alto)
- ~30% estão na fronteira (custo médio-alto)
- ~50% estão longe (custo baixo)







---


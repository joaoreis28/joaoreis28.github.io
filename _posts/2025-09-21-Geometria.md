---
title: Anotações de Geometria Computacional
date: 2025-09-21 09:53:00 -0300
categories: [Geometria Computacional]
tags: [geometria]     # TAG names should always be lowercase
description: Notas de aula  da disciplina de Geometria computacional.
math: true
pin : true 
image : /assets/images/voronoi.png
---

## Conceitos básicos e primitivas

### Orientação relativa de vetores
Para determinar a orientação de dois vetores no 
$$  \mathbb{R}^2 $$
, podemos calcular o determinante formado pelos vetores $$ \vec{v_1} = (x_1, y_1) $$ 
e 
$$ \vec{v_2} = (x_2, y_2) $$
, definido como:  

$$ \vec{v_1} \times \vec{v_2} = x_1 y_2 - x_2 y_1. $$

Se 
$$ \vec{v_1} \times \vec{v_2} < 0 $$
o vetor $$ \vec{v_2} $$ está à esquerda de $$ \vec{v_1} $$. De forma simétrica, se $$ \vec{v_1} \times \vec{v_2} > 0 $$
, o vetor $$ \vec{v_2} $$ está à direita de $$ \vec{v_1} $$ .


![Desktop View](assets/images/porta.png){: width="700" height="400" }
*O vetor $$ \vec{v_2} $$  está à esquerda do vetor $$ \vec{v_1} $$ .*

![Desktop View](assets/images/novocerto.png){: width="700" height="400" }
*O vetor $$ \vec{v_2} $$  está à direita do vetor $$ \vec{v_1} $$ .*


### Área orientada de polígonos planos
O valor absoluto do determinante formado pelos vetores $$ \vec{v_1} $$ e $$ \vec{v_2} $$ está relacionado com a área do paralelogramo definido entre eles.

### Fórmula Shoelace

Seja um polígono simples com vérices $$   (x_1, y_1), (x_2, y_2), \dots, (x_n, y_n)   $$ no plano  $$ \mathbb{R}^2 $$ a área $$S $$ desse polígono é dada por:

$$  S = \frac{1}{2} |(x_1y_2 + x_2y_3 + \dots + x_ny_1) - (y_1x_2 + y_2x_3 + \dots + y_nx_1)| $$ 

**Exemplo**: Vértices na ordem: $$ (8,2),\; (-7,10),\; (-5,-2),\; (-9,-2),\; (0,-10). $$
Montando o "determinante" do método:

$$
\begin{aligned}
\text{soma}_\rightarrow &= x_1y_2 + x_2y_3 + x_3y_4 + x_4y_5 + x_5y_1 \\
&= 8\cdot 10 \;+\; (-7)\cdot(-2) \;+\; (-5)\cdot(-2) \;+\; (-9)\cdot(-10) \;+\; 0\cdot 2 \\
&= 80 \;+\; 14 \;+\; 10 \;+\; 90 \;+\; 0 \\
&= 194, \\[6pt]
\text{soma}_\leftarrow &= y_1x_2 + y_2x_3 + y_3x_4 + y_4x_5 + y_5x_1 \\
&= 2\cdot(-7) \;+\; 10\cdot(-5) \;+\; (-2)\cdot(-9) \;+\; (-2)\cdot 0 \;+\; (-10)\cdot 8 \\
&= -14 \;-\; 50 \;+\; 18 \;+\; 0 \;-\; 80 \\
&= -126.
\end{aligned}
$$

Logo,

$$
S \;=\; \frac{1}{2}\,\big|\,\text{soma}_\rightarrow - \text{soma}_\leftarrow\,\big|
\;=\;
\frac{1}{2}\,\big|\,194 - (-126)\,\big|
\;=\;
\frac{1}{2}\cdot 320
\;=\;
\boxed{160}.
$$

![Desktop View](assets/images/poligono.png){: width="700" height="400" }
*O Polígono do exemplo gerado no GeoGebra  .*

## Implementação

### Orientação Relativa
Aqui temos uma implentação direta do conceito de orientação relativa, primitiva que será amplamente utilizada posteriormente, testamos se o ponto $$ r $$ está à esquerda do segmento formado pelos pontos $$p$$ e $$q$$.


```c++
int left2(CGL::Point2 p, CGL::Point2 q, CGL::Point2 r)
    {
        return (q.x() - p.x()) * (r.y() - p.y()) - (q.y() - p.y()) * (r.x() - p.x()) > 0;
    }

    int right2(CGL::Point2 p, CGL::Point2 q, CGL::Point2 r)
    {
        return (q.x() - p.x()) * (r.y() - p.y()) - (q.y() - p.y()) * (r.x() - p.x()) < 0;
    }

    int collinear2(CGL::Point2 p, CGL::Point2 q, CGL::Point2 r)
    {
        return (q.x() - p.x()) * (r.y() - p.y()) - (q.y() - p.y()) * (r.x() - p.x()) == 0;
    }

```




### Fórmula de Shoelace
Implementação da fórmula de shoelace para um polígono com n lados.

```c++
double areaPoligono(CGL::Polygon2 poligono, int nVertices)
    {
        double soma = 0.0;

        // formuula shoelace
        for(int i = 0 ; i < nVertices; i++)
        {
            CGL::Point2 p1 = poligono[i];
            CGL::Point2 p2 = poligono[(i + 1) % nVertices];
            soma += (p1.x() * p2.y()) - (p1.y() * p2.x());

        }

        return soma/2.0;
    }
```

## Intersecção de Segmentos
Nesta seção detalharemos mais uma operação básica que será amplamente utilizada posteriormente, que é a intersecção de segmentos
### Intersecção Própria
**Problema**: dado dois segmentos $$ ab $$ e $$cd$$ no $$ \mathbb{R}^2 $$ e usando o produto vetorial definido acima, determinar se eles se interceptam.

**Solução**: os segmentos se interceptam se os pontos $$C$$ e $$D$$ estão em lados opostos em relação ao segmento $$ab$$ e, ao mesmo tempo, os pontos $$A$$ e $$B$$ estão de lados opostos em relação ao segmento $$cd$$. Para tal é necessário que os vetores $$ \overrightarrow{AC}$$ e $$ \overrightarrow{AD}$$ tenham orientações diferentes com relação a $$ \overrightarrow{AB}$$, ou seja, $$(\overrightarrow{AB} \times \overrightarrow{AC}) \cdot (\overrightarrow{AB} \times \overrightarrow{AD}) < 0$$. O mesmo é necessário para $$ \overrightarrow{CA}$$ e $$ \overrightarrow{CB}$$ com a relação a $$ \overrightarrow{CD}$$.

![Desktop View](assets/images/inter.png){: width="700" height="400" }
*Intersecção dos segmentos do  gerado no GeoGebra  .*

#### Implementação
Implementação da ideia descrita acima utilizando as primitivas já citadas de __left__ e __collinear__ .
```c++
int intersecPropria2(CGL::Segment2 s, CGL::Segment2 t)
    {
        if (collinear2(s[0], s[1], t[0]) || collinear2(s[0], s[1], t[1]) || collinear2(t[0], t[1], s[0]) || collinear2(t[0], t[1], s[1])) return 0;
        int a = left2(s[0], s[1], t[0]) xor left2(s[0], s[1], t[1]);
        int b = left2(t[0], t[1], s[0]) xor left2(t[0], t[1], s[1]);
        return a && b;
    }

```


### Intersecção Imprópria
No nosso contexto, definimos intersecção imprópria em dois casos:
-   Um extremo de um segmento coincide com um extremo do outro
![Desktop View](assets/images/interIm1.png){: width="700" height="400" }
*Intersecção dos segmentos do  gerado no GeoGebra  .*
-   Os segmentos são colineares e sobrepostos
![Desktop View](assets/images/interIm2.png){: width="700" height="400" }
*Intersecção dos segmentos do  gerado no GeoGebra  .*

### Implementação

```c++
int noMeio(int a, int b, int c)
    {
        return (std::min(a, b) <= c) && (c <= std::max(a, b));
    }

    int noSegmento(CGL::Point2 a, CGL::Point2 b, CGL::Point2 c)
    {
        return collinear2(a, b, c) && noMeio(a.x(), b.x(), c.x()) && noMeio(a.y(), b.y(), c.y());
    }

    int intersecImpropria2(CGL::Segment2 s, CGL::Segment2 t)
    {
        if (s[0] == t[0] || s[0] == t[1] || s[1] == t[0] || s[1] == t[1]) return 1;
        if (noSegmento(s[0], s[1], t[0]) || noSegmento(s[0], s[1], t[1]) || noSegmento(t[0], t[1], s[0]) || noSegmento(t[0], t[1], s[1])) return 1;
        return 0;
    }


```

## Triangulação de polígonos

# Particionamento de Polígonos em Pedaços Monótonos

## Introdução

O particionamento de polígonos é um problema fundamental em geometria computacional. Dado um polígono simples (sem auto-interseções), o objetivo é dividi-lo em pedaços mais simples que sejam mais fáceis de processar. Uma das decomposições mais úteis é o **particionamento em polígonos monótonos**, especialmente polígonos y-monótonos.

Um polígono é **y-monótono** quando qualquer linha horizontal o intersecta em no máximo dois pontos. Isso significa que, ao percorrer o polígono de cima para baixo, nunca precisamos "voltar" na direção vertical.

## Por que Particionar?

Polígonos monótonos são muito mais fáceis de triangular do que polígonos arbitrários. Uma vez que temos um polígono y-monótono, podemos triangulá-lo em tempo linear usando um algoritmo simples de varredura. Portanto, a estratégia geral para triangular um polígono qualquer é:

1. Particionar o polígono em pedaços y-monótonos
2. Triangular cada pedaço separadamente

## Classificação de Vértices

O algoritmo começa classificando cada vértice do polígono em um de cinco tipos, baseado em sua posição relativa aos vértices adjacentes e no ângulo interno:

### 1. **Start Vertex** 
- Ambos os vizinhos estão **abaixo** do vértice
- O ângulo interno é **menor que 180°** (convexo)
- Intuitivamente: o polígono "começa" neste ponto ao descer

### 2. **Split Vertex** 
- Ambos os vizinhos estão **abaixo** do vértice
- O ângulo interno é **maior que 180°** (côncavo)
- Este vértice "divide" o polígono e precisa de uma diagonal

### 3. **End Vertex** 
- Ambos os vizinhos estão **acima** do vértice
- O ângulo interno é **menor que 180°** (convexo)
- Intuitivamente: o polígono "termina" neste ponto ao descer

### 4. **Merge Vertex** 
- Ambos os vizinhos estão **acima** do vértice
- O ângulo interno é **maior que 180°** (côncavo)
- Dois "ramos" do polígono se juntam aqui

### 5. **Regular Vertex** 
- Um vizinho está acima e outro abaixo
- O polígono continua normalmente através deste vértice

## O Algoritmo de Varredura (Sweep Line)

O algoritmo usa a técnica de **linha de varredura** (sweep line), processando os vértices de cima para baixo (ordenados pela coordenada y decrescente).

### Estruturas de Dados

1. **Status da linha de varredura**: Mantém as arestas que atualmente intersectam a linha de varredura horizontal
2. **Helper**: Para cada aresta no status, armazena o último vértice que "ajudou" essa aresta (conceito explicado abaixo)
3. **Diagonais**: Lista de diagonais que precisam ser inseridas

### Conceito de Helper

O **helper** de uma aresta $e_j$ é o vértice mais baixo  que está acima da linha de varredura atual, tal que o segmento horizontal conectando esse vértice a aresta $e_j$ esteja completamente dentro do polígono.

### Tratamento de Cada Tipo de Vértice

#### Start Vertex
```
1. Insere a aresta que sai do vértice no status
2. Define este vértice como helper dessa aresta
```

#### End Vertex
```
1. Verifica se o helper da aresta que entra é MERGE
2. Se sim, adiciona diagonal entre o vértice atual e o helper
3. Remove a aresta do status
```

#### Split Vertex
```
1. Encontra a aresta diretamente à esquerda do vértice
2. Adiciona diagonal entre o vértice atual e o helper dessa aresta
3. Atualiza o helper da aresta à esquerda
4. Insere a aresta que sai no status
```

#### Merge Vertex
```
1. Se o helper da aresta que entra é MERGE, adiciona diagonal
2. Remove a aresta que entra do status
3. Encontra a aresta à esquerda
4. Se o helper dessa aresta é MERGE, adiciona diagonal
5. Atualiza o helper da aresta à esquerda
```

#### Regular Vertex
O tratamento depende se o interior do polígono está à direita ou à esquerda:

**Se interior está à esquerda:**
```
1. Se helper da aresta que entra é MERGE, adiciona diagonal
2. Remove aresta que entra do status
3. Insere aresta que sai no status
4. Define vértice atual como helper da nova aresta
```

**Se interior está à direita:**
```
1. Encontra aresta à esquerda
2. Se seu helper é MERGE, adiciona diagonal
3. Atualiza helper da aresta à esquerda
```

## Encontrando a Aresta à Esquerda

Uma operação crucial é encontrar a aresta imediatamente à esquerda de um vértice. Isso é feito:

1. Para cada aresta no status, calcula onde ela intersecta a linha horizontal que passa pelo vértice
2. Dentre as interseções à esquerda do vértice, escolhe a mais próxima (com maior coordenada x)


## Implementação

```c++
#include <iostream>
#include <CGAL/Arrangement_2.h>
#include <CGAL/Exact_predicates_exact_constructions_kernel.h>
#include <CGAL/Arr_segment_traits_2.h>
#include <vector>
#include <fstream>
#include <cmath>
#include <CGAL/draw_arrangement_2.h>
#include <string>

using std::string;
typedef CGAL::Exact_predicates_exact_constructions_kernel K;
typedef CGAL::Arr_segment_traits_2<K> Traits;
typedef CGAL::Arrangement_2<Traits> Arrangement;
typedef Traits::Point_2 Point_2;
typedef Traits::X_monotone_curve_2 Segment_2;




/// SVG Utils
double to_double(const K::FT& x) {
    return CGAL::to_double(x);
}

struct BBox {
    double xmin =  std::numeric_limits<double>::max();
    double ymin =  std::numeric_limits<double>::max();
    double xmax = -std::numeric_limits<double>::max();
    double ymax = -std::numeric_limits<double>::max();
};

BBox compute_bbox(const Arrangement& arr) {
    BBox b;
    for (auto vit = arr.vertices_begin(); vit != arr.vertices_end(); ++vit) {
        if (vit->is_at_open_boundary()) continue;
        auto p = vit->point();
        double x = to_double(p.x());
        double y = to_double(p.y());
        b.xmin = std::min(b.xmin, x);
        b.ymin = std::min(b.ymin, y);
        b.xmax = std::max(b.xmax, x);
        b.ymax = std::max(b.ymax, y);
    }
    return b;
}

double map_x(double x, const BBox& b, int W) {
    return 40 + (x - b.xmin) / (b.xmax - b.xmin) * (W - 80);
}

double map_y(double y, const BBox& b, int H) {
    return H - (40 + (y - b.ymin) / (b.ymax - b.ymin) * (H - 80));
}

void svg_arrow(std::ofstream& out,
               double x1, double y1,
               double x2, double y2,
               const string& color)
{
    out << "<line x1=\"" << x1 << "\" y1=\"" << y1
        << "\" x2=\"" << x2 << "\" y2=\"" << y2
        << "\" stroke=\"" << color
        << "\" stroke-width=\"2\" marker-end=\"url(#arrow)\" />\n";
}

void export_arrangement_svg(const Arrangement& arr,
                            const string& filename)
{
    const int W = 800;
    const int H = 800;

    std::ofstream svg(filename);
    BBox box = compute_bbox(arr);

    svg << "<svg xmlns=\"http://www.w3.org/2000/svg\" "
        << "width=\"" << W << "\" height=\"" << H << "\">\n";

    svg << R"(
<defs>
<marker id="arrow" markerWidth="10" markerHeight="10"
        refX="10" refY="3"
        orient="auto"
        markerUnits="strokeWidth">
  <path d="M0,0 L0,6 L9,3 z" fill="black"/>
</marker>
</defs>
)";

    for (auto he = arr.halfedges_begin();
         he != arr.halfedges_end(); ++he)
    {
        if (he->is_fictitious()) continue;

        auto p = he->source()->point();
        auto q = he->target()->point();

        double x1 = map_x(to_double(p.x()), box, W);
        double y1 = map_y(to_double(p.y()), box, H);
        double x2 = map_x(to_double(q.x()), box, W);
        double y2 = map_y(to_double(q.y()), box, H);

        bool interior_left = !he->face()->is_unbounded();

        string color = interior_left ? "red" : "gray";

        svg_arrow(svg, x1, y1, x2, y2, color);
    }

    for (auto vit = arr.vertices_begin();
         vit != arr.vertices_end(); ++vit)
    {
        if (vit->is_at_open_boundary()) continue;

        auto p = vit->point();
        double cx = map_x(to_double(p.x()), box, W);
        double cy = map_y(to_double(p.y()), box, H);

        svg << "<circle cx=\"" << cx
            << "\" cy=\"" << cy
            << "\" r=\"4\" fill=\"blue\" />\n";
    }

    svg << "</svg>\n";
    svg.close();
}

/// SVG Utils   






enum VertexType { START, SPLIT, END, MERGE, REGULAR };

bool is_above(const Point_2& p1, const Point_2& p2)
{
    if (p1.y() == p2.y())
        return p1.x() < p2.x(); 
    return p1.y() > p2.y();     
}

VertexType classify_vertex(Arrangement::Vertex_handle v)
{
    // Circulator que aponta para todas as arestas que apontam para um vértice 
    Arrangement::Halfedge_around_vertex_circulator circ = v->incident_halfedges();
    Arrangement::Halfedge_around_vertex_circulator curr = circ;
    

    // Itera sobre os circulators para achar a aresta associada a face
    Arrangement::Halfedge_handle he_in;
    bool found = false;
    do {
        if (!curr->face()->is_unbounded())
        {
            he_in = curr;
            found = true;
            break;
        }
        ++curr;
    } while (curr != circ);

    if (!found) return REGULAR;

    Point_2 prev_pt = he_in->source()->point(); // ponto anterior no CCW
    Point_2 next_pt = he_in->next()->target()->point();// proximo ponto
    Point_2 curr_pt = v->point(); // ponto atual

    bool prev_is_below = is_above(curr_pt, prev_pt);
    bool next_is_below = is_above(curr_pt, next_pt);

    auto orientation = CGAL::orientation(prev_pt, curr_pt, next_pt);
    bool is_reflex = (orientation == CGAL::RIGHT_TURN);


    // classificação dos vértices do livro De Berg
    if (prev_is_below && next_is_below)
    {
        if (is_reflex) return SPLIT;
        return START;
    }
    if(!prev_is_below && !next_is_below)
    {
        if(is_reflex) return MERGE;
        return END;
    }
    
    return REGULAR; 
}

Arrangement::Halfedge_handle get_incident_edge(Arrangement::Vertex_handle v)
{
    auto circ = v->incident_halfedges();
    auto curr = circ;
    do {
        if (!curr->face()->is_unbounded()) return curr; 
        ++curr;
    } while (curr != circ);
    return Arrangement::Halfedge_handle(); 
}

struct Diagonal {
    Point_2 p1, p2;
};

std::map<Arrangement::Halfedge_handle, Arrangement::Vertex_handle> helper;
std::vector<Arrangement::Halfedge_handle> status;
std::vector<Diagonal> diagonals_to_insert; 

void handle_start_vertex(Arrangement::Vertex_handle v)
{
    Arrangement::Halfedge_handle hin = get_incident_edge(v);
    if (hin == Arrangement::Halfedge_handle()) return; // Proteção
    
    Arrangement::Halfedge_handle hout = hin->next(); // aresta que sai

    status.push_back(hout); // coloca no status
    helper[hout] = v; // seta o helper
}

void handle_end_vertex(Arrangement::Vertex_handle v, Arrangement& arr)
{
    Arrangement::Halfedge_handle hin = get_incident_edge(v);
    if (hin == Arrangement::Halfedge_handle()) return; 
    

    if (helper.count(hin) == 0) return;
    
    // Verifica se o helper da aresta é merge
    Arrangement::Vertex_handle v_helper = helper[hin];
    
    if(classify_vertex(v_helper) == MERGE) {
        diagonals_to_insert.push_back({v->point(), v_helper->point()});
    }

    // Remove a aresta do status
    auto it = std::find(status.begin(), status.end(), hin);
    if (it != status.end())
    {
        status.erase(it);
    }
}

Arrangement::Halfedge_handle find_left_edge(Arrangement::Vertex_handle v, 
                                            const std::vector<Arrangement::Halfedge_handle>& status) {
    
    double vy = to_double(v->point().y());
    double vx = to_double(v->point().x());
    
    Arrangement::Halfedge_handle best_he;
    double max_x = -std::numeric_limits<double>::infinity(); 

    for (auto he : status) {
        auto p1 = he->source()->point();
        auto p2 = he->target()->point();
        
        double x1 = to_double(p1.x());
        double y1 = to_double(p1.y());
        double x2 = to_double(p2.x());
        double y2 = to_double(p2.y());

        double intersect_x;
        
        if (std::abs(y2 - y1) < 1e-9) {
            intersect_x = x1;
        } else {
            intersect_x = x1 + (vy - y1) * ((x2 - x1) / (y2 - y1));
        }

        if (intersect_x < vx && intersect_x > max_x) {
            max_x = intersect_x;
            best_he = he;
        }
    }

    return best_he;
}

void handle_split_vertex(Arrangement::Vertex_handle v, Arrangement& arr) {
    Arrangement::Halfedge_handle h_left = find_left_edge(v, status);

    if (h_left == Arrangement::Halfedge_handle()) return;
    
    if (helper.count(h_left) == 0) return;
    
    Arrangement::Vertex_handle helper_v = helper[h_left]; 
    
    diagonals_to_insert.push_back({v->point(), helper_v->point()}); // coloca diagonala entre v e o helper de ej
    
    helper[h_left] = v; // seta o helper
    
    Arrangement::Halfedge_handle hout = get_incident_edge(v);
    if (hout != Arrangement::Halfedge_handle())
    {
        hout = hout->next();
        status.push_back(hout); // coloca em status
        helper[hout] = v; // seta o helper
    }
}

void handle_merge_vertex(Arrangement::Vertex_handle v, Arrangement& arr)
{
    Arrangement::Halfedge_handle hin = get_incident_edge(v);
    if (hin == Arrangement::Halfedge_handle()) return;
    
    // PROTEÇÃO
    if (helper.count(hin) > 0)
    {
        Arrangement::Vertex_handle v_helper = helper[hin];
        if(classify_vertex(v_helper) == MERGE) // diagonal se o helper for merge        
        {
            diagonals_to_insert.push_back({v->point(), v_helper->point()});
        }
    }

    // renove do status
    auto it = std::find(status.begin(), status.end(), hin);
    if (it != status.end())
    {
        status.erase(it);
    }

    Arrangement::Halfedge_handle h_left = find_left_edge(v, status);
    
    if (h_left != Arrangement::Halfedge_handle() && helper.count(h_left) > 0)
    {
        Arrangement::Vertex_handle helper_v = helper[h_left];
        
        if (classify_vertex(helper_v) == MERGE) {
            diagonals_to_insert.push_back({v->point(), helper_v->point()});
        }
        
        helper[h_left] = v;
    }
}

void handle_regular_vertex(Arrangement::Vertex_handle v, Arrangement& arr)
{
    Arrangement::Halfedge_handle he_in = get_incident_edge(v);
    if (he_in == Arrangement::Halfedge_handle()) return;
    
    Arrangement::Halfedge_handle he_out = he_in->next();
    
    bool interior_is_right = !is_above(v->point(), he_out->target()->point());

    if (interior_is_right) {
        Arrangement::Halfedge_handle h_left = find_left_edge(v, status);
        
        if (h_left != Arrangement::Halfedge_handle() && helper.count(h_left) > 0) 
        {
            Arrangement::Vertex_handle helper_v = helper[h_left]; 
            
            if (classify_vertex(helper_v) == MERGE) {
                diagonals_to_insert.push_back({v->point(), helper_v->point()});
            }
            helper[h_left] = v; 
        }
    } 
    else 
    {
        if (helper.count(he_in) > 0) {
            if (classify_vertex(helper[he_in]) == MERGE) {
                diagonals_to_insert.push_back({v->point(), helper[he_in]->point()});
            }
        }

        auto it = std::find(status.begin(), status.end(), he_in); 
        if (it != status.end()) {
            status.erase(it);
        }
        
        status.push_back(he_out);
        helper[he_out] = v;
    }
}

int main()
{
    Arrangement arr;

    std::vector<Point_2> points;
    points.push_back(Point_2(0, 10));   
    points.push_back(Point_2(8, 10));   
    points.push_back(Point_2(8, 8));     
    points.push_back(Point_2(2, 8));    
    points.push_back(Point_2(2, 6));     
    points.push_back(Point_2(6, 6));    
    points.push_back(Point_2(6, 4));     
    points.push_back(Point_2(2, 4));    
    points.push_back(Point_2(2, 2));     
    points.push_back(Point_2(8, 2));    
    points.push_back(Point_2(8, 0));    
    points.push_back(Point_2(0, 0));

    for (size_t i = 0; i < points.size(); ++i)
    {
        Point_2 p1 = points[i];
        Point_2 p2 = points[(i + 1) % points.size()]; 
        Segment_2 seg(p1, p2); 
        CGAL::insert(arr, seg);
    }


    export_arrangement_svg(arr, "polygon_partitionedantes.svg");


    std::cout << "O arranjo tem " << arr.number_of_vertices() << " vertices e " 
              << arr.number_of_edges() << " arestas." << std::endl;

    std::vector<Arrangement::Vertex_handle> vertices;

    for(auto it = arr.vertices_begin(); it != arr.vertices_end(); it++)
    {
        vertices.push_back(it);
    }

    std::sort(vertices.begin(), vertices.end(), 
        [] (Arrangement::Vertex_handle v1, Arrangement::Vertex_handle v2)
    {
        if(v1->point().y() == v2->point().y())
            return v1->point().x() < v2->point().x();
        return v1->point().y() > v2->point().y();
    }); 

    std::cout << "Iniciando Sweep-Line..." << std::endl;

    helper.clear();
    status.clear();
    diagonals_to_insert.clear();

    for (auto v : vertices) {
        VertexType type = classify_vertex(v);
        
        switch (type) {
            case START:
                handle_start_vertex(v);
                break;
            case END:
                handle_end_vertex(v, arr);
                break;
            case SPLIT:
                handle_split_vertex(v, arr);
                break;
            case MERGE:
                handle_merge_vertex(v, arr);
                break;
            case REGULAR:
                handle_regular_vertex(v, arr);
                break;
        }
    }

    std::cout << "Sweep concluido. Inserindo " << diagonals_to_insert.size() 
              << " diagonais..." << std::endl;

    for (const auto& diag : diagonals_to_insert) {
        Segment_2 seg(diag.p1, diag.p2);
        CGAL::insert(arr, seg);
    }

    std::cout << "Particao concluida." << std::endl;
    std::cout << "O arranjo final tem " << arr.number_of_edges() 
              << " arestas (novas diagonais incluidas)." << std::endl;

    export_arrangement_svg(arr, "polygon_partitioned.svg");
    std::cout << "SVG gerado: polygon_partitioned.svg\n";

    return 0;
}

```


## Resultados

O código também gera uma visualização SVG:


![Desktop View](assets/images/poliantes.png){: width="700" height="400" }
*Polígono antes .*


![Desktop View](assets/images/polidps.png){: width="700" height="400" }
*Polígono depois .*



## Convex Hull - Grahan Scan


{% include embed/video.html src='/assets/videos/receba.mp4' autoplay=true
  loop=true %}


## Problema do Par mais próximo


## Intersecção de Segmentos (números)


## Delaunay

## Voronoi
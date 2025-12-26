---
title: Otimização de Malhas - Mesh Improvement
date: 2025-09-23 09:53:00 -0300
categories: [Computação Gráfica, Geometria Computacional]
tags: [c++, algoritmos, malhas]
math: true
pin: true
image:
  path: /assets/images/trab1.png
  alt: Visualização da malha otimizada
---

## O Desafio: Malhas de Alta Qualidade

A triangulação de um polígono é uma etapa fundamental em diversos problemas de geometria computacional. No entanto, não basta apenas triangular: a "qualidade" dos triângulos gerados impacta diretamente aplicações como o **Método dos Elementos Finitos (FEM)** e processos de interpolação.

O objetivo deste projeto foi implementar um algoritmo capaz de ler um polígono (formato OBJ), gerar uma triangulação inicial ingênua e, em seguida, otimizar essa malha maximizando uma métrica de qualidade através de operações de *Edge Flip*.

Utilizei a biblioteca **CGAL** (através da interface CGL da disciplina) e a estrutura de dados **DCEL (Doubly-Connected Edge List)**.

## Entrada - Formato OBJ

O formato de OBJ é um formato de arquivo aberto para definição de geometrias, definindo a geometria 3D do objeto, seus vértices e faces.

```c

v 67.0 -9.0 0.0
v 41.0 -9.0 0.0
v -28.0 8.0 0.0
v 72.0 41.0 0.0
v -6.0 96.0 0.0
v -81.0 54.0 0.0
v 23.0 57.0 0.0
v -91.0 -7.0 0.0
v -66.0 -28.0 0.0
v -83.0 -49.0 0.0
v 19.0 -50.0 0.0
v 82.0 -64.0 0.0
v 93.0 -61.0 0.0
v 80.0 11.0 0.0
v 74.0 -3.0 0.0
f 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

```

- Vértice $$v$$ indica as coordenadas de um vértice, ordenados no sentido anti-horário.
- Face $$f$$ indica a face composta pelos índices dos vértices

![Desktop View](assets/images/exemplo.png){: width="500" height="300" }
*O Polígono do exemplo  .*

## Critério de Qualidade

Para guiar a otimização, formulei um critério que penaliza triângulos muito "magros" ou distorcidos. A métrica escolhida mede o quão próximo um triângulo é de ser equilátero:

$$
    f = \frac {4 \sqrt{3} A} {a^2 + b^2 + c^2}
$$

Onde:
* $$f$$: Fator de qualidade ($0 \le f \le 1$). $1$ é um triângulo equilátero perfeito.
* $$A$$: Área do triângulo.
* $$a, b, c$$: Comprimentos das arestas.

> **Por que essa métrica?**
> No FEM, triângulos com ângulos muito pequenos aumentam o erro de aproximação e podem tornar o cálculo instável. Esta métrica garante que buscamos triângulos o mais "gordos" possível.

### Implementação da Métrica

Abaixo, o trecho de código em C++ que calcula essa qualidade a partir de uma face da malha:

```c++
void qualidade(CGL::Mesh &mesh, CGL::Mesh::Face_index f)
{
    CGL::Mesh::Halfedge_index h = mesh.halfedge(f);

    CGL::Mesh::Vertex_index v1_idx = mesh.source(h);
    CGL::Mesh::Vertex_index v2_idx = mesh.source(mesh.next(h));
    CGL::Mesh::Vertex_index v3_idx = mesh.source(mesh.next(mesh.next(h)));

    std::cout << v1_idx << " " << v2_idx << " " << v3_idx << std::endl;
    std::cout << mesh.point(v1_idx) << " " << v2_idx.idx() << " " << v3_idx << std::endl;

    CGL::Point3 p1 = mesh.point(v1_idx);
    CGL::Point3 p2 = mesh.point(v2_idx);
    CGL::Point3 p3 = mesh.point(v3_idx);

    std::vector<CGL::Point3> triangulo;
    triangulo.push_back(p1);
    triangulo.push_back(p2);
    triangulo.push_back(p3);

    double a = CGAL::squared_distance(p1,p2);
    double b = CGAL::squared_distance(p2,p3);
    double c = CGAL::squared_distance(p3,p1);


    double area = 0.0;

        
    for(int i = 0 ; i < 3; i++)
    {
        CGL::Point3 p1 = triangulo[i];
        CGL::Point3 p2 = triangulo[(i + 1) % 3];
        area += (p1.x() * p2.y()) - (p1.y() * p2.x());

    }

    area = area /2.0;

    double numerator = 4.0 * std::sqrt(3.0) * area;
    double denominator = a + b + c;

     

    std:: cout<< numerator / denominator << std::endl;
}
```

## Otimização via Flip de Arestas
A otimização ocorre percorrendo as arestas internas da malha. Para cada aresta compartilhada por dois triângulos, verificamos se a operação de Flip (trocar a diagonal do quadrilátero formado) aumentaria a qualidade mínima local.

A operação de troca foi implementada utilizando as operações de Euler `` join_face `` e `` split_face ``
 da CGAL. É crucial verificar se o quadrilátero formado pelos dois triângulos é convexo antes de tentar o flip, caso contrário, a operação geraria uma geometria inválida.

```c++
if(qualidade_depois > qualidade_antes)
            {
                CGL::Mesh::Halfedge_index h3 = mesh.next(h1);
                CGL::Mesh::Halfedge_index h4 = mesh.opposite(h1);
                h4 = mesh.next(h4);

                CGAL::Euler::join_face(h1, mesh);
                CGAL::Euler::split_face(h3, h4, mesh);

                melhoria_global = true; 
            }
```



## Resultados

Os resultados visuais mostram claramente a eliminação de triângulos degenerados



![Desktop View](assets/images/normal.png){: width="500" height="300" }
*O Polígono do exemplo  .*


![Desktop View](assets/images/naive.png){: width="500" height="300" }
*Triangulação Naive .*


![Desktop View](assets/images/melhorado.png){: width="500" height="300" }
*Triangulação melhorada  .*

## Conclusão e Código Completo
Este trabalho demonstrou como operações locais na topologia (Edge Flips), guiadas por uma métrica geométrica simples, podem melhorar drasticamente a qualidade global de uma malha. Essa técnica é a base para algoritmos mais complexos como a Triangulação de Delaunay.

Código Completo : [Ver Código no GitHub](https://github.com/joaoreis28/Trabalho-1-Geometria-Computacional/tree/master){: target="_blank" }

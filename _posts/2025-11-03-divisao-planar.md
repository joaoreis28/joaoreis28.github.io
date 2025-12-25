---
title: Divisão Planar - Compressão de imagens
date: 2025-11-03 09:53:00 -0300
categories: [Geometria Computacional]
tags: [geometria, otimização]     # TAG names should always be lowercase
description: Descrição do trabalho de divisão planar .
math: true
image: /assets/images/trab2.png
---

# Enunciado 
A ideia do trbalho é implementar uma melhora para a triangulação de um polígono lido a partir de um arquivo do tipo __object file__(OBJ) e representado usando uma estrutura do tipo halfedge.


## Estrutura Half-Edge, doubly-connected edge list(DCEL)



## Minha Solução 
O critério que eu formulei para avaliar a qualidade dos triângulos foi :

$$
    f = \dfrac {4 \sqrt{3} A} {a² + b² + c²}
$$

onde $$f$$ varia de 0 (triângulo degenerado) até 1 (triângulo equilátero), $$A$$ é a área do triângulo, e $$(a,b,c)$$ são os tamanhos dos lados do triângulo.

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
A métrica foi escolhida porque ela mede o quanto um triângulo se afasta da forma equilátera, especialmente útil em aplicações como o método dos elementos finitos(FEM) e em processos de interpolação. No FEM, por exemplo, o domínio do problema físico é dividido em pequenos triângulos, e cada um deles serve como base para aproximar a solução de uma equação diferencial. Se esses triângulos forem mal formados, com ângulos muito pequenos ou áreas quase nulas, a aproximação perde precisão e o cálculo pode se tornar instável. Já na interpolação, em que se reconstrói uma função a partir de valores conhecidos nos vértices, triângulos de baixa qualidade distorcem a distribuição da função e aumentam o erro. Por isso, usar $$f$$ como métrica é útil para identificar e evitar esses problemas, garantindo triangulações mais confiáveis e adequadas para diferentes aplicações.


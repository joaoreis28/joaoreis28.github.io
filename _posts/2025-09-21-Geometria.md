---
title: Anotações de Geometria Computacional
date: 2025-09-21 09:53:00 -0300
categories: [Geometria Computacional]
tags: [geometria]     # TAG names should always be lowercase
description: Notas de aula  da disciplina de Geometria computacional.
math: true
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


![Desktop View](assets/images/positivo.png){: width="700" height="400" }
*O vetor $$ \vec{v_2} $$  está à esquerda do vetor $$ \vec{v_1} $$ .*

![Desktop View](assets/images/negativo.png){: width="700" height="400" }
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

### Intersecção Própria
### Intersecção Imprópria
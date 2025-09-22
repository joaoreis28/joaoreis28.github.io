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
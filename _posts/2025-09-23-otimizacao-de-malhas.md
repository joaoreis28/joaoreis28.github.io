---
title: Otimização de Malhas - Mesh Improvement
date: 2025-09-23 09:53:00 -0300
categories: [Geometria Computacional]
tags: [geometria, otimização]     # TAG names should always be lowercase
description: Descrição do trabalho de otimização .
math: true
---

# Enunciado 
A ideia do trbalho é implementar uma melhora para a triangulação de um polígono lido a partir de um arquivo do tipo __object file__(OBJ) e representado usando uma estrutura do tipo halfedge.

## Formato OBJ
O formato de OBJ é um formato de arquivo aberto para definição de geometrias, definindo a geomtria 3D do objeto, seus vértices e faces.

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

- Vertices $$v$$ indica as coordenadas de um vértice, ordenados no sentido anti-horário.
- Faces $$f$$ indica a face composta pelos índices dos vértices


## Estrutura Half-Edge, doubly-connected edge list(DCEL)
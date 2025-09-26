---
title: Entendendo os conceitos de Ray Tracing    
date: 2025-05-12 09:53:00 -0300
categories: [Computação Gráfica]
tags: [computação gráfica]     # TAG names should always be lowercase
description: Explicação dos conceitos presentes no livro Ray tracind in one weekend.
math: true
---
# O que é um Ray Tracing
Ray tracing é uma técnica de computação gráfica usada para renderizar imagens tridimensionais de forma realista, simulando o comportamento da  luz no mundo real. No entanto, para tornar o processo viável computacionalmente, a simulação é realizada de maneira inversa: em vez de seguir cada raio emitido por fontes de luz, os raios são projetados a partir da câmera (observador) em direção à cena, determinando como eles interagem com os objetos até alcançar a fonte de luz.

## Viewport


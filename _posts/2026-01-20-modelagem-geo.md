---
title: "Representações Celulares com Quadtrees: Exercício de Modelagem Geométrica"
date: 2026-01-24 10:00:00 -0300
categories: [IMPA, Computação Visual]
tags: [modelagem-geometrica, quadtree, impa, c++, svg, subdivisao-espacial]
math: true
image : /assets/images/banMG.png

---

## Contexto

Este post documenta a solução de um exercício da disciplina de verão **Temas de Computação Visual - 2026** do IMPA, especificamente sobre **Modelagem Geométrica**. O objetivo é calcular representações celulares uniformes e adaptativas de regiões do plano usando a técnica de subdivisão espacial conhecida como quadtree.

## O Problema

O exercício propõe dois desafios:

### Problema 1: Disco em Região Retangular

Calcular representações celulares uniformes e adaptativas de um disco dentro do quadrado unitário $[0, 1] \times [0, 1]$.

**Parâmetros de referência:**
- Centro: $(0.3, 0.4)$
- Raio: $0.21$

### Problema 2: Região Parabólica

Calcular representações celulares uniformes e adaptativas para a região do plano dada por:

$$y \geq x^2 + c$$

na área retangular $[-2, 2] \times [-2, 2]$.

**Parâmetro de referência:** $c = 0$

## Fundamentação Teórica

### Quadtrees e Subdivisão Espacial

Uma quadtree é uma estrutura hierárquica que particiona recursivamente o espaço bidimensional em quatro quadrantes. No contexto de modelagem geométrica, usamos essa estrutura para criar representações celulares que aproximam regiões definidas por curvas ou formas geométricas.

### Classificação de Células

Cada célula retangular da subdivisão pode ser classificada em três categorias:

- **DENTRO**: a célula está completamente contida na região de interesse
- **FORA**: a célula está completamente externa à região
- **MISTO**: a fronteira da região atravessa a célula

### Modos de Subdivisão

**Subdivisão Uniforme:** todas as células são subdivididas até um nível fixo $n$, gerando $4^n$ células no nível mais refinado.

**Subdivisão Adaptativa:** células classificadas como DENTRO ou FORA não são mais subdivididas, economizando memória e processamento. Apenas células MISTO continuam sendo refinadas.

## Solução Implementada

### Estrutura do Código

Implementei dois programas em C++ que geram visualizações SVG progressivas da subdivisão:

#### 1. Programa para o Disco (`main.cpp`)

**Estrutura de dados:**
```cpp
struct Circulo { double cx, cy, r; };
enum Classificacao { FORA, DENTRO, MISTO };
```

**Algoritmo de classificação:**

O algoritmo usa dois testes geométricos:

1. **Teste de exclusão**: encontra o ponto da célula mais próximo do centro do círculo. Se esse ponto está a uma distância maior que o raio, a célula está FORA.

2. **Teste de inclusão**: verifica se todos os quatro vértices da célula estão dentro do círculo. Se sim, a célula está DENTRO.

3. Se nenhum dos casos acima se aplica, a célula é MISTO.

```cpp
Classificacao classificar(double xmin, double xmax, double ymin, double ymax, Circulo c)
{
    // Ponto mais próximo do círculo dentro do retângulo
    double closestX = std::max(xmin, std::min(c.cx, xmax));
    double closestY = std::max(ymin, std::min(c.cy, ymax));
    double distSq = (closestX - c.cx)*(closestX - c.cx) + 
                    (closestY - c.cy)*(closestY - c.cy);

    if (distSq > c.r * c.r) return FORA;

    // Testa os 4 vértices
    bool tl = ((xmin - c.cx)*(xmin - c.cx) + (ymax - c.cy)*(ymax - c.cy)) <= c.r*c.r;
    bool tr = ((xmax - c.cx)*(xmax - c.cx) + (ymax - c.cy)*(ymax - c.cy)) <= c.r*c.r;
    bool bl = ((xmin - c.cx)*(xmin - c.cx) + (ymin - c.cy)*(ymin - c.cy)) <= c.r*c.r;
    bool br = ((xmax - c.cx)*(xmax - c.cx) + (ymin - c.cy)*(ymin - c.cy)) <= c.r*c.r;

    if (tl && tr && bl && br) return DENTRO;
    return MISTO;
}
```

#### 2. Programa para a Parábola (`parabola.cpp`)

**Estrutura de dados:**
```cpp
struct Parabola { double c; };
```

**Algoritmo de classificação:**

Para classificar uma célula $[x_{min}, x_{max}] \times [y_{min}, y_{max}]$ em relação à região $y \geq x^2 + c$:

1. Calculamos o intervalo de valores de $x^2$ na célula:
   - $$x^2_{max}$$ = $$\max(x^2_{min}, x^2_{max})$$
   - $$x^2_{min} = 0$$ se $x_{min} \leq 0 \leq x_{max}$, caso contrário $$\min(x^2_{min}, x^2_{max})$$

2. Testamos:
   - Se $y_{min} \geq x^2_{max} + c$, a célula está DENTRO
   - Se $y_{max} < x^2_{min} + c$, a célula está FORA
   - Caso contrário, é MISTO

```cpp
Classificacao classificar(double xmin, double xmax, double ymin, double ymax, Parabola p)
{
    double max_x2 = std::max(xmin * xmin, xmax * xmax);

    double min_x2;
    if (xmin <= 0 && xmax >= 0) {
        min_x2 = 0.0;  // x² tem mínimo em x=0
    } else {
        min_x2 = std::min(xmin * xmin, xmax * xmax);
    }

    if (ymin >= max_x2 + p.c) return DENTRO;
    if (ymax < min_x2 + p.c) return FORA;
    return MISTO;
}
```

### Subdivisão Recursiva

Ambos os programas implementam a subdivisão recursiva:

```cpp
void explore(std::ofstream& svg, double xmin, double xmax, double ymin, double ymax, 
             int currentLevel, int targetLevel, /* ... */)
{
    Classificacao cl = classificar(xmin, xmax, ymin, ymax, /* ... */);

    // Modo adaptativo: para em células homogêneas
    if (!uniforme && cl != MISTO) {
        desenharSvgRect(svg, xmin, xmax, ymin, ymax, cl);
        return;
    }

    // Chegou no nível máximo
    if (currentLevel == targetLevel) {
        desenharSvgRect(svg, xmin, xmax, ymin, ymax, cl);
        return;
    }

    // Subdivide em 4 quadrantes
    double xmid = (xmin + xmax) / 2.0;
    double ymid = (ymin + ymax) / 2.0;

    explore(svg, xmin, xmid, ymin, ymid, currentLevel + 1, targetLevel, /* ... */);
    explore(svg, xmid, xmax, ymin, ymid, currentLevel + 1, targetLevel, /* ... */);
    explore(svg, xmin, xmid, ymid, ymax, currentLevel + 1, targetLevel, /* ... */);
    explore(svg, xmid, xmax, ymid, ymax, currentLevel + 1, targetLevel, /* ... */);
}
```

## Visualização

Os programas geram arquivos SVG com o seguinte esquema de cores:

- **Vermelho** (`#ff7f7f`): células DENTRO da região
- **Azul** (`#a0a0ff`): células FORA da região
- **Contorno preto**: células MISTO (modo adaptativo)
- **Cinza**: células MISTO no nível máximo (quando não pode mais refinar)

![Descrição do GIF](/assets/videos/circuloadpa.gif)
*Disco com abordagem adaptativa*

![Descrição do GIF](/assets/videos/circulouni.gif)
*Disco com abordagem uniforme*


![Descrição do GIF](/assets/videos/parabolaAd.gif)
*Parabóla com abordagem adaptativa*

![Descrição do GIF](/assets/videos/parabolauni.gif)
*Parabóla com abordagem uniforme*



### Complexidade Computacional

**Subdivisão Uniforme (nível $n$):**
- Número de células: $4^n$
- Complexidade espacial: $O(4^n)$
- Todas as células têm o mesmo tamanho

**Subdivisão Adaptativa:**
- Número de células: depende da geometria
- Complexidade: $O(k)$ onde $k$ é o número de células MISTO
- Células variam de tamanho conforme a proximidade da fronteira




## Código Completo

### Círculo
```c++
#include <iostream>
#include <fstream>
#include <string>
#include <cmath>
#include <algorithm>
#include <sstream> 

enum Classificacao { FORA, DENTRO, MISTO };

struct Circulo { double cx, cy, r; };

Classificacao classificar(double xmin, double xmax, double ymin, double ymax, Circulo c)
{
    double closestX = std::max(xmin, std::min(c.cx, xmax));
    double closestY = std::max(ymin, std::min(c.cy, ymax));
    double distSq = (closestX - c.cx)*(closestX - c.cx) + (closestY - c.cy)*(closestY - c.cy);

    if (distSq > c.r * c.r) return FORA;

    bool tl = ((xmin - c.cx)*(xmin - c.cx) + (ymax - c.cy)*(ymax - c.cy)) <= c.r*c.r; // Testa se cada um dos pontos do retangulo está dentro da circ
    bool tr = ((xmax - c.cx)*(xmax - c.cx) + (ymax - c.cy)*(ymax - c.cy)) <= c.r*c.r;
    bool bl = ((xmin - c.cx)*(xmin - c.cx) + (ymin - c.cy)*(ymin - c.cy)) <= c.r*c.r;
    bool br = ((xmax - c.cx)*(xmax - c.cx) + (ymin - c.cy)*(ymin - c.cy)) <= c.r*c.r;

    if (tl && tr && bl && br) return DENTRO;
    return MISTO;
}

void desenharSvgRect(std::ofstream& svg, double xmin, double xmax, double ymin, double ymax, Classificacao c)
{
    std::string corFill, corStroke = "#555555";
    double strokeWidth = 0.002; 

    switch (c)
        {
        case DENTRO: corFill = "#ff7f7f"; break; 
        case FORA:   corFill = "#a0a0ff"; break; 
        default:     corFill = "none"; break;    
    }
    
    if (c == MISTO) {
         svg << "<rect x=\"" << xmin << "\" y=\"" << ymin 
             << "\" width=\"" << xmax-xmin << "\" height=\"" << ymax-ymin 
             << "\" fill=\"none\" stroke=\"black\" stroke-width=\"" << strokeWidth/2.0 << "\"/>\n";
    } else {
         svg << "<rect x=\"" << xmin << "\" y=\"" << ymin 
             << "\" width=\"" << xmax-xmin << "\" height=\"" << ymax-ymin 
             << "\" fill=\"" << corFill 
             << "\" stroke=\"" << corStroke << "\" stroke-width=\"" << strokeWidth << "\"/>\n";
    }
}


void exploreUniforme(std::ofstream& svg, double xmin, double xmax, double ymin, double ymax, int currentLevel, int targetLevel, Circulo c)
{
    if (currentLevel == targetLevel)
    {
        
        Classificacao cl = classificar(xmin, xmax, ymin, ymax, c);
        
        desenharSvgRect(svg, xmin, xmax, ymin, ymax, cl);
        return;
    }

  

    double xmid = (xmin + xmax) / 2.0;
    double ymid = (ymin + ymax) / 2.0;

    exploreUniforme(svg, xmin, xmid, ymin, ymid, currentLevel + 1, targetLevel, c);
    exploreUniforme(svg, xmid, xmax, ymin, ymid, currentLevel + 1, targetLevel, c);
    exploreUniforme(svg, xmin, xmid, ymid, ymax, currentLevel + 1, targetLevel, c);
    exploreUniforme(svg, xmid, xmax, ymid, ymax, currentLevel + 1, targetLevel, c);
}

void explore(std::ofstream& svg, double xmin, double xmax, double ymin, double ymax, int currentLevel, int targetLevel, Circulo c)
{
    Classificacao cl = classificar(xmin, xmax, ymin, ymax, c);

    if (cl != MISTO) {
        desenharSvgRect(svg, xmin, xmax, ymin, ymax, cl);
        return;
    }




    if (currentLevel == targetLevel)
    {
        svg << "<rect x=\"" << xmin << "\" y=\"" << ymin 
            << "\" width=\"" << xmax-xmin << "\" height=\"" << ymax-ymin 
            << "\" fill=\"#cccccc\" stroke=\"black\" stroke-width=\"0.002\"/>\n";
        return;
    }

    double xmid = (xmin + xmax) / 2.0;
    double ymid = (ymin + ymax) / 2.0;

    explore(svg, xmin, xmid, ymin, ymid, currentLevel + 1, targetLevel, c);
    explore(svg, xmid, xmax, ymin, ymid, currentLevel + 1, targetLevel, c);
    explore(svg, xmin, xmid, ymid, ymax, currentLevel + 1, targetLevel, c);
    explore(svg, xmid, xmax, ymid, ymax, currentLevel + 1, targetLevel, c);
}

int main() {
    double mundoSize = 1.0;
    double imageSizePixels = 800.0;
    Circulo meuCirculo = {0.3, 0.6, 0.21}; // coordenada y invertida
    int maxNiveis = 7; 


    for (int i = 0; i <= maxNiveis; i++)
    {
        
        std::stringstream ss;
        ss << "quadtree_passo_" << i << ".svg";
        std::string nomeArquivo = ss.str();

        std::ofstream svg;
        svg.open(nomeArquivo);

         svg << "<svg width=\"" << imageSizePixels << "\" height=\"" << imageSizePixels 
            << "\" viewBox=\"0 0 " << mundoSize << " " << mundoSize 
            << "\" xmlns=\"http://www.w3.org/2000/svg\">\n";
        
        svg << "<rect width=\"100%\" height=\"100%\" fill=\"white\"/>\n";

        svg << "<circle cx=\"" << meuCirculo.cx << "\" cy=\"" << meuCirculo.cy 
            << "\" r=\"" << meuCirculo.r << "\" fill=\"none\" stroke=\"#00000033\" stroke-width=\"0.005\"/>\n";

        exploreUniforme(svg, 0, mundoSize, 0, mundoSize, 0, i, meuCirculo);

        svg << "</svg>";
        svg.close();
        
        std::cout << "Gerado: " << nomeArquivo << " (Nivel " << i << ")\n";
    }

    return 0;
}
```

### Parabóla

---


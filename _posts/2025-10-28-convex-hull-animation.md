---
title: Convex Hull Animation - Grahan Scan
date: 2025-10-28 09:53:00 -0300
categories: [Geometria Computacional]
tags: [geometria]     # TAG names should always be lowercase
description: Implementação da simulação do algoritmo de fecho convexo Grahan Scan com a biblioteca SFML.
math: true
---

## Explicação


```c++
#include <SFML/Graphics.hpp>
#include <GL/gl.h>
#include <vector>
#include <string>
#include <fstream>
#include <iostream>
#include <cmath>
#include <climits>

int left(sf::Vector2f p, sf::Vector2f  q, sf::Vector2f  r)
{
   return ( (q.x - p.x) * (r.y - p.y)  - (r.x - p.x) * (q.y - p.y)  >= 0 ); 
}
 
 
 
int colinear(sf::Vector2f  p, sf::Vector2f  q, sf::Vector2f  r)
{
   return ( (q.x - p.x) * (r.y - p.y)  - (r.x - p.x) * (q.y - p.y)  == 0 ); 
}
 
 
double Angulo(sf::Vector2f  p, sf::Vector2f  q) {
   return (atan2( q.y - p.y, q.x - p.x)) ;
}

sf::VertexArray ler_pontos()
{

    std::ifstream arquivo("entrada.txt");
    int tamanho = 0;
    arquivo >> tamanho;

    sf::VertexArray pontos(sf::Points, tamanho);

    int i = 0;
    float x, y;
    while( arquivo >> x >> y)
    {
        pontos[i] = sf::Vertex(sf::Vector2f(x, y), sf::Color::Red);
        i++;
    }
    
    arquivo.close();


    return pontos;
}


std::vector<sf::Vector2f> grahan_scan(sf::VertexArray& conjunto, int tamanho)
{
    float miny, xatual;
    miny = LLONG_MAX;
    xatual = LLONG_MAX;
    sf::Vector2f minimo(miny, xatual);
    for(int i = 0; i < tamanho; i++)
    {
        if(conjunto[i].position.y == miny)
            if(conjunto[i].position.x < minimo.x)
                minimo = {conjunto[i].position.x , conjunto[i].position.y };
        if(conjunto[i].position.y < miny)
        {
            minimo = {conjunto[i].position.x , conjunto[i].position.y };
            miny = conjunto[i].position.y;
        }
    }

    std::cout << minimo.x << " " << minimo.y << std::endl;

    std::vector<sf::Vertex> vertices;
    for(size_t i = 0; i < conjunto.getVertexCount(); ++i)
        vertices.push_back(conjunto[i]);



    sort(vertices.begin(), vertices.end(), [&](const sf::Vertex &a, const sf::Vertex &b) {
    double angA = Angulo(minimo, a.position);
    double angB = Angulo(minimo, b.position);
    if (angA == angB) {
        double da = (a.position.x - minimo.x)*(a.position.x - minimo.x) + (a.position.y - minimo.y)*(a.position.y - minimo.y);
        double db = (b.position.x - minimo.x)*(b.position.x - minimo.x) + (b.position.y - minimo.y)*(b.position.y - minimo.y);
        return da < db;
    }
    return angA < angB;
    });
    
    conjunto.clear(); 
    for (const sf::Vertex& v : vertices) {
        conjunto.append(v); 
    }

    std::cout << conjunto[0].position.x << " " << conjunto[0].position.y << std::endl;

    std::vector <sf::Vector2f> S;


    for(int i = 0; i < conjunto.getVertexCount(); i++)
    {
        while(S.size() > 1  && !left(S[S.size() - 2], S.back(), conjunto[i].position))
            S.pop_back();
        S.push_back(conjunto[i].position);
    }
    std::cout << S.size() << std::endl;


    return S;

}



int main()
{
    sf::RenderWindow window(sf::VideoMode(800, 600), "Plano Cartesiano");

    window.setFramerateLimit(60);


    sf::View planoCartesiano(sf::FloatRect(0,0, 800, 600)); // Configurar o Plano Cartesiano
    planoCartesiano.setCenter(0.f, 0.f);
    planoCartesiano.setSize(800.f, -600.f);


    sf::VertexArray pontos = ler_pontos();    
    std::vector <sf::Vector2f> fecho = grahan_scan(pontos, pontos.getVertexCount());
    int tamanho = (int)fecho.size();
    sf::Vertex lista_fecho[tamanho];



    for(int i = 0; i < fecho.size(); i++ )
    {
        lista_fecho[i] = sf::Vertex(sf::Vector2f(fecho[i].x, fecho[i].y), sf::Color::Green);

    }

    sf::VertexArray linhas_tracadas(sf::PrimitiveType::LinesStrip);


    for(int i = 0; i < fecho.size(); i++ ) linhas_tracadas.append(lista_fecho[i]);
    linhas_tracadas.append(lista_fecho[0]);


    int flag = 0;
    int flag2 = 0;
    while (window.isOpen())
    {
        sf::Event event;
        while (window.pollEvent(event))
        {
            if (event.type == sf::Event::Closed)
                window.close();

            if(event.type == sf::Event::KeyPressed)
            {
                if(event.key.code == sf::Keyboard::A)
                {
                    flag = 1;
                }
                if(event.key.code == sf::Keyboard::B)
                {
                    flag2 = 1;
                }
            }
        }

        window.clear(sf::Color::White);

        window.setView(planoCartesiano); // Configurar o Plano Cartesiano

       

        sf::Vertex lista_vetor[2] = 
        {
            sf::Vertex(sf::Vector2f(200.f, 200.f), sf::Color::Green),
            sf::Vertex(sf::Vector2f(100.f, 90.f), sf::Color::Green),

        };


        glPointSize(10.0f);
        window.draw(pontos);
        if(flag)
            window.draw(lista_fecho, 8, sf::Points);
        if(flag2)
            window.draw(linhas_tracadas);


        window.display();
    }

    return 0;
}
```


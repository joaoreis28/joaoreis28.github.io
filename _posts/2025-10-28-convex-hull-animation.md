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


sf::VertexArray grahan_scan(sf::VertexArray& conjunto, int tamanho)
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



    return conjunto;

}



int main()
{
    sf::RenderWindow window(sf::VideoMode(800, 600), "Plano Cartesiano");

    window.setFramerateLimit(60);


    sf::View planoCartesiano(sf::FloatRect(0,0, 800, 600)); // Configurar o Plano Cartesiano
    planoCartesiano.setCenter(0.f, 0.f);
    planoCartesiano.setSize(800.f, -600.f);



    /////////////////////////////////////////////////////////////////////////

    sf::VertexArray linhas_tracadas(sf::PrimitiveType::LinesStrip);// arestas a desenhar
    sf::VertexArray pontos_do_fecho(sf::PrimitiveType::Points);// arestas a desenhar


    sf::VertexArray pontos = ler_pontos();  // Ler a entrada

    sf::VertexArray pontos_ordenados_por_angulo_polar = grahan_scan(pontos, pontos.getVertexCount());

    for(int i = 0; i < pontos_ordenados_por_angulo_polar.getVertexCount(); i++)
    {
        std::cout << pontos_ordenados_por_angulo_polar[i].position.x << " " << pontos_ordenados_por_angulo_polar[i].position.x << std::endl;
    }
 



    // int flag = 0;
    int flag2 = 0;
    int pressionado = 0;
    int i = 0;
    int ultimo = 0;

    std::vector <sf::Vector2f> S;


  

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
                    //flag = 1;
                }
                if(event.key.code == sf::Keyboard::B)
                {
                    flag2 = 1;
                }
                if(event.key.code == sf::Keyboard::J)
                {
                    pressionado = 1;
                }
            }
        }

        window.clear(sf::Color::Black);

        window.setView(planoCartesiano); // Configurar o Plano Cartesiano

   
    for(; i < pontos_ordenados_por_angulo_polar.getVertexCount() && pressionado; i++)
     {
         while(S.size() > 1  && !left(S[S.size() - 2], S.back(), pontos_ordenados_por_angulo_polar[i].position))
         {
            S.pop_back();
            if (linhas_tracadas.getVertexCount() > 0)
            {
            linhas_tracadas.resize(linhas_tracadas.getVertexCount() - 1);
            //linhas_tracadas.resize(pontos_do_fecho.getVertexCount() - 1);

            }

         }
         S.push_back(pontos_ordenados_por_angulo_polar[i].position);
         linhas_tracadas.append(sf::Vector2f(pontos_ordenados_por_angulo_polar[i].position.x, pontos_ordenados_por_angulo_polar[i].position.y));
         pontos_do_fecho.append(sf::Vertex(sf::Vector2f(pontos_ordenados_por_angulo_polar[i].position.x, pontos_ordenados_por_angulo_polar[i].position.y), sf::Color::Green));


         std::cout << "receba" << std::endl;
         std::cout << linhas_tracadas.getVertexCount() << std::endl;
         pressionado = 0;
     }
     

        if(i == pontos_ordenados_por_angulo_polar.getVertexCount())
            linhas_tracadas.append(sf::Vector2f(pontos_ordenados_por_angulo_polar[0].position.x, pontos_ordenados_por_angulo_polar[0].position.y));
        

        glPointSize(10.0f);
        window.draw(pontos);
        // if(flag)
        //     window.draw(pontos_do_fecho, 8, sf::Points);
        //if(flag2)
        window.draw(linhas_tracadas);
        window.draw(pontos_do_fecho);



        window.display();
    }

    return 0;
}
```


---
title: "Solução do Problema Point in Polygon do CSES"
date: 2025-10-03 10:00:00 +0000
categories: [CSES]
tags: [problema, programação-competitiva, geometria]
description: Resolução problema Point in Polygon do CSES.
math : true
---
# Problem
You are given a polygon of $$n$$ vertices and a list of $$m$$ points. Your task is to determine for each point if it is inside, outside or on the boundary of the polygon.
The polygon consists of $$n$$ vertices $$(x_1,y_1),(x_2,y_2),\dots,(x_n,y_n)$$. The vertices $$(x_i,y_i)$$ and $$(x_{i+1},y_{i+1})$$ are adjacent for $$i=1,2,\dots,n-1,$$ and the vertices $$(x_1,y_1)$$ and $$(x_n,y_n)$$ are also adjacent.
## Input
The first input line has two integers $$n$$ and $$m$$: the number of vertices in the polygon and the number of points.
After this, there are $$n$$ lines that describe the polygon. The ith such line has two integers $$x_i$$ and $$y_i$$.
You may assume that the polygon is simple, i.e., it does not intersect itself.
Finally, there are $$m$$ lines that describe the points. Each line has two integers $$x$$ and $$y$$.

## Output
For each point, print "INSIDE", "OUTSIDE" or "BOUNDARY".
```c++
#include <bits/stdc++.h>
using namespace std;
 
#define int long long
  
struct Ponto
{
    int x;
    int y;
};
 
 
int colinear(Ponto p, Ponto q, Ponto r)
{
   return ( (q.x - p.x) * (r.y - p.y)  - (r.x - p.x) * (q.y - p.y)  == 0 ); 
}
  
bool onSegment(Ponto p, Ponto q, Ponto r) {
    if (r.x <= max(p.x, q.x) && r.x >= min(p.x, q.x) &&
        r.y <= max(p.y, q.y) && r.y >= min(p.y, q.y)) {
        return true;
    }
    return false;
}
 

// calcula o ângulo de rotação do vetor vetor origem-p para o vetor origem-q
double Angulo(Ponto p, Ponto q) // resultado negativo indica que o caminho mais curto é no sentido horário
{
    double dtheta, t1, t2;
 
    t1 = atan2(p.y, p.x);
    t2 = atan2(q.y, q.x);
    dtheta = t2 - t1;
 
    while (dtheta > M_PI)
      dtheta -= M_PI*2;
   while (dtheta < -M_PI)
      dtheta += M_PI*2;
 
   return(dtheta);
}
 

void solve()
{
    int n, m;
    cin >> n >> m;
    vector <Ponto> poligono;
    vector <Ponto> pontos;
 
    for(int i = 0 ;  i< n; i++)
    {
        int x, y;
        cin >> x >> y;
        Ponto p = {x,y};
        poligono.push_back(p);
    }
 
    for(int j = 0; j < m; j++)
    {
        int x, y;
        cin >> x >> y;
        Ponto p1 = {x, y};
        pontos.push_back(p1);
    }
 
    for(auto ponto : pontos)
    {
 
        int flag = 0;
        for(int i = 0; i < n; i++)
        {
            Ponto p3, p4;
            p3 = poligono[i];
            p4 = poligono[(i+1) % n];
            if(colinear(p3,p4,ponto) && onSegment(p3,p4,ponto) && !flag)
            {
                cout << "BOUNDARY" << endl;
                flag = 1;
            }
        }
 
 
        if(!flag)
        {
        double angulo = 0;
        Ponto p1,p2;
 
 
        for(int i = 0; i < n ;i++)
        {
            p1.x = poligono[i].x - ponto.x;
            p1.y = poligono[i].y - ponto.y;
            p2.x = poligono[(i+1)%n].x - ponto.x;
            p2.y = poligono[(i+1)%n].y - ponto.y;
            angulo += Angulo(p1, p2);
        }
 
        if (abs(angulo) < M_PI)
            cout << "OUTSIDE" << endl;
        else
            cout << "INSIDE" << endl;
    }
    
 
    }
   
}
 
 
signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
 
    int t = 1;
    while (t--)
    {
        solve();
        
    }
 
    return 0;
}
```
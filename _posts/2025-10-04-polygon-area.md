---
title: "Solução do Problema Polygon Area do CSES"
date: 2025-10-03 10:00:00 +0000
categories: [CSES]
tags: [problema, programação-competitiva]
description: Resolução problema Polygon Area do CSES.
math : true
---
# Problem
Your task is to calculate the area of a given polygon.
The polygon consists of n vertices $$(x_1,y_1),(x_2,y_2),\dots,(x_n,y_n)$$. The vertices $$(x_i,y_i)$$ and $$(x_{i+1},y_{i+1})$$ are adjacent for $$i=1,2,\dots,n-1$$, and the vertices $$(x_1,y_1)$$ and $$(x_n,y_n)$$ are also adjacent.

## Input
The first input line has an integer n: the number of vertices.
After this, there are n lines that describe the vertices. The ith such line has two integers $$x_i$$ and $$y_i$$.
You may assume that the polygon is simple, i.e., it does not intersect itself.
## Output
Print one integer: $$2a$$ where the area of the polygon is a (this ensures that the result is an integer).

```c++
#include <bits/stdc++.h>
using namespace std;
 
#define int long long
 
 
struct Ponto
{
    int x;
    int y;
};
  
void solve()
{
    int n;
    cin >> n;
    vector <Ponto> poligono;
    for(int i = 0 ;  i< n; i++)
    {
        int x, y;
        cin >> x >> y;
        Ponto p = {x,y};
        poligono.push_back(p);
    }
 
 
    int area = 0.0;
 
    for(int i = 0; i < n; i++)
    {
        Ponto p1 = poligono[i];
        Ponto p2 = poligono[(i + 1) % n];
        area += (p1.x * p2.y) - (p1.y * p2.x);
    }
 
    if(area < 0)
        cout << area * -1 << endl;
    else
        cout << area << endl;  
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
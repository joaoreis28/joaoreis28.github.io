---
title: "Solução do Problema Line Segment Intersection do CSES"
date: 2025-10-03 10:00:00 -0300
categories: [CSES]
tags: [problema, programação-competitiva, geometria]
description: Resolução problema Line Segment Intersection do CSES.
math : true 
---

# Problem
There are two line segments: the first goes through the points $$(x_1,y_1)$$ and $$(x_2,y_2)$$, and the second goes through the points $$(x_3,y_3)$$ and $$(x_4,y_4)$$.
Your task is to determine if the line segments intersect, i.e., they have at least one common point.

## Input
The first input line has an integer $$t$$: the number of tests.
After this, there are $$t$$ lines that describe the tests. Each line has eight integers $$x_1, y_1, x_2, y_2, x_3, y_3, x_4$$ and $$y_4$$.
## Output
For each test, print "YES" if the line segments intersect and "NO" otherwise.


```c++
#include <bits/stdc++.h>
using namespace std;
 
#define int long long
 
struct Ponto
{
    int x;
    int y;
};
  
bool onSegment(Ponto p, Ponto q, Ponto r) {
    if (r.x <= max(p.x, q.x) && r.x >= min(p.x, q.x) &&
        r.y <= max(p.y, q.y) && r.y >= min(p.y, q.y)) {
        return true;
    }
    return false;
}
int noMeio(Ponto p, Ponto q, Ponto r)
{
    int maxX = max(p.x, q.x);
    int minX = min(p.x, q.x);
    int maxy = max(p.y, q.y);
    int miny = min(p.y, q.y);
    if ((r.x > minX) && (maxX > r.x)   &&  (r.y > miny) && (maxy > r.y))
        return 1;
    else
        return 0;      
 
}
 
int left(Ponto p, Ponto q, Ponto r)
{
   return ( (q.x - p.x) * (r.y - p.y)  - (r.x - p.x) * (q.y - p.y)  > 0 ); 
}
 
int colinear(Ponto p, Ponto q, Ponto r)
{
   return ( (q.x - p.x) * (r.y - p.y)  - (r.x - p.x) * (q.y - p.y)  == 0 ); 
}
 

void solve()
{
    int x1,x2,x3,y1,y2,y3, x4, y4;
 
    cin >> x1 >> y1 >> x2 >> y2 >> x3 >> y3 >> x4 >> y4;
 
    Ponto a = {x1,y1};
    Ponto b = {x2,y2};
    Ponto c = {x3,y3};
    Ponto d = {x4,y4};
 
 
    if((a.x == c.x && a.y == c.y) ||
        (a.x == d.x && a.y == d.y ) ||
        (b.x == c.x && b.y == c.y) ||
        (b.x == d.x && b.y == d.y)  )
        {
            cout << "YES" << endl;
            return;
        }
    
 
    if(colinear(a,b,c) && onSegment(a,b,c))
    {
        cout << "YES" << endl;
        return;
    }
    if(colinear(a,b,d) && onSegment(a,b,d))
    {
        cout << "YES" << endl;
        return;
    }
 
    if(colinear(c,d,a) && onSegment(c,d,a))
    {
        cout << "YES" << endl;
        return;
    }
    if(colinear(c,d,b) && onSegment(c,d,b))
    {
        cout << "YES" << endl;
        return;
    }
 
    int pri = left(a, b, c) ^ left (a, b, d);
    int sec = left(c, d, a) ^ left (c, d, b);
 
    if(pri && sec)
        cout << "YES" << endl;
    else
        cout << "NO" << endl;   
}
 

signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
 
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
 
    return 0;
}
```

---
title: "Solução do Problema Point Location Test do CSES"
date: 2025-10-03 10:00:00 +0000
categories: [CSES]
tags: [problema, programação-competitiva]
description: Resolução problema Point Location Test do CSES.
math : true
---
# Problem
There is a line that goes through the points $$p_1=(x_1,y_1)$$ and $$p_2=(x_2,y_2)$$. There is also a point $$p_3=(x_3,y_3)$$.
Your task is to determine whether $$p_3$$ is located on the left or right side of the line or if it touches the line when we are looking from $$p_1 to p_2$$.

## Input
The first input line has an integer $$t$$: the number of tests.
After this, there are $$t$$ lines that describe the tests. Each line has six integers: $$x_1, y_1, x_2, y_2, x_3 and y_3 $$.
## Output
For each test, print "LEFT", "RIGHT" or "TOUCH".

```c++
#include <bits/stdc++.h>
using namespace std;
 
#define int long long
  
struct Ponto
{
    int x;
    int y;
};
  
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
    int x1,x2,x3,y1,y2,y3;
 
    cin >> x1 >> y1 >> x2 >> y2 >> x3 >> y3 ;
 
    Ponto p = {x1,y1};
    Ponto q = {x2,y2};
    Ponto r = {x3,y3};
 
    if(left(p,q,r))
        cout << "LEFT" << endl; 
    else if(colinear(p, q, r))
         cout << "TOUCH" << endl; 
    else
         cout << "RIGHT" << endl; 
         
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
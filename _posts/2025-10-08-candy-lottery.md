---
title: "Solução do Problema Candy Lottery do CSES"
date: 2025-10-08 10:00:00 +0000
categories: [CSES]
tags: [problema, programação-competitiva, math]
description: Resolução problema Candy Lottery do CSES.
math : true
---
# Problem
There are $$n$$ children, and each of them independently gets a random integer number of candies between 1 and $$k$$.
What is the expected maximum number of candies a child gets?
## Input
The only input line contains two integers $$n$$ and $$k$$.
## Output
Print the expected number rounded to six decimal places (rounding half to even).
```c++

#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef long double ld;
 
int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);
    cout.tie(0);
 
    ll n;
    int k; 
    cin >> n >> k;
 
    ld ans = 0;
    for (int i = 1; i <= k; i++)
    {
        
        ld prob_max_menor_que_i = pow((ld)(i - 1) / k, n);
        
        ans += (1.0 - prob_max_menor_que_i);
    }
    
    cout << fixed << setprecision(6) << ans;
    return 0;
}

```
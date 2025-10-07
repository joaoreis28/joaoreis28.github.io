---
title: "Solução do Problema Christmas Party do CSES"
date: 2025-10-07 10:00:00 +0000
categories: [CSES]
tags: [problema, programação-competitiva, math]
description: Resolução problema Christmas Party do CSES.
math : true
---
# Problem
There are $$n$$ children at a Christmas party, and each of them has brought a gift. The idea is that everybody will get a gift brought by someone else.
In how many ways can the gifts be distributed?
## Input
The only input line has an integer $$n$$: the number of children.
## Output
Print the number of ways modulo 10^9+7.
```c++

#include <bits/stdc++.h>
using namespace std;
#define MOD 1000000007
#define int long long

void solve()
{
    int n;
    cin >> n;
    int vec[n + 1];
    vec[1] = 0;
    vec[2] = 1;

    for(int i = 3; i <= n; i++) // derangement formula
        vec[i] = (((vec[i-1] + vec[i-2]) % MOD) * (i -1)) % MOD;

    cout << vec[n] << endl;
}


signed main()
{
    int t;
    t = 1;
    while(t--)
    {
        solve();
    }

}

```
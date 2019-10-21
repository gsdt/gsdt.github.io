---
layout: post
title: "LARMY SPOJ solution"
description: "LARMY SPOJ solution"
categories: [dynamic-programming, divide-and-conquer]
tags: [algorithm, spoj]
redirect_from:
  - /2019/10/21/
---

# 1. Problem
- Name: LARMY - Lannister Army
- Time limit: 5s
- Memory limit: 1.5G
- Source: [SPOJ](https://www.spoj.com/problems/LARMY/)


# 2. Solution

## a. Simple idea
We have recurrence formular:
$$dp[i][j] = min\{dp[i-1][k-1] + cost(k,j)\}$$ with ($$k \leq j$$).

Please notice that $$dp[i][j]$$ is the lowest possible unhappiness of the warriors if we break firt $$j$$ of them in $$i$$ rows
and $$cost[i][j]$$ is unhappiness of warrior in row formed by taking continous segment from $$i$$ to $$j$$.

## b. Optimize cost function
We need to make cost function takes $$O(1)$$ time. There are may approaches to solve this problem. Now we consider $$d[i][j] = 1$$ if warrior at possition $$i$$ is taller than warrior at $$j$$ else $$d[i][j] = 0$$. Value of $$cost[i][j]$$ now equal to sum of value in maxtrix start from $$d[i][i]$$ to $$d[j][j]$$.

We can pre-compute value of array $$d$$ in $$O(n^2)$$ times.

~~~ cpp
void init() {
    memset(d, 0, sizeof d);
    for(int i=1; i<=N; i++) {
        for(int j=1; j<=i; j++) {
            d[i][j] = h[j] > h[i];
        }
    }

    for(int i=1; i<=N; i++) {
        for(int j=1; j<=N; j++) {
            d[i][j] = d[i-1][j] + d[i][j-1] - d[i-1][j-1] + d[i][j];
        }
    }
}

int cost(int i, int j) {
    return d[j][j] - d[i-1][j] - d[j][i-1] + d[i-1][i-1];
}
~~~

## c. Optimize dynamic programing using divided and conquer
For this optimization you can view more detail at: [cp-algorithms](https://cp-algorithms.com/dynamic_programming/divide-and-conquer-dp.html)

## d. More optimize for bypass TLE vedict.
Notice that we need to find optimize value of $$k$$ so that $$dp[i][j]$$ has best value. The value of $$k$$ now must be equal or greater than value of $$j$$ (number of warrior to consider until now.)

# 3. Full solution code
This code run in ~ 3.5 seconds on SPOJ: [https://www.spoj.com/status/LARMY,giaosudauto/](https://www.spoj.com/status/LARMY,giaosudauto/)
~~~ cpp
#include <bits/stdc++.h>
using namespace std;

const int maxN = 5000;

int N, K; 
int h[maxN+1];
vector<int> dp_cur(maxN + 1), dp_before(maxN+1);
int d[maxN+1][maxN+1];

int cost(int i, int j) {
    return d[j][j] - d[i-1][j] - d[j][i-1] + d[i-1][i-1];
}

void compute(int l, int r, int optl, int optr) {
    if(l > r) return ;
    int mid = (l+r) / 2;

    pair<int, int> best = {INT_MAX, -1};
    for(int k=optl; k<=min(mid, optr); k++) {
        best = min(best, {dp_before[k-1] + cost(k, mid) , k});
    }

    dp_cur[mid] = best.first;
    int opt = best.second;

    compute(l, mid-1, optl, opt);
    compute(mid+1, r, opt, optr);
}

void init() {
    memset(d, 0, sizeof d);
    for(int i=1; i<=N; i++) {
        for(int j=1; j<=i; j++) {
            d[i][j] = h[j] > h[i];
        }
    }

    for(int i=1; i<=N; i++) {
        for(int j=1; j<=N; j++) {
            d[i][j] = d[i-1][j] + d[i][j-1] - d[i-1][j-1] + d[i][j];
        }
    }

    for(int i=1; i<=N; i++) {
        dp_before[i] = cost(1, i);
    }
}

int main()
{
    scanf("%d%d", &N, &K);
    for(int i=1; i<=N; i++) scanf("%d", &h[i]);
    init();

    for(int i=2; i<=K; i++) {
        compute(i, N, i, N);
        dp_before = dp_cur;
    }

    printf("%d", dp_before[N]);
    return 0;
}
~~~

Do you have any question? Please comment below.
---
layout: post
title: "RSA Attack: Stereotyped message attack"
description: "This post introduce how to exploit Stereotyped message attack using sagemath."
categories: [rsa, crypto, sagemath]
tags: [tutorial]
redirect_from:
  - /2018/07/20/
---

* Kramdown table of contents
{:toc .toc}

# Problem
The message has format:

> Your PIN code is XXXX

Someone encrypt this message using RSA algorithm. You know cipher text and public key (N, e=3).

How to recover original message?

# Explain Attack

We replace `X` by `b'\x00'`, the we have \\( (m + x)^e \mod n = c \\) with:

\\(m\\) is message after replace with `b'\x00'`

\\(x\\) is `XXXX` we need to find

\\( (e, n) \\) is a part of public key

\\(c\\) is cipher text

The problem now is how to find \\(x\\)?

Coppersmith has found a way solve this problem, more detail you can found [here](https://github.com/mimoo/RSA-and-LLL-attacks/blob/master/survey_final.pdf):

# Condition for attack success

* \\(e = 3\\)
* \\(x < N^\frac{1}{e} \\)

# Implementation

Simple way to implement in coding is use SageMath. To run code `sage exploit.sage`

{% highlight python linenos=table %}
# File: exploit.sage

import time
import sys

# You will not known this
msg = b'Your PIN code is 4394'

def long_to_bytes(data):
    data = str(hex(long(data)))[2:-1]
    return "".join([chr(int(data[i:i + 2], 16)) for i in range(0, len(data), 2)])
    
def bytes_to_long(data):
    return int(data.encode('hex'), 16)

def main():
    e,N = (3L, 101100845141156293469516586973179461987930689009763964117872470309684853512775295312081121501322683984914454311655512983781714534411655378725344931438891842226528067586198216797211681076517718505980665732445770547794541814618131322049740520275847849231052080791884055178607671253203354019327951368529475389269L)

    c = bytes_to_long(msg)**e % N
    m = bytes_to_long("Your PIN code is \x00\x00\x00\x00")
    P.<x> = PolynomialRing(Zmod(N), implementation='NTL')
    pol = (m + x)^e - c
    roots = pol.small_roots(epsilon=1/30)
    print("Potential solutions:")
    for root in roots:
       print(root, long_to_bytes(m+root))
	
main()
{% endhighlight %}
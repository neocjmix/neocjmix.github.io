---
layout: post
title: "Dynamic Programming"
description: ""
category: "algorithm"
tags: []
---
{% include JB/setup %}

##Fibonacci

###Recursive

```java
public int recurrsive(int num){
    if(num < 2) return num;
    return recurrsive(num-1) + recurrsive(num-2);
}
```

###Memoization

```java
long[] resolved = new long[1000];
    
public long memoization(int num){
    if(resolved[num]==0){
        if(num < 2) resolved[num] = num;
        else resolved[num] = resolved[num-1] + resolved[num-2];
    }
    
    return resolved[num];
}
```

###Bottom-up

```java
public long bottomUp(int num) {
    long[] resolved = new long[1000];
    
    resolved[0] = 0;
    resolved[1] = 1;
    
    for (int i = 2; i <= num; i++){
        resolved[i] = resolved[i-2] + resolved[i-1];
    }
    
    return resolved[num];
}
```

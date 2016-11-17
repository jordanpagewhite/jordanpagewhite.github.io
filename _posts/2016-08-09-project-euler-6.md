---
layout: post
title: Project Euler 6
category: project-euler
---

```javascript
function sumSquares(n) {
    sum = 0;

    for (i = 1; i <= n; i++) {
        sum += Math.pow(i,2);  
    }

    return sum;
}

function squareSums(n) {
    sum = 0;

    for (i = 1; i <= n; i++) {
        sum += i;
    }

    return Math.pow(sum,2);
}

console.log("The answer is: " + String(squareSums(100) - sumSquares(100)));
```

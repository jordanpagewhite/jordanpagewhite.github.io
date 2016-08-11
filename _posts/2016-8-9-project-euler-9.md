---
layout: post
title: Project Euler 9
category: project-euler
---

```javascript
// Natural numbers a,b,c such that a < b < c
function isPythagoreanTriplet(a, b, c) {
    return Math.pow(a,2) + Math.pow(b,2) == Math.pow(c,2);
}

for (a = 2; a < 998; a++) {
    for (b = a + 1; b < 999; b++) {
        for (c = b + 1; c < 1000; c++) {
            if(isPythagoreanTriplet(a,b,c) && a+b+c == 1000) {
                console.log("The answer is: " + String(a*b*c));
            }
        }
    }
}
```

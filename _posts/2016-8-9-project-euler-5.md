---
layout: post
title: Project Euler 5
category: project-euler
---

```javascript
// 2 * 3 * 5 * 7 * 11 * 13 * 17 * 19
initialGuess = 9699690;

for (i = initialGuess; i < 2432902008176640000; i+=10) {
    // Just check the highest power of each prime under 20.
    if (i % 7 == 0 && i % 9 == 0 && i % 11 == 0 && i % 13 == 0 && i % 16 == 0 && i % 17 == 0 && i % 19 ==0) {
        result = i;
        break;
    }
}

console.log("The answer is: " + String(result));
```

---
layout: post
title: Project Euler 1
category: project-euler
---

```javascript
var sum = 0;

for (x = 3; x<1000; x++) {
    if (x % 3 == 0 || x % 5 == 0) {
        sum += x;
    }
}

console.log(sum);
```

---
layout: post
title: Project Euler 2
category: project-euler
---

```javascript
var sum = 2;
var previous = 1;
var current = 2;

while (current < 4000000) {
    var next = previous + current;

    if (next % 2 == 0) {
        sum += next;
    }

    previous = current;
    current = next;
}

console.log(sum);
```

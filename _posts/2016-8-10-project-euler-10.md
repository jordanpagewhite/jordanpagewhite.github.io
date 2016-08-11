---
layout: post
title: Project Euler 10
category: project-euler
---

```javascript
function primeSieve(n) {
    // Eratosthenes algorithm to find all primes under n
    var array = [], upperLimit = Math.ceil(Math.sqrt(n)), output = [];

    // Make an array from 2 to (n - 1)
    for (var i = 0; i < n; i++) {
        array.push(true);
    }

    // Remove multiples of primes starting from 2, 3, 5,...
    for (var i = 2; i <= upperLimit; i++) {
        if (array[i]) {
            for (var j = i * i; j < n; j += i) {
                array[j] = false;
            }
        }
    }

    // All array[i] set to true are primes
    for (var i = 2; i < n; i++) {
        if(array[i]) {
          output.push(i);
        }
    }

    return output;
};

var result = primeSieve(2000000); 
var answer = 0;

for (i = 0; i < result.length; i++) {
    answer += result[i];
}

console.log("The answer is: " + String(answer));
```

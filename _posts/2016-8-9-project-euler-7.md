---
layout: post
title: Project Euler 7
category: project-euler
---

### Thoughts

I feel the same about this problem as I do with my current solution to #3. I need to revisit it and come up with a more interesting solution.

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

var result = primeSieve(1000000); 

if (result.length >= 10001) {
    console.log("The answer is: " + String(result[10000]));
} else {
    console.log("You didnt generate 10001 prime numbers. Bump up the input to primeSieve.");
}
```

---
layout: post
title: Project Euler 3
category: project-euler
---

### Thoughts

This solution seems kind of lackluster. I would like to come back and take another swing at this one now after reviewing the approach that I took. I will try to come back soon and dedicate more time and come up with a better algorithm.

```javascript
function primeSieve(n, currInt) {
    // Eratosthenes algorithm to find all primes under n
    var array = [], upperLimit = Math.floor(Math.sqrt(n)), output = [];

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
        if(array[i] && (currInt % i == 0)) {
            currInt /= i;
            if (currInt == 1) {
                return i;
            }
        }
    }

    return 0;
};

var initialNum = 600851475143;
var result = primeSieve(10000, initialNum); 

if (result != 0) {
    console.log("The answer is: " + String(result));
} else {
    console.log("The answer wasn't found in your primeArray. Bump up the input of primeSieve.");
}
```

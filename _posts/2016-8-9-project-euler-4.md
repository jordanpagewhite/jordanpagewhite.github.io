---
layout: post
title: Project Euler 4
category: project-euler
---

```javascript
// Returns a reversed String input
function reverseString(str) {
    return str.split("").reverse().join("");
}

// Checks if an int input is a palindrome
function isPalindrome(n) {
    if (reverseString(String(n)) == String(n)) {
        return 1;
    }

    return 0;
}

result = 0;

for (i = 100; i <= 999; i++) {
    for (j = 100; j <= 999; j++) {
        if (isPalindrome(i*j) && (i*j > result)) {
            result = i*j;
        }
    }
}

console.log("The answer is: " + String(result));
```

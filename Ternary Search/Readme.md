## Ternary Search

https://codeforces.com/blog/entry/43440

### Ternary Search in Integers

```cpp
int lo = -1, hi = n;
while (hi - lo > 1){
    int mid = (hi + lo)>>1;
    if (f(mid) > f(mid + 1)) 
         hi = mid;
    else 
         lo = mid; 
}
//lo + 1 is the answer
```

### Ternary Search in Double

- trivial
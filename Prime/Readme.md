## Prime 

### linear sieve

- 如何證明是線性？
- answer: ```每個合數只會被刪除一次, 因為每次刪除時我們都刪掉最小質因數, 每個數字都被最小質因數給刪除```

```cpp
vector<int> spf(n); // spf[i]: smallest prime, i%spf[i]==0
iota(spf.begin(), spf.end(), 0);

vector<int> prime;
for (int i = 2; i < n; ++i) {
    if (spf[i] == i) prime.push_back(i);
    for (int p: prime) {
        if (i*p >= n) break;
        spf[i*p] = p;
        if (spf[i] == p) 
            break;
    }
}
```

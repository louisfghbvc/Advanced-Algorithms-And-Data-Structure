## Weight Union Find

### template

- 可用來處理有權重的有向圖
- 能夠高效進行合併, 查詢
- find的時候更新value
- union的時候只需更新root的權重, 因為x,y會懶更新上去
- 公式推導如下

```cpp
int find(int x) {
    if (p[x] < 0) return x;
    int fa = p[x];
    p[x] = find(p[x]);
    value[x] += value[fa];
    return p[x];
}

// e.g x - y = w
//  b     c
// / \   /
// a x  y
// val[x] = x - b, val[y] = y - c  
// new val[b] => b - c
// b - c = val[y] - val[x] + w  
// = (y - c) - (x - b) + (x - y) 
// = (y - x) + b - c + (x - y)
bool uni(int x, int y, LL w) {
    int root_x = find(x), root_y = find(y);
    if (root_x == root_y) 
        return value[x] - value[y] == w;
    // x merge to y, we only move root, lazy update child in find!!!
    p[root_x] = root_y;
    value[root_x] = value[y] - value[x] + w;
    return true;
}
```

### 例題
https://atcoder.jp/contests/abc328/tasks/abc328_f

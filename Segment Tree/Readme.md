## Segment Tree

### template1 - Lazy Tag

- 能夠進行區間查詢, 區間修改
- 透過懶標記進行區間修改高效化(單點查詢也可以區間修改)
- 以下是區間加值, 區間查詢的範例程式

```cpp
struct SegmentTree {
        SegmentTree(int n) {
            tree.resize(4*n);
            lazyTag.resize(4*n);
        }

        void push(long id, int l, int r) {
            if (l == r) return;
            if (!lazyTag[id]) return;
            long inc = lazyTag[id];
            int m = (l+r)/2;
            lazyTag[id*2+1] += inc;
            tree[id*2+1] += inc * (m-l+1);

            lazyTag[id*2+2] += inc;
            tree[id*2+2] += inc * (r-(m+1)+1);
            lazyTag[id] = 0;
        }

        long query(long id, int l, int r, int ll, int rr) {
            push(id, l, r);
            if (ll <= l && r <= rr) return tree[id];
            if (l > rr || r < ll) return 0;
            int m = (l+r)/2;
            return query(id*2+1, l, m, ll, rr) + query(id*2+2, m+1, r, ll, rr);
        }

        void update(long id, int l, int r, int ll, int rr, int val) {
            push(id, l, r);
            if (l > rr || r < ll) return;
            if (ll <= l && r <= rr) {
                tree[id] += val * (r-l+1);
                lazyTag[id] += val; // inc
                return;
            }
            int m = (l+r)/2;
            update(id*2+1, l, m, ll, rr, val);
            update(id*2+2, m+1, r, ll, rr, val);
            tree[id] = tree[id*2+1] + tree[id*2+2];
        }

        vector<long> tree;
        vector<long> lazyTag;
    };
```

### template2 - Range Query + Point Update

- 簡單版, 沒有懶標記
- 結合可以用max, min, +, OR, 有結合性的操作

```c++
void build(vector<int> &arr, int id, int l, int r) {
    if (l == r) {
        tree[id] = node(arr[l]);
        return;
    }
    int m = (l+r)/2;
    build(arr, id*2+1, l, m);
    build(arr, id*2+2, m+1, r);
    tree[id] = tree[id*2+1] + tree[id*2+2];
};
void update(int p, int x, int id, int l, int r) {
    if (r < p || l > p) return;
    if (l == r) {
        tree[id] = x;
        return;
    }
    int m = (l+r)/2;
    update(p, x, id*2+1, l, m);
    update(p, x, id*2+2, m+1, r);
    tree[id] = max(tree[id*2+1], tree[id*2+2]);
}

int query(int L, int R, int id, int l, int r) {
    if (r < L || l > R) return 0;
    if (L <= l && r <= R) return tree[id];
    int m = (l+r)/2;
    return max(query(L, R, id*2+1, l, m), query(L, R, id*2+2, m+1, r));
}
```

## Union Find Rollback

### template

- 能夠還原Union find

```cpp
struct DSU {
    vi p, size, sum;
    vector<pair<LL&, LL>> history;
    DSU(int n) : size(n, 1), sum(n, 0), p(n, -1) {}
    int find(int x) { return p[x] < 0 ? x : find(p[x]); }
    void add(int x, int val) { 
        x = find(x);
		history.push_back({sum[x], sum[x]});
        sum[x] += val; 
    }
    bool unite(int x, int y) {
        x = find(x), y = find(y);
        if (x == y) return false;
        if (size[x] < size[y]) swap(x, y);
        // record
        history.push_back({p[y], p[y]});
        history.push_back({size[x], size[x]});
        history.push_back({sum[x], sum[x]});
        p[y] = x;
        sum[x] += sum[y];
        size[x] += size[y];
        return true;
    }
    int snapshot() { return history.size(); }
	void rollback(int until) {
		while (snapshot() > until) {
			history.back().first = history.back().second;
			history.pop_back();
		}
	}
};        
```

### 應用

#### 對於動態查詢
- u -> v 連接
- u -> v 斷開
- 查詢 anything
- 用interval tree將這些詢問離散化
- u->v 啥時連接啥時斷開都能離線處理
e.g.
```cpp
struct Query {
    int t, u, v, x;    
};

vector<Query> tree[4*N];
void update(Query& q, int id, int l, int r, int ll, int rr) {
    if (l > rr || r < ll) return;
    if (ll <= l && r <= rr) {
        tree[id].push_back(q);
        return;
    }
    int m = (l+r)/2;
    update(q, id*2+1, l, m, ll, rr);
    update(q, id*2+2, m+1, r, ll, rr);
}

void solve(int gg) {
    // cout << "Case #" << T+1 << ": ";
    
    // goal:
    // add/delete the edge 
    // query value
    
    // idea:
    // offline + rollback dsu
    // trick, convert query to interval
    
    int n, q;
    cin >> n >> q;
    vi a(n);
    cin >> a;
    
    DSU dsu(n);
    dsu.sum = a;
    
    map<pair<int, int>, int> rec;
    for (int i = 0; i < q; ++i) {
        int t;
        cin >> t;
        if (t == 0) {
            int u, v;
            cin >> u >> v;
            if (u > v) swap(u, v);
            rec[{u, v}] = i;
        }
        else if (t == 1) {
            int u, v;
            cin >> u >> v;
            if (u > v) swap(u, v);
            Query tmp = {1, u, v};
            // add [last,i-1]
            update(tmp, 0, 0, q-1, rec[{u,v}], i-1);
            rec[{u, v}] = -1;
        }
        else if (t == 2) {
            int v, x;
            cin >> v >> x;
            Query tmp = {2, -1, v, x};
            // add [i,q-1]
            update(tmp, 0, 0, q-1, i, q-1);
        }
        else {
            int v;
            cin >> v;
            Query tmp = {3, -1, v};
            // add [i,i]
            update(tmp, 0, 0, q-1, i, i);
        }
    }
    
    for (auto &[k, idx]: rec) {
        if (idx != -1) {
            Query tmp = {1, k.first, k.second};
            // add [i,q-1]
            update(tmp, 0, 0, q-1, idx, q-1);
        }
    }
    
    // traverse interval tree => O(qlogq)
    vi ans(q, -1);
    auto dfs = [&](auto& self, int id, int l, int r) -> void {
        int last = dsu.snapshot();
        for (auto &q: tree[id]) {
            if (q.t == 1) dsu.unite(q.u, q.v);
            if (q.t == 2) dsu.add(q.v, q.x);
        }
        if (l == r) { // get ans in leaf !
            for (auto &q: tree[id]) {
                if (q.t == 3) ans[l] = dsu.sum[dsu.find(q.v)];
            }
        }
        else { // keep traverse
            int m = (l+r)/2;
            self(self, id*2+1, l, m);
            self(self, id*2+2, m+1, r);
        }
        dsu.rollback(last);
    };
    dfs(dfs, 0, 0, q-1);
        
    // output
    for (int i = 0; i < q; ++i) {
        if (ans[i] != -1) {
            cout << ans[i] << '\n';
        }
    }
} 
```

### 例題
https://judge.yosupo.jp/problem/dynamic_graph_vertex_add_component_sum

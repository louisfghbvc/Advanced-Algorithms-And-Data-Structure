## BCC 雙連通分量

### template

- 找割點，找橋
- 割點
    - 如果 u 是根節點（即 parent[u] == -1），且有兩個或更多子節點，u是割點
    - 如果 u 不是根節點，且存在 low[v] >= disc[u] 的子節點 v，u是割點
- 橋的定義low[v] > dfn[u]
- 利用tarjan對無向圖進行dfs樹

```cpp
// usign tarjan find briage, and compute each tree size
LL findMaxNoConnectedPairs(vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> low(n), dfn(n), size(n), par(n);
    vector<pair<int, int>> briages;

    // tarjan
    int time = 0;
    function<void(int, int)> tarjan = [&](int u, int p) {
        dfn[u] = low[u] = ++time;
        size[u] = 1;
        for (auto &v: graph[u]) {
            if (v == p) continue;
            if (!dfn[v]) {
                par[v] = u;
                tarjan(v, u);
                low[u] = min(low[u], low[v]);
                size[u] += size[v];
                if (low[v] > dfn[u]) { // find briage
                    briages.push_back({u, v});
                }
            } else { // cross edge
                low[u] = min(low[u], dfn[v]);
            }
        }
    };
    for (int i = 0; i < n; ++i) if (!dfn[i]) tarjan(i, -1);

    // compute size
    LL ans = 0;
    for (auto &[u, v]: briages) {
        LL left = 0, right = 0;
        if (par[v] == u) {
            left = size[v];
            right = n - size[v];
        }
        else {
            left = size[u];
            right = n - size[u];
        }
        ans = max(ans, left*right);
    }
    return ans;
}

void solve(int T) {    
    // cout << "Case #" << T+1 << ": ";

    LL n, m;
    cin >> n >> m;

    vector<vector<int>> graph(n);
    for (int i = 0; i < m; ++i) {
        int u, v;
        cin >> u >> v;
        --u, --v;
        graph[u].push_back(v);
        graph[v].push_back(u);
    }

    // goal: find the maximum non-reach nodes by erase a edge
    // idea:
    // enumerate the critical edge, find out left size, right size, remove them

    cout << n*(n-1)/2 - findMaxNoConnectedPairs(graph) << '\n';
}            
```

### 例題

https://codeforces.com/contest/1986/problem/F

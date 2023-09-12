## SCC (Strong Connected Compoment)

### template

[tutorial](https://slides.com/sylveon/graph-7#/5)

- 可用來縮環, 找環的大小
- 只能用在有向圖
- 可將一張圖變成有向無環

```cpp
vector<int> graph[N];
int low[N], dfn[N], timer = 0;
bool inst[N]; // in stack
stack<int> st;

void dfs(int u) {
    low[u] = dfn[u] = ++timer;
    st.push(u);
    inst[u] = 1;

    for (int v: graph[u]) {
        if (!dfn[v]) {
            dfs(v);
            low[u] = min(low[u], low[v]);
        }
        else if (inst[v]) { // back edge
            low[u] = min(low[u], dfn[v]);
        }
    }

    if (low[u] == dfn[u]) {
        int x;
        do {
            x = st.top(); st.pop();
            inst[x] = false;
            // do something, e.g. scc[x] = id
        } while (x != u);
    }
}

void main() {
    for (int i = 0; i < n; ++i) {
        if (!dfn[i]) dfs(i);
    }
}
```

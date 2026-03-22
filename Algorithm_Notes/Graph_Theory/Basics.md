# C++ STL 速查表 与 图论存图

## 一、 C++ STL 常用容器速查表 (Standard Template Library)

| 分类  | 容器 (Container) | 头文件 | 增 (Add) | 删 (Delete) | 查/访 (Access) | 复杂度 (Complexity) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 📈 **动态数组** | `vector` | `<vector>` | `push_back(x)` | `pop_back()` | `v[i]` / `v.at(i)` | O(1) |
| ↔️ **双端队列** | `deque` | `<deque>` | `push_front/back` | `pop_front/back` | `d.front()` / `d.back()` | O(1) |
| 🧱 **栈** | `stack` | `<stack>` | `s.push(x)` | `s.pop()` | `s.top()` | O(1) (LIFO) |
| 🚶 **队列** | `queue` | `<queue>` | `q.push(x)` | `q.pop()` | `q.front()` | O(1) (FIFO) |
| 🏆 **优先队列** | `priority_queue` | `<queue>` | `q.push(x)` | `q.pop()` | `q.top()` | O(log N) (Heap) |
| 🌲 **有序集合** | `set` | `<set>` | `s.insert(x)` | `s.erase(it/x)` | `s.count(x)` | O(log N) (红黑树) |
| 🌲 **有序映射** | `map` | `<map>` | `m[key] = val` | `m.erase(it/key)` | `m.count(key)` | O(log N) (红黑树) |
| ⚡ **无序集合** | `unordered_set` | `<unordered_set>` | `s.insert(x)` | `s.erase(it/x)` | `s.count(x)` | O(1) (哈希表) |
| ⚡ **无序映射** | `unordered_map` | `<unordered_map>` | `m[key] = val` | `m.erase(it/key)` | `m.count(key)` | O(1) (哈希表) |

> **⚠️ 核心避坑指南：**
> 1. **优先队列 (`priority_queue`)**: 默认是**大根堆**（Top 是最大的）。若要小根堆，必须写全：`priority_queue<int, vector<int>, greater<int>> pq;`
> 2. **哈希容器 (`unordered_*`)**: 虽然平均 O(1)，但如果遇到极端哈希冲突（比如 Codeforces 上常有的针对性防 Hack 测试点）会退化到 O(N)。
> 3. **红黑树容器 (`set/map`)**: 内部自动排序，遍历时天然是有序的。

---

## 二、 五大存图方式

前置说明：`n` 为点的个数，`m` 为边的条数。

### 1. 邻接矩阵 (Adjacency Matrix)
* **原理：** 二维数组 `w[u][v]` 存储从点 `u` 到点 `v` 的边权。
* **复杂度：** 时间复杂度 O(n²)，空间复杂度 O(n²)。
* **应用场景：** 只在点数不多的**稠密图**上使用，如 n ≤ 1000，m ≈ 10⁶。

```cpp
int w[N][N]; // 存边权
int vis[N];  // 标记是否访问过

void dfs(int u) {
    vis[u] = true;
    for(int v = 1; v <= n; v++) {
        if (w[u][v]) {
            cout << u << v << w[u][v];
            if (vis[v]) continue;
            dfs(v);
        }
    }
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        cin >> a >> b >> c;
        w[a][b] = c;
        // w[b][a] = c; // 如果是无向图，加上这句
    }
    dfs(1);
}
```

### 2. 边集数组 (Edge List)
* **原理：** 一维数组 `e[i]` 存储第 `i` 条边的 `{起点 u, 终点 v, 边权 w}`。
* **复杂度：** 遍历时间复杂度 O(nm)，空间复杂度 O(m)。
* **应用场景：** 专用于**Kruskal 最小生成树算法**，因为需要直接将所有边按权值排序。

```cpp
struct edge {
    int u, v, w;
} e[M];
int vis[N];

void dfs(int u) {
    vis[u] = true;
    for(int i = 1; i <= m; i++) {
        if (e[i].u == u) {
            int v = e[i].v, w = e[i].w;
            cout << u << v << w;
            if (vis[v]) continue;
            dfs(e[i].v);
        }
    }
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        cin >> a >> b >> c;
        e[i] = { a, b, c };
        // e[i] = { b, a, c }; // 处理无向图需小心数组越界
    }
    dfs(1);
}
```

### 3. 邻接表 (Adjacency List using `vector`)
* **原理：** 出边数组 `e[u][i]` 存储 `u` 点的所有出边的 `{终点 v, 边权 w}`。
* **复杂度：** 时间复杂度 O(n + m)，空间复杂度 O(n + m)。
* **应用场景：** 适合各种图，但处理反向边（如网络流）时不太方便。

```cpp
struct edge { int v, w; };
vector<edge> e[N];

void dfs(int u, int fa) {
    for (auto ed : e[u]) {
        int v = ed.v, w = ed.w;
        if (v == fa) continue; // 防止树上 dfs 走回头路 
        cout << u << v << w;
        dfs(v, u);
    } 
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        cin >> a >> b >> c;
        e[a].push_back({ b, c }); 
        e[b].push_back({ a, c }); // 无向图双向建边
    }
    dfs(1, 0);
}
```

### 4. 链式邻接表
* **原理：** 结合了边集与表头。`e`存边，`h[u]` 存 `u` 点的所有出边在 `e` 中的序号。
* **复杂度：** 时间复杂度 O(n + m)，空间复杂度 O(n + m)。
* **应用场景：** 各种图，能处理反向边（常用于需要找反向边的场景）。

```cpp
struct edge { 
    int u, v, w; 
}; 
vector<edge> e;
vector<int> h[N];

void add(int a, int b, int c) {
    e.push_back({ a, b, c });
    h[a].push_back(e.size() - 1); 
}

void dfs(int u, int fa) {
    for (int i = 0; i < h[u].size(); i++) {
        int j = h[u][i];
        int v = e[j].v, w = e[j].w;
        if (v == fa) continue;
        cout << u << v << w;
        dfs(v, u);
    }
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        cin >> a >> b >> c;
        add(a, b, c);
        add(b, a, c);
    }
}
```

### 5. 链式邻接表
* **原理：** 数组模拟链表。一个表头数组悬挂多个链表。
* **数据结构：** 
`e[i]`: 第 `i` 条边的 `{终点 v, 边权 w, 下一条同起点边的编号 ne}`。
`h[u]`: 点 `u` 的最后一条（第一条被遍历的）出边编号。
* **复杂度：** 时间复杂度 O(n + m)，空间复杂度 O(n + m)。
* **应用场景：** 各种图，能处理反向边

```cpp
struct edge { 
    int v, w, ne; 
};
edge e[M];
int idx, h[N]; //idx 为边的全局唯一编号

void add(int a, int b, int c) {
    e[idx] = { b, c, h[a] }; //头插法：当前边的 ne 指向之前的头节点
    h[a] = idx++; //更新头节点为当前边
}

void dfs(int u, int fa) {
    for (int i = h[u]; ~i; i = e[i].ne) { //~i 等价于 i != -1
        int v = e[i].v, w = e[i].w;
        if (v == fa) continue;
        cout << u << v << w;
        dfs(v, u);
    } 
}

int main() {
    cin >> n >> m;
    memset(h, -1, sizeof h); //初始化表头，极其重要！
    for (int i = 1; i <= m; i++) {
        cin >> a >> b >> c;
        add(a, b, c);
        add(b, a, c);
    }
    dfs(1, 0);
}
```

> **💡 链式前向星：微信聊天记录类比法则**
> * **`idx` (全局消息编号)：** 腾讯主服务器有一个永远在增加的编号器。今天发的第一条微信编号是 0，第二条是 1，第三条是 2……
> * **`e` 数组 (云端硬盘)：** 所有的聊天消息，不管是谁发给谁的，全都一股脑按编号塞进这个大硬盘里。e[5] 就代表全局编号为 5 的那条消息。
> * **`h` 数组 (微信联系人列表)：** 比如 h["张三"]。这个里面只存一个数字，就是你跟张三聊天的“最新（最下面）的一条消息的编号”。
> * **`-1` 数组 (聊天记录尽头)：** 如果你点开一个新加的好友，里面空空如也。所以一开始，所有人的 h 里面存的都是 -1，代表“暂无消息”。
> * **读消息的循环：** `for(int i = h["张三"]; ~i; i = e[i].ne)`
>>* **`int i = h["张三"]` (打开对话框看最下面)：** 看一眼手机，发现和张三的最新消息编号是 100。好，我们先读第 100 条消息！
>>* **`i = e[i].ne` (手指往上滑一次)：** 读完第 100 条，你想看上一条。于是你揭开第 100 条消息上的隐形便利贴（ne），发现上面写着“上一条是编号 85”。好，我们跳去读第 85 条消息！继续往上滑，85 的便利贴写着 32，32 的便利贴写着 14……
>>* **`~i` 或 `i != -1` (滑到顶了！)：** 一直往上滑，终于你读到了某条极其古老的消息，揭开它的便利贴一看，上面赫然写着 -1。这就意味着：已经到顶了！上面没有消息了！循环结束！
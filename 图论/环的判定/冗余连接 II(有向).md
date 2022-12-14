# 冗余连接 II
[LeetCode 685. 冗余连接 II](https://leetcode.cn/problems/redundant-connection-ii/)

# 解题思路1
有向树有很多情况
  - ①只有环 
  有一个子节点连到了根节点 $\Rightarrow$ 基环树，去掉环上任意一条边都行
  - ② 无环，但有个点入度为 $2$ (出现了两个根节点)
  任意去除入度 $2$ 的任意一边
  - ③【以上①②都有】只能去掉①②公共边
  既在环里，又是某个点的 $2$ 条入边之一

- 整体框架        
  - `st1[i]` 表示：第i条边在环里
  - `st2[i]` 表示：两条入边之一
  从后往前找第一个`i`，使得`st1[i],st2[i] == true`

具体实现
- **targan算法: 找出每条边是否在图里**
  此题特殊：一棵树里带了一个环。 所以写简化版即可
  
  `dfs`根节点到该点肯定会路过它的祖先节点。`stk` 栈存一下`dfs`上的点, `in_stk[]` 布尔数组存某个点是否在栈里
   所以搜的时候，如果找到了栈当中的点，那么就找到了环

- **存入边：找出入边为2的点**

### Code
```cpp
class Solution {
public:
    int n; //点数= 边数
    vector<bool> st1, st2, st, in_k, in_c;
    //某个边是否在环里，某条边是否某个点的两个入边之一，找环的时候判重数组，某个点是否在栈里，某个点是否在环里。
    vector<vector<int>> g; //开一个邻接表来存所有的边
    stack<int> stk;

    //找环函数
    bool dfs(int u){
        st[u] = true; //标记这个点被遍历过了
        stk.push(u), in_k[u] = true; //u放进栈里，所以它在栈里in_k(u)=在的
        for (int x: g[u]){ //枚举出边
            if (!st[x]){
                if (dfs(x)) return true; //如果找到环 return true
            }
            else if (in_k[x]){ //如果x是在栈里
                //还需把环里所有点找出来
                while (stk.top()!= x){ //一直弹栈
                    in_c[stk.top()] = true; //都在环里，标记
                    stk.pop();
                }
                in_c[x] = true; //最后标记x也在环里
                return true; //找到环了
            }
        }
        stk.pop(), in_k[u] = false;//最后弹栈
        return false;
    }
    void work1(vector<vector<int>>& edges){
        //先初始化邻接表
        for (auto& e : edges)
        {
            int a = e[0], b = e[1];
            g[a].push_back(b); //g的邻居是b
        }
        // 枚举（遍历这个图）直到找到环
        for (int i = 1; i <= n; i++)
            if (!st[i] && dfs(i))
                break;
        for (int i = 0; i < n; i++){
            int a = edges[i][0], b = edges[i][1];
            if (in_c[a] && in_c[b]) //如果这(构成ab边的)两个点a和b都在环里
                st1[i] = true; //说明这个边在环里
        }

    }
    void work2(vector<vector<int>>& edges){
        vector<int> p(n + 1, -1); //初始化每个点的入边。 初始化为-1
        //从前往后枚举所有边
        for (int i = 0; i < n; i++){
            int a = edges[i][0], b = edges[i][1]; // 这个边的第一个点a，第二个点b
            if (p[b] != -1){ //如果这个点已经存过某条入边的话。
                st2[p[b]] = st2[i] = true; // 两个入边标记一下。
                //入边p[b]是某个点的两条入边
                // 这个研究一下s[p[b]]的含义 和这个循环的作用
                break;
            }
            else p[b] = i; //否则入边标记为i
        }
    }

    vector<int> findRedundantDirectedConnection(vector<vector<int>>& edges) {
        // 有向树有很多情况。3
        // ①【只有环】 有一个子节点连到了根节点 ---》奇环树，去掉环上任意一条边都行
        // ② 【无环，但有个点入度为2】 (出现了两个根节点)，任意去除入度2的任意一边
        // ③【以上①②都有】只能去掉①②公共边。既在环里，又是某个点的2条入边之一
        // st1[i] 表示：第i条边在环里
        // st2[i] 表示：两条入边之一
        // 解：从后往前找第一个i，使得st1[i],st2[i] = true.
        // targan算法 找出每条边是否在图里。
        // 此题特殊：一棵树里带了一个环。 所以写简化版即可。
        // dfs根节点到该点肯定会路过它的祖先节点。
        // stk 栈存一下dfs上的点
        // in_stk[] bool布尔数组存某个点是否在栈里
        // 所以【搜的时候，如果找到了栈当中的点，那么就找到了环】
        // O(N)
        // 【实际为找环模板题】
        n = edges.size();
        //初始化。长度n+1
        st1 = st2= st= in_k = in_c = vector<bool>(n + 1);
        g.resize(n + 1);
        work1(edges);
        work2(edges);

        //枚举一下，看是否是情况③，即情况①和情况②都满足
        for (int i = n - 1; i >= 0; i --)
            if (st1[i] && st2[i]) //如果edge(i)既是st1也是st2的边
                return edges[i]; //返回这个，即情况③

        //否则，说明是情况一或情况二。找最后一边即可。
        for (int i = n - 1; i >= 0; i --)
            if (st1[i] || st2[i]) //第一个集合或者第二个集合出现的话。当前边就是答案。
                return edges[i];
        return {}; //这个不执行，只是lc格式
    }
};
```

# 解题思路2

[并查集](https://leetcode.cn/problems/redundant-connection-ii/solution/685-rong-yu-lian-jie-iibing-cha-ji-de-ying-yong-xi/)

### Code
```cpp
class Solution {
private:
    static const int N = 1010; // 如题：二维数组大小的在3到1000范围内
    int father[N];
    int n; // 边的数量
    // 并查集初始化
    void init() {
        for (int i = 1; i <= n; ++i) {
            father[i] = i;
        }
    }
    // 并查集里寻根的过程
    int find(int u) {
        return u == father[u] ? u : father[u] = find(father[u]);
    }
    // 将v->u 这条边加入并查集
    void join(int u, int v) {
        u = find(u);
        v = find(v);
        if (u == v) return ;
        father[v] = u;
    }
    // 判断 u 和 v是否找到同一个根
    bool same(int u, int v) {
        u = find(u);
        v = find(v);
        return u == v;
    }
    // 在有向图里找到删除的那条边，使其变成树
    vector<int> getRemoveEdge(const vector<vector<int>>& edges) {
        init(); // 初始化并查集
        for (int i = 0; i < n; i++) { // 遍历所有的边
            if (same(edges[i][0], edges[i][1])) { // 构成有向环了，就是要删除的边
                return edges[i];
            }
            join(edges[i][0], edges[i][1]);
        }
        return {};
    }

    // 删一条边之后判断是不是树
    bool isTreeAfterRemoveEdge(const vector<vector<int>>& edges, int deleteEdge) {
        init(); // 初始化并查集
        for (int i = 0; i < n; i++) {
            if (i == deleteEdge) continue;
            if (same(edges[i][0], edges[i][1])) { // 构成有向环了，一定不是树
                return false;
            }
            join(edges[i][0], edges[i][1]);
        }
        return true;
    }
public:

    vector<int> findRedundantDirectedConnection(vector<vector<int>>& edges) {
        int inDegree[N] = {0}; // 记录节点入度
        n = edges.size(); // 边的数量
        for (int i = 0; i < n; i++) {
            inDegree[edges[i][1]]++; // 统计入度
        }
        vector<int> vec; // 记录入度为2的边（如果有的话就两条边）
        // 找入度为2的节点所对应的边，注意要倒叙，因为优先返回最后出现在二维数组中的答案
        for (int i = n - 1; i >= 0; i--) {
            if (inDegree[edges[i][1]] == 2) {
                vec.push_back(i);
            }
        }
        // 处理图中情况1 和 情况2
        // 如果有入度为2的节点，那么一定是两条边里删一个，看删哪个可以构成树
        if (vec.size() > 0) {
            if (isTreeAfterRemoveEdge(edges, vec[0])) {
                return edges[vec[0]];
            } else {
                return edges[vec[1]];
            }
        }
        // 处理图中情况3
        // 明确没有入度为2的情况，那么一定有有向环，找到构成环的边返回就可以了
        return getRemoveEdge(edges);

    }
};
```

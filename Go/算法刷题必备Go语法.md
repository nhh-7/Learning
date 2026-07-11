# Go 语言算法刷题必备语法

## 速查表 — 20 个最高频语法


| #  | 语法             | 示例                                                                           |
| -- | ---------------- | ------------------------------------------------------------------------------ |
| 1  | 短变量声明       | `a := 10` / `b, ok := m[1]`                                                    |
| 2  | make 切片        | `s := make([]int, n)`                                                          |
| 3  | make 二维切片    | `dp := make([][]int, n); for i := range dp { dp[i] = make([]int, m) }`         |
| 4  | append           | `s = append(s, x)` / `s = append(s, other...)`                                 |
| 5  | 切片截取         | `s[1:4]` — 左闭右开，共享底层数组                                             |
| 6  | map 操作         | `m := make(map[int]int)` / `v, ok := m[k]` / `delete(m, k)`                    |
| 7  | range 遍历       | `for i, v := range arr {}` / `for k, v := range m {}`                          |
| 8  | 排序             | `slices.Sort(s)` / `slices.SortFunc(s, func(a,b int) int { return a-b })`      |
| 9  | 二分查找         | `idx, found := slices.BinarySearch(s, x)`                                      |
| 10 | 栈操作           | `push: s = append(s, v)` / `pop: s = s[:len(s)-1]`                             |
| 11 | 队列操作         | `push: q = append(q, v)` / `pop: q = q[1:]`                                    |
| 12 | 堆操作           | `heap.Push(h, v)` / `v := heap.Pop(h).(int)`                                   |
| 13 | 内置 min/max     | `min(a, b)` / `max(a, b)` — Go 1.21+                                          |
| 14 | 字符串与数字互转 | `strconv.Atoi("123")` / `strconv.Itoa(123)`                                    |
| 15 | strings.Builder  | `var sb strings.Builder; sb.WriteString("hi"); sb.String()`                    |
| 16 | 反转链表         | `head.Next, prev, head = prev, head, head.Next`（三句循环）                    |
| 17 | 交换             | `a, b = b, a`                                                                  |
| 18 | 取模             | `ans = (ans + x) % MOD` / `((a-b)%MOD + MOD) % MOD`                            |
| 19 | 输入（ACM 模式） | `sc := bufio.NewScanner(os.Stdin); sc.Scan(); n, _ := strconv.Atoi(sc.Text())` |
| 20 | 递归闭包         | `var dfs func(i int) int; dfs = func(i int) int { ... }; dfs(0)`               |

---

## 一、基础语法

### 1.1 变量与零值

```go
// 短声明（最常用）
a := 10
b, ok := m[1]

// var 声明（需要指定类型或零值时）
var n int       // 0
var s []int     // nil
var m map[int]int // nil — 不能直接写入！必须 make

// 常量
const MOD = 1_000_000_007
const (A = iota; B; C)  // 0, 1, 2
```

### 1.2 控制流

```go
// for — Go 唯一的循环关键字
for i := 0; i < n; i++ { }   // 标准 for
for condition { }            // 相当于 while
for { }                      // 无限循环

// range 遍历（高频！）
for i, v := range arr { }    // i=索引, v=值（值拷贝！）
for k, v := range m { }      // 遍历 map（无序！）
for i, c := range "hello" { } // i=字节偏移, c=rune

// if 支持初始化语句
if v := f(); v > 0 { }

// switch 默认带 break，不会穿透
switch x {
case 1:
case 2, 3:
default:
}
```

### 1.3 类型转换

```go
int(3.14)        // float64 -> int
float64(42)      // int -> float64
int64(n)         // int -> int64
[]byte("abc")    // string -> []byte
string([]byte{97, 98, 99})  // []byte -> string

// 类型断言
var val any = 42
if v, ok := val.(int); ok { }

// nil 判断
if s == nil { }  // 切片
if m == nil { }  // map
if p == nil { }  // 指针/接口
```

---

## 二、切片与数组（核心 🔥）

### 2.1 创建

```go
// 数组（固定长度，值类型 — 很少用）
arr := [3]int{1, 2, 3}
arr := [...]int{1, 2, 3}

// 切片（动态，引用类型 — 天天用）
s := []int{1, 2, 3}
s := make([]int, n)       // 长度 n，满零值
s := make([]int, n, cap)  // 长度 n，容量 cap

// 二维切片
dp := make([][]int, n)
for i := range dp { dp[i] = make([]int, m) }
```

### 2.2 常用操作

```go
// 增删
s = append(s, x)          // 追加单个
s = append(s, x, y, z)    // 追加多个
s = append(s, other...)   // 合并切片
s = s[:len(s)-1]          // 删末尾（弹栈）

// 截取（左闭右开，共享底层数组！）
s := []int{0, 1, 2, 3, 4, 5}
s[1:4]    // [1 2 3]
s[:3]     // [0 1 2]
s[3:]     // [3 4 5]

// 复制
copy(dst, src)            // 返回 min(len(dst), len(src))
copy := append([]int(nil), s...)  // 深拷贝
```

> ⚠️ **切片陷阱**：截取共享底层数组，修改会影响原切片。需独立副本时用 `copy` 或深拷贝。

---

## 三、Map（🔥）

```go
m := make(map[int]int)
m := map[int]int{1: 10, 2: 20}

delete(m, 1)          // 删除键
v, ok := m[1]         // 判断键是否存在（ok 模式很重要！）

// 遍历（无序！）
for k, v := range m { }
for k := range m { }
for _, v := range m { }
```

### 用 map 实现 Set

```go
set := make(map[int]struct{})   // struct{} 不占内存
set[1] = struct{}{}
if _, ok := set[1]; ok { }
delete(set, 1)
```

---

## 四、字符串（🔥）

### 4.1 基本操作

```go
s := "hello"
len(s)    // 字节长度（非字符数！）
s[0]      // 取字节（不可修改！）

// Unicode 处理：用 range，不要用索引
for i, r := range s { }  // r 是 rune
rs := []rune(s)          // string → rune 切片
s = string(rs)           // rune 切片 → string
```

### 4.2 strings 包速查


| 函数                              | 作用                      |
| --------------------------------- | ------------------------- |
| `Contains(s, sub)`                | 是否包含                  |
| `HasPrefix(s, pre)` / `HasSuffix` | 前/后缀                   |
| `Index(s, sub)`                   | 首次出现位置，-1 表示没有 |
| `Split(s, sep)`                   | 切分，返回`[]string`      |
| `Join(parts, sep)`                | 拼接                      |
| `Trim(s, cutset)`                 | 去除两端字符              |
| `Replace(s, old, new, n)`         | 替换（n<0 表示全部）      |
| `Count(s, sub)`                   | 计数                      |
| `ToLower(s)` / `ToUpper(s)`       | 大小写                    |
| `Repeat(s, n)`                    | 重复 n 次                 |

### 4.3 字符串与数字互转

```go
import "strconv"

n, _ := strconv.Atoi("123")       // string → int
s := strconv.Itoa(123)             // int → string
n, _ := strconv.ParseInt("FF", 16, 64)  // 按进制解析
s := strconv.FormatInt(255, 16)    // 格式化为进制字符串
```

### 4.4 高效拼接

```go
var sb strings.Builder
sb.WriteByte('a')
sb.WriteString("hello")
sb.WriteRune('好')
result := sb.String()
```

---

## 五、排序与二分查找（🔥）

Go 1.21+ **统一用 `slices` 包**，比老 `sort` 包更简洁安全。

### 5.1 基础排序

```go
import "slices"

nums := []int{3, 1, 4, 1, 5}
slices.Sort(nums)  // [1 1 3 4 5]

strs := []string{"banana", "apple"}
slices.Sort(strs)  // ["apple" "banana"]
```

### 5.2 自定义排序：SortFunc

```go
// 比较函数返回：a-b<0 表示 a 排前面；a-b>0 表示 a 排后面；==0 表示相等
// 升序
slices.SortFunc(nums, func(a, b int) int { return a - b })

// 降序
slices.SortFunc(nums, func(a, b int) int { return b - a })

// 按绝对值升序
slices.SortFunc(nums, func(a, b int) int { return abs(a) - abs(b) })

// 结构体多字段排序
slices.SortFunc(people, func(a, b Person) int {
    if a.Age != b.Age { return a.Age - b.Age }
    return strings.Compare(a.Name, b.Name)
})

// 稳定排序
slices.SortStableFunc(nums, func(a, b int) int { return a - b })
```

### 5.3 二分查找

```go
// 必须已排序（升序）！
nums := []int{1, 1, 3, 4, 5}

idx, found := slices.BinarySearch(nums, 3)  // (2, true)
idx, found := slices.BinarySearch(nums, 2)  // (2, false) — 找不到时返回插入位置

// 自定义排序的二分
idx, found := slices.BinarySearchFunc(nums, target, func(n, target int) int {
    return n - target
})
```

### 5.4 检查是否已排序

```go
slices.IsSorted(nums)
slices.IsSortedFunc(nums, cmp)
```

### 5.5 旧版 sort 包（Go < 1.21 兼容）

```go
sort.Ints(arr)            // 升序
sort.Slice(arr, func(i, j int) bool { return arr[i] < arr[j] })
sort.SearchInts(arr, x)   // 二分
```

---

## 六、数学与位运算

### 6.1 内置函数（Go 1.21+）

```go
max(a, b)    // 两个值取最大
min(a, b)    // 两个值取最小
clear(m)     // 清空 map
clear(s)     // 切片元素置零
```

### 6.2 必背函数

```go
func abs(x int) int { if x < 0 { return -x }; return x }

func gcd(a, b int) int {
    for b != 0 { a, b = b, a%b }
    return a
}

func lcm(a, b int) int { return a / gcd(a, b) * b }

// 快速幂
func powMod(a, b, mod int) int {
    res := 1
    for b > 0 {
        if b&1 == 1 { res = res * a % mod }
        a = a * a % mod
        b >>= 1
    }
    return res
}
```

### 6.3 位运算速查

```go
x & (-x)       // 提取最低位的 1
x & (x-1)      // 消除最低位的 1
x >> k & 1     // 取第 k 位
x | (1 << k)   // 第 k 位置 1
x & ^(1 << k)  // 第 k 位置 0
x & 1          // 判断奇偶
x ^ x == 0     // 自我异或为 0

// 子集枚举
for mask := 0; mask < (1 << n); mask++ {
    for i := 0; i < n; i++ {
        if mask>>i&1 == 1 { /* 选中第 i 个 */ }
    }
}
```

---

## 七、算法模板（🔥）

### 7.1 二分搜索

```go
// 标准二分（在有序数组中找 target）
l, r := 0, n-1
for l <= r {
    mid := l + (r-l)/2   // 防溢出写法
    if nums[mid] == target { return mid }
    if nums[mid] < target { l = mid + 1 } else { r = mid - 1 }
}

// 左边界（第一个 >= target）
l, r := 0, n
for l < r {
    mid := l + (r-l)/2
    if nums[mid] >= target { r = mid } else { l = mid + 1 }
}

// 右边界（最后一个 <= target）
l, r := -1, n-1
for l < r {
    mid := (l+r+1)/2     // 上取整
    if nums[mid] <= target { l = mid } else { r = mid - 1 }
}
```

### 7.2 DFS / BFS

```go
var dirs = [][]int{{0,1}, {0,-1}, {1,0}, {-1,0}}

// DFS
var dfs func(r, c int)
dfs = func(r, c int) {
    if r < 0 || r >= m || c < 0 || c >= n || visited[r][c] { return }
    visited[r][c] = true
    for _, d := range dirs { dfs(r+d[0], c+d[1]) }
}

// BFS
q := [][]int{{sr, sc}}
visited[sr][sc] = true
for len(q) > 0 {
    cur := q[0]; q = q[1:]
    for _, d := range dirs {
        nr, nc := cur[0]+d[0], cur[1]+d[1]
        if nr >= 0 && nr < m && nc >= 0 && nc < n && !visited[nr][nc] {
            visited[nr][nc] = true
            q = append(q, []int{nr, nc})
        }
    }
}
```

### 7.3 回溯（排列组合）

```go
func permute(nums []int) [][]int {
    var res [][]int
    used := make([]bool, len(nums))
    var dfs func(path []int)
    dfs = func(path []int) {
        if len(path) == len(nums) {
            tmp := make([]int, len(path))
            copy(tmp, path)
            res = append(res, tmp)
            return
        }
        for i := 0; i < len(nums); i++ {
            if used[i] { continue }
            used[i] = true
            dfs(append(path, nums[i]))
            used[i] = false   // 回溯恢复
        }
    }
    dfs(nil)
    return res
}
```

### 7.4 DP 模板

```go
// 一维 DP
dp := make([]int, n+1)
dp[0] = 1
for i := 1; i <= n; i++ {
    dp[i] = dp[i-1] + ...   // 状态转移
}

// 二维 DP
dp := make([][]int, m+1)
for i := range dp { dp[i] = make([]int, n+1) }

// 滚动数组（省空间）
dp0, dp1 := 0, 0
for i := 0; i < n; i++ { dp0, dp1 = dp1, max(dp0+v, dp1) }

// 01 背包
dp := make([]int, target+1)
for _, w := range weights {
    for j := target; j >= w; j-- {   // 逆序！防止重复使用
        dp[j] = max(dp[j], dp[j-w]+v)
    }
}
```

### 7.5 记忆化搜索

```go
memo := make([]int, n)
for i := range memo { memo[i] = -1 }

var dfs func(i int) int
dfs = func(i int) int {
    if i <= 1 { return 1 }
    if memo[i] != -1 { return memo[i] }
    memo[i] = dfs(i-1) + dfs(i-2)
    return memo[i]
}
```

---

## 八、数据结构实现

### 8.1 栈

```go
stack := make([]int, 0)
// push
stack = append(stack, v)
// pop
v := stack[len(stack)-1]; stack = stack[:len(stack)-1]
// top
top := stack[len(stack)-1]
```

### 8.2 队列

```go
// 简单队列（出队 O(n)，数据量小时用）
q := make([]int, 0)
q = append(q, v)   // 入队
v := q[0]; q = q[1:]  // 出队

// 高效队列（双指针，O(1) 出入）
q := make([]int, n); front, rear := 0, 0
q[rear] = v; rear++       // 入队
v := q[front]; front++    // 出队
```

### 8.3 优先队列（heap）

```go
import "container/heap"

// 最小堆模板
type MinHeap []int
func (h MinHeap) Len() int           { return len(h) }
func (h MinHeap) Less(i, j int) bool { return h[i] < h[j] }  // 改 > 变最大堆
func (h MinHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *MinHeap) Push(x any)        { *h = append(*h, x.(int)) }
func (h *MinHeap) Pop() any {
    old := *h; n := len(old); x := old[n-1]; *h = old[:n-1]; return x
}

// 使用
h := &MinHeap{}
heap.Init(h)
heap.Push(h, 5)
v := heap.Pop(h).(int)   // 类型断言！
top := (*h)[0]           // 查看堆顶
```

### 8.4 链表

```go
type ListNode struct {
    Val  int
    Next *ListNode
}

// 遍历
for p := head; p != nil; p = p.Next { }

// 反转（经典）
func reverse(head *ListNode) *ListNode {
    var prev *ListNode
    for head != nil {
        next := head.Next
        head.Next = prev
        prev = head
        head = next
    }
    return prev
}
```

### 8.5 树

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

// 前序 DFS
func preorder(root *TreeNode) []int {
    var res []int
    var dfs func(*TreeNode)
    dfs = func(node *TreeNode) {
        if node == nil { return }
        res = append(res, node.Val) // 前序
        dfs(node.Left)
        dfs(node.Right)
    }
    dfs(root)
    return res
}

// BFS 层序
func levelOrder(root *TreeNode) [][]int {
    var res [][]int
    q := []*TreeNode{root}
    for len(q) > 0 {
        level := make([]int, 0, len(q))
        next := make([]*TreeNode, 0)
        for _, node := range q {
            level = append(level, node.Val)
            if node.Left  != nil { next = append(next, node.Left) }
            if node.Right != nil { next = append(next, node.Right) }
        }
        res = append(res, level)
        q = next
    }
    return res
}
```

---

## 九、输入输出（ACM 模式）

> LeetCode 核心代码模式不需要处理输入输出，直接实现函数即可。以下为 ACM 模式（牛客网等）。

### 9.1 输入

```go
import (
    "bufio"
    "os"
    "strconv"
)

// 推荐：bufio.Scanner
sc := bufio.NewScanner(os.Stdin)

// 按行读
for sc.Scan() { line := sc.Text() }

// 按单词读
sc.Split(bufio.ScanWords)
sc.Scan()
n, _ := strconv.Atoi(sc.Text())

// 一次性全读（数据量不大时最方便）
input, _ := os.ReadFile(os.Stdin)
nums := strings.Fields(string(input))  // 按空白切分
```

### 9.2 输出

```go
fmt.Println(ans)
fmt.Printf("%d %d\n", a, b)

// 大量输出时用 bufio（否则可能超时）
wr := bufio.NewWriter(os.Stdout)
defer wr.Flush()
fmt.Fprintln(wr, ans)
```

---

## 十、泛型（Go 1.18+）

刷题中泛型最有用的场景：写一次通用工具，避免为每种类型重复实现。

```go
// 泛型约束
type Number interface {
    ~int | ~int32 | ~int64 | ~float32 | ~float64
}

func min[T Number](a, b T) T { if a < b { return a }; return b }

// 泛型栈
type Stack[T any] struct { data []T }
func (s *Stack[T]) Push(v T)      { s.data = append(s.data, v) }
func (s *Stack[T]) Pop() T        { v := s.data[len(s.data)-1]; s.data = s.data[:len(s.data)-1]; return v }
func (s *Stack[T]) IsEmpty() bool { return len(s.data) == 0 }

// 泛型 Set
type Set[T comparable] map[T]struct{}
func NewSet[T comparable]() Set[T] { return make(Set[T]) }
func (s Set[T]) Add(v T)           { s[v] = struct{}{} }
func (s Set[T]) Has(v T) bool      { _, ok := s[v]; return ok }
func (s Set[T]) Remove(v T)        { delete(s, v) }
```

> `~int` 约束同时接受 `int` 和 `type MyInt int`；不用 `~` 则只接受 `int`。

---

## 十一、slices 包其他常用函数

```go
import "slices"

slices.Clone(s)                  // 深拷贝
slices.Reverse(s)                // 原地反转
slices.Equal(a, b)               // 比较相等
slices.Contains(s, x)            // 是否包含
slices.Index(s, x)               // 首次出现位置，-1 表示无
slices.Delete(s, i, j)           // 删除 s[i:j]，返回新切片
slices.Insert(s, i, v...)        // 在 i 插入
slices.Compact(s)                // 移除相邻重复
slices.Max(s) / slices.Min(s)    // 最大/最小值
slices.Repeat(s, n)              // 重复切片 n 次
slices.Concat(s1, s2, ...)       // 拼接多个切片
```

---

## 十二、maps 包常用函数（Go 1.21+）

```go
import "maps"

maps.Clone(m)           // 浅拷贝 map
maps.Copy(dst, src)     // 将 src 全部键值拷入 dst（覆盖同名键）
maps.Equal(m1, m2)      // 比较两个 map 是否相等
maps.Keys(m)            // 返回所有键的切片，常用在"先取 key 再排序"
maps.Values(m)          // 返回所有值的切片
maps.DeleteFunc(m, func(k, v V) bool { return ... })  // 按条件删除
```

### 典型用法：有序遍历 map

```go
// map 遍历无序，需要有序时先取 keys，排序后按 key 取值
keys := maps.Keys(m)
slices.Sort(keys)
for _, k := range keys {
    v := m[k]
}
```

### 常用场景

| 场景 | 用法 |
|------|------|
| 需要排序后遍历 map | `keys := maps.Keys(m); slices.Sort(keys)` |
| 深拷贝一份 map | `m2 := maps.Clone(m)` |
| 合并两个 map | `maps.Copy(dst, src)` |
| 判断两个 map 是否相同 | `maps.Equal(m1, m2)` |

> `maps.Keys` + `slices.Sort` 是刷题中 map 需要有序输出时的标准组合。

---

## 十三、实用技巧

```go
// 交换
a, b = b, a

// 取模（防负数）
ans = (ans + x) % MOD
ans = ((a - b) % MOD + MOD) % MOD

// 二维数组字面量初始化
grid := [][]int{{0, 1}, {1, 0}}

// 比较两个切片
slices.Equal(a, b)  // Go 1.21+

// 计时
defer func(t time.Time) { fmt.Println(time.Since(t)) }(time.Now())
```

### fmt 调试速查

```go
fmt.Printf("%v", s)   // 通用格式
fmt.Printf("%+v", s)  // 结构体带字段名
fmt.Printf("%#v", s)  // Go 语法格式
fmt.Printf("%T", s)   // 打印类型
fmt.Printf("%d", n)   // 十进制
fmt.Printf("%b", n)   // 二进制
fmt.Printf("%x", n)   // 十六进制
```

---

## 十四、make vs new

```go
// make：仅用于 slice/map/chan，返回初始化的值
s := make([]int, 5)
m := make(map[int]int)

// new：分配零值内存，返回指针
p := new(int)     // *int，值为 0

// ⚠️ map 必须 make，否则写会 panic
var m map[int]int
m[1] = 10  // panic: assignment to entry in nil map
```

---

## 十五、int 溢出

```go
// Go 溢出静默回绕，不 panic！
var a int32 = 2147483647
a++  // a == -2147483648

// 刷题防溢出：每一步都取模
const MOD = 1_000_000_007
ans = (ans + x) % MOD
ans = (ans * x) % MOD
```

---

## 十六、常见陷阱（必读 ⚠️）


| 严重度 | 陷阱                                | 说明                                                                 |
| ------ | ----------------------------------- | -------------------------------------------------------------------- |
| 🔴     | **nil map 写入会 panic**            | `var m map[int]int; m[1]=1` → panic。必须 `make`                    |
| 🔴     | **强转 interface nil 判断为 false** | `var p *int=nil; var i any=p; i==nil` → **false**                   |
| 🔴     | **map 并发写会 fatal error**        | 不能并发读写 map，需`sync.Mutex` 或 `sync.Map`                       |
| 🟡     | **range 是值拷贝**                  | `for _, v := range arr { v=1 }` 不修改原数组，用 `arr[i]`            |
| 🟡     | **切片截取共享底层数组**            | `b := a[1:3]; b[0]=1` 会影响 a。独立副本用 `copy`                    |
| 🟡     | **append 可能重新分配内存**         | `s = append(s, x)` 后旧的指针可能失效，**必须用返回值**              |
| 🟡     | **闭包捕获循环变量**                | `for i:=0; i<n; i++ { go func(){ fmt.Println(i) }() }` — 全部输出 n |
| 🟡     | **字符串不可修改**                  | `s[0]='a'` 编译错误。需修改时转 `[]byte` 或 `[]rune`                 |
| 🟡     | **int 位数取决于平台**              | 64 位机器是 64 位，不要假设是 32 位                                  |
| 🟢     | **for range map 无序**              | 每次遍历顺序可能不同                                                 |
| 🟢     | **defer 参数在定义时求值**          | 不是执行时求值                                                       |
| 🟢     | **nil 切片 vs 空切片**              | `var s []int` vs `s:=[]int{}`，len 都是 0，JSON 不同                 |
| 🟢     | **heap.Pop() 返回 any**             | 需要类型断言`v := heap.Pop(h).(int)`                                 |

---

## 附录 A：container/list（双向链表）

LRU Cache 等场景使用。

```go
import "container/list"

l := list.New()
l.PushBack(1); l.PushFront(2)
back := l.Back(); front := l.Front()
l.Remove(elem); l.MoveToFront(elem)

for e := l.Front(); e != nil; e = e.Next() {
    v := e.Value.(int)
}
```

---

## 附录 B：Channel（极少考，了解即可）

Channel 用于 goroutine 同步，只在"交替打印"等并发题中出现。

```go
ch := make(chan int)        // 无缓冲
ch := make(chan int, 10)    // 有缓冲

ch <- v    // 发送
v := <-ch  // 接收
close(ch)  // 关闭

// 遍历直到关闭
for v := range ch { }

// 非阻塞
select {
case v := <-ch:
default:
}

// 只读/只写
func reader(ch <-chan int) {}
func writer(ch chan<- int) {}
```

---

## 附录 C：LeetCode  vs ACM 模式对比


|          | LeetCode      | ACM（牛客等）                      |
| -------- | ------------- | ---------------------------------- |
| 输入     | 函数参数传入  | 自己从 stdin 读                    |
| 输出     | return 返回值 | 自己写到 stdout                    |
| 包名     | 不需要        | 需要`package main` + `func main()` |
| 输入速度 | 不关心        | 数据量大时`fmt.Scan` 可能超时      |
| 推荐方式 | 直接写函数体  | `bufio.Scanner` + `bufio.Writer`   |

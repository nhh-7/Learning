# Go 语言算法刷题必备语法

## 一、变量与类型

### 基本类型
```go
bool
int, int8, int16, int32, int64
uint, uint8, uint16, uint32, uint64
float32, float64
byte      // uint8 的别名
rune      // int32 的别名，表示 Unicode 码点
string
```

### 零值初始化
```go
var a int       // 0
var b bool      // false
var c string    // "" （空字符串）
var d []int     // nil
var e *int      // nil
var m map[int]int // nil（不能直接写入！）
```

### 短变量声明
```go
a := 10
b, ok := m[1]
```

### 常量
```go
const MOD = 1_000_000_007
const (
    A = iota   // 0
    B          // 1
    C          // 2
)
```

---

## 二、控制流

### 只有 for 循环（没有 while/do-while）
```go
// 标准 for
for i := 0; i < n; i++ { }

// 相当于 while
for condition { }

// 无限循环
for { break }

// range 遍历
for i, v := range arr { }
for k, v := range m { }
for i, c := range "hello" { }  // i 是字节下标，c 是 rune
```

### if 支持初始化语句
```go
if v := f(); v > 0 { }
```

### switch 默认带 break
```go
switch x {
case 1:
    // 不会 fallthrough
case 2, 3:
    // 匹配多个值
default:
}
```

---

## 三、输入输出（重点）

### 方式一：fmt.Scan (慢，数据量大时超时)
```go
var n int
fmt.Scan(&n)
```

### 方式二：bufio.Scanner（推荐，高效）
```go
sc := bufio.NewScanner(os.Stdin)
// 按单词读
sc.Scan()
n, _ := strconv.Atoi(sc.Text())

// 按行读
for sc.Scan() {
    line := sc.Text()
}

// 读多行不定长数据
sc := bufio.NewScanner(os.Stdin)
sc.Split(bufio.ScanWords)
for i := 0; i < n; i++ {
    sc.Scan()
    v, _ := strconv.Atoi(sc.Text())
}
```

### 方式三：bufio + strings
```go
input, _ := os.ReadFile(os.Stdin)
nums := strings.Fields(string(input)) // 按空白字符切分
```

### 输出
```go
fmt.Println(ans)
fmt.Printf("%d %d\n", a, b)

// 大量输出用 bufio
wr := bufio.NewWriter(os.Stdout)
defer wr.Flush()
fmt.Fprintln(wr, ans)
```

> **注意**：算法竞赛中推荐 `bufio.Scanner` 读 + `bufio.Writer` 写

---

## 四、数组与切片（核心数据结构）

### 数组（固定长度，值类型）
```go
var arr [3]int          // [0 0 0]
arr := [3]int{1, 2, 3}
arr := [...]int{1, 2, 3} // 自动推断长度
```

### 切片（动态数组，引用类型）
```go
var s []int             // nil 切片
s := []int{1, 2, 3}
s := make([]int, n)     // 长度 n，元素零值
s := make([]int, n, m)  // 长度 n，容量 m
```

### 常用操作
```go
s = append(s, x)           // 末尾添加
s = append(s, x, y, z)     // 添加多个
s = append(s, other...)    // 合并切片
s = s[:len(s)-1]           // 弹栈（删除末尾一个元素）
copy(dst, src)             // 复制（返回复制的元素个数，取 min(len(src), len(dst))）

// 二维切片（动态 DP 数组等）
dp := make([][]int, n)
for i := range dp {
    dp[i] = make([]int, m)
}
```

### 切片截取
```go
s := []int{0, 1, 2, 3, 4, 5}
s[1:4]    // [1 2 3]   左闭右开
s[:3]     // [0 1 2]
s[3:]     // [3 4 5]
```

> **切片陷阱**：截取共享底层数组！修改会影响原切片。如需独立副本，用 `copy` 或 `append([]int(nil), s...)`

---

## 五、映射（Map）

```go
m := make(map[int]int)
m := map[int]int{1: 10, 2: 20}
delete(m, 1)               // 删除键
v, ok := m[1]              // ok 判断键是否存在

// 遍历（无序！）
for k, v := range m { }
for k := range m { }
for _, v := range m { }
```

### 用 map 实现 Set
```go
set := make(map[int]struct{})
set[1] = struct{}{}
if _, ok := set[1]; ok { }
delete(set, 1)
```

---

## 六、通道（Channel）

Channel 是 Go 的内置并发原语，少数算法题（如并发编排、交替打印）会用到。

```go
ch := make(chan int)          // 无缓冲通道（同步）
ch := make(chan int, 10)      // 有缓冲通道（容量 10）

ch <- v                       // 发送
v := <-ch                     // 接收
close(ch)                     // 关闭

// 遍历通道（直到关闭）
for v := range ch { }

// 非阻塞收发（select + default）
select {
case v := <-ch:
    // 收到数据
default:
    // 不会阻塞
}

// 只读/只写通道作为参数
func reader(ch <-chan int) {}   // 只读
func writer(ch chan<- int) {}   // 只写

// 用 channel 实现信号量（控制并发数）
sem := make(chan struct{}, 5)
```

> 刷题中 channel 主要用于 **goroutine 同步/交替打印** 类问题，常规算法题很少涉及。

---

## 七、结构体（Struct）

```go
type Point struct {
    X, Y int
}

p := Point{1, 2}
p := Point{X: 1}          // Y 默认为 0
pp := &Point{1, 2}        // 指针

// 结构体比较：所有字段可比较时，结构体也可比较
p1 == p2                  // 可做 map 键
```

---

## 八、字符串操作

### 字符串是不可变的字节序列
```go
s := "hello"
len(s)            // 字节长度（不是字符数）
s[0]              // 取字节（不可修改）
```

### strings 包
```go
strings.HasPrefix(s, "he")   // true
strings.HasSuffix(s, "lo")   // true
strings.Contains(s, "ell")   // true
strings.Index(s, "ll")       // 2
strings.Split("a,b,c", ",")   // ["a","b","c"]
strings.Join([]string{"a","b"}, ",")  // "a,b"
strings.Trim(s, " ")          // 去除两端空白
strings.ToLower(s)
strings.ToUpper(s)
strings.Repeat("a", 5)        // "aaaaa"
strings.Replace(s, "l", "x", -1)  // 全部替换
strings.Count(s, "l")         // 2
```

### 字符串与数字互转
```go
n, _ := strconv.Atoi("123")
s := strconv.Itoa(123)
n, _ := strconv.ParseInt("FF", 16, 64)
s := strconv.FormatInt(255, 16)   // "ff"
b, _ := strconv.ParseBool("true")
```

### 字符处理
```go
byte(s[i])    // 取第 i 个字节
rune(s[i])    // 错误！应用 range 遍历
for i, r := range s { }  // i 是字节偏移，r 是 rune

// 字符串转 rune 切片（处理 Unicode）
rs := []rune(s)
s = string(rs)

unicode.IsDigit(r)
unicode.IsLetter(r)
unicode.IsLower(r), unicode.IsUpper(r)
```

### 快速构建字符串
```go
var sb strings.Builder
sb.WriteByte('a')
sb.WriteString("hello")
sb.WriteRune('好')
s := sb.String()
```

---

## 九、排序

```go
import "sort"

sort.Ints(arr)        // 原地排序 []int
sort.Strings(s)       // 原地排序 []string
sort.Float64s(f)      // 原地排序 []float64

// 自定义排序
sort.Slice(arr, func(i, j int) bool {
    return arr[i] < arr[j]  // 升序
})

// 降序
sort.Slice(arr, func(i, j int) bool {
    return arr[i] > arr[j]
})

// 自定义类型的排序（实现 sort.Interface）
type Pair struct{ a, b int }
type Pairs []Pair
func (p Pairs) Len() int           { return len(p) }
func (p Pairs) Less(i, j int) bool { return p[i].a < p[j].a }
func (p Pairs) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }
sort.Sort(Pairs(arr))

// 二分查找（需要已排序）
sort.SearchInts(arr, x)      // 返回 x 在 arr 中的插入位置
sort.Search(n, func(i int) bool {
    return arr[i] >= x
})
// sort.Search 返回使 f(i) 为 true 的最小 i
```

---

## 十、数学运算

### Go 1.21+ 内置
```go
max(a, b)
min(a, b)
clear(m)    // 清空 map
clear(s)    // 将切片元素置零
```

### math 包
```go
math.MaxInt32, math.MinInt32
math.MaxInt64, math.MinInt64
math.Abs(x)            // float64
math.Pow(x, y)         // float64
math.Sqrt(x)           // float64
math.Max(x, y)         // float64
math.Min(x, y)         // float64
```

### 手动实现（经常需要用）
```go
func abs(x int) int {
    if x < 0 { return -x }
    return x
}

func min(a, b int) int {
    if a < b { return a }
    return b
}

func max(a, b int) int {
    if a > b { return a }
    return b
}

// 最大公约数
func gcd(a, b int) int {
    for b != 0 {
        a, b = b, a%b
    }
    return a
}

// 最小公倍数
func lcm(a, b int) int {
    return a / gcd(a, b) * b
}

// 快速幂（模版）
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

---

## 十一、常用数据结构实现

### 栈（Stack）
```go
// 用切片实现
stack := make([]int, 0)
push: stack = append(stack, v)
pop:  v = stack[len(stack)-1]; stack = stack[:len(stack)-1]
top:  v = stack[len(stack)-1]
```

### 队列（Queue）
```go
// 用切片实现（性能较差，频繁出队时用链表或环形队列）
q := make([]int, 0)
enqueue: q = append(q, v)
dequeue: v = q[0]; q = q[1:]   // O(n)! 大数据量不推荐

// 推荐：用两个索引模拟队列
q := make([]int, n)
front, rear := 0, 0
enqueue: q[rear] = v; rear++
dequeue: v = q[front]; front++
// 注意：可能存在内存泄漏，但刷题够用
```

### 优先队列（container/heap）
```go
import "container/heap"

// 最小堆模板（必须实现以下接口）
type MinHeap []int
func (h MinHeap) Len() int           { return len(h) }
func (h MinHeap) Less(i, j int) bool { return h[i] < h[j] }  // 最小堆
func (h MinHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *MinHeap) Push(x any)        { *h = append(*h, x.(int)) }
func (h *MinHeap) Pop() any {
    old := *h; n := len(old); x := old[n-1]; *h = old[:n-1]; return x
}

// 使用
h := &MinHeap{}
heap.Init(h)
heap.Push(h, 5)
v := heap.Pop(h).(int)   // 返回最小值
top := (*h)[0]           // 查看堆顶（不弹出）

// 最大堆：将 Less 改为 > 即可
func (h MaxHeap) Less(i, j int) bool { return h[i] > h[j] }
```

### 环形队列
```go
type MyCircularQueue struct {
    data []int
    front, rear, size, cap int
}
func New(k int) MyCircularQueue {
    return MyCircularQueue{data: make([]int, k+1), rear: 0, cap: k+1}
}
func (q *MyCircularQueue) EnQueue(v int) bool {
    if q.IsFull() { return false }
    q.data[q.rear] = v
    q.rear = (q.rear + 1) % q.cap
    q.size++
    return true
}
func (q *MyCircularQueue) DeQueue() bool {
    if q.IsEmpty() { return false }
    q.front = (q.front + 1) % q.cap
    q.size--
    return true
}
func (q *MyCircularQueue) IsEmpty() bool  { return q.size == 0 }
func (q *MyCircularQueue) IsFull() bool   { return q.size == q.cap-1 }
func (q *MyCircularQueue) Front() int     { return q.data[q.front] }
func (q *MyCircularQueue) Rear() int      { return q.data[(q.rear-1+q.cap)%q.cap] }
```

---

## 十二、链表操作

```go
type ListNode struct {
    Val  int
    Next *ListNode
}

// 遍历
for p := head; p != nil; p = p.Next { }

// 删除节点
prev.Next = prev.Next.Next

// 反转链表
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

---

## 十三、树操作

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

// 前序遍历
func preorder(root *TreeNode) []int {
    var res []int
    var dfs func(*TreeNode)
    dfs = func(node *TreeNode) {
        if node == nil { return }
        res = append(res, node.Val)  // 前序
        dfs(node.Left)
        dfs(node.Right)
    }
    dfs(root)
    return res
}

// BFS 层序遍历
func levelOrder(root *TreeNode) [][]int {
    var res [][]int
    q := []*TreeNode{root}
    for len(q) > 0 {
        level := make([]int, len(q))
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

## 十四、闭包与函数式

```go
// 闭包（DP 记忆化、DFS 常用）
var dfs func(i int) int
dfs = func(i int) int {
    if i <= 1 { return 1 }
    return dfs(i-1) + dfs(i-2)
}

// 用 defer 处理回溯状态恢复
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
            used[i] = false
        }
    }
    dfs(nil)
    return res
}
```

### 记忆化搜索
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

## 十五、常见算法模板

### 二分搜索
```go
// 标准二分
l, r := 0, n-1
for l <= r {
    mid := l + (r-l)/2  // 防止溢出
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
    mid := (l+r+1)/2  // 上取整
    if nums[mid] <= target { l = mid } else { r = mid-1 }
}
```

### DFS / BFS
```go
// DFS 框架
var visited [][]bool
var dirs = [][]int{{0,1},{0,-1},{1,0},{-1,0}}
var dfs func(r, c int)
dfs = func(r, c int) {
    if r < 0 || r >= m || c < 0 || c >= n || visited[r][c] { return }
    visited[r][c] = true
    for _, d := range dirs { dfs(r+d[0], c+d[1]) }
}

// BFS 框架
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

### DP 常见形式
```go
// 一维 DP
dp := make([]int, n)
for i := range dp {
    // 状态转移
}

// 二维 DP
dp := make([][]int, m)
for i := range dp { dp[i] = make([]int, n) }

// 滚动数组优化
dp0, dp1 := 0, 0
for i := 0; i < n; i++ {
    dp0, dp1 = dp1, max(dp1, dp0+v)
}

// 背包 DP
dp := make([]int, target+1)
for _, w := range weights {
    for j := target; j >= w; j-- {  // 01背包：逆序
        dp[j] = max(dp[j], dp[j-w]+v)
    }
}
```

---

## 十六、位运算

```go
x & (-x)       // 提取最低位的 1
x & (x-1)      // 消除最低位的 1
x >> k & 1     // 取第 k 位（从0开始）
x | (1 << k)   // 将第 k 位置 1
x & ^(1 << k)  // 将第 k 位置 0
x ^ y          // 异或（相同为0，不同为1）
x ^ x == 0     // 自我异或为0
x ^ 0 == x     // 与0异或为自身
x & 1          // 判断奇偶（1为奇，0为偶）

// 子集枚举（位掩码）
for mask := 0; mask < (1 << n); mask++ {
    for i := 0; i < n; i++ {
        if mask>>i&1 == 1 {
            // 选中第 i 个元素
        }
    }
}
```

---

## 十七、类型转换与空值判断

```go
// 类型转换
int(3.14)      // float64 -> int
float64(42)    // int -> float64
int64(n)       // int -> int64
string('A')    // rune -> string
[]byte("abc")  // string -> []byte
string([]byte{97,98,99}) // []byte -> string

// 类型断言
var val any = 42
if v, ok := val.(int); ok { }

// nil 判断
if s == nil { }           // 切片
if m == nil { }           // map
if p == nil { }           // 指针/接口
```

---

## 十八、常用技巧

### 交换
```go
a, b = b, a
```

### 取模
```go
ans := (ans + x) % MOD
// 处理负数
ans = ((ans-x)%MOD + MOD) % MOD
```

### go:build 条件编译
```go
// LeetCode 在线评测不需要写 package，本地可以
package main
```

### 计时
```go
defer func(t time.Time) {
    fmt.Println(time.Since(t))
}(time.Now())
```

### 比较两个切片是否相等
```go
slices.Equal(a, b)  // Go 1.21+

// 或手动
func equal(a, b []int) bool {
    if len(a) != len(b) { return false }
    for i := range a {
        if a[i] != b[i] { return false }
    }
    return true
}
```

### 二维数组初始化
```go
grid := make([][]int, m)
for i := range grid {
    grid[i] = make([]int, n)
}
// 或直接字面量
grid := [][]int{{0,1},{1,0}}
```

---

---

## 二十、泛型（Go 1.18+）

对于刷题，泛型最实用的场景是写一次通用的工具函数，避免为每种类型重复实现。

```go
// 泛型约束
type Number interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64
}

func min[T Number](a, b T) T {
    if a < b { return a }; return b
}
func max[T Number](a, b T) T {
    if a > b { return a }; return b
}
func abs[T Number](a T) T {
    if a < 0 { return -a }; return a
}

// 泛型栈
type Stack[T any] struct { data []T }
func (s *Stack[T]) Push(v T)      { s.data = append(s.data, v) }
func (s *Stack[T]) Pop() T        { v := s.data[len(s.data)-1]; s.data = s.data[:len(s.data)-1]; return v }
func (s *Stack[T]) Top() T        { return s.data[len(s.data)-1] }
func (s *Stack[T]) Len() int      { return len(s.data) }
func (s *Stack[T]) IsEmpty() bool { return len(s.data) == 0 }

// 泛型队列
type Queue[T any] struct { data []T; l, r int }
func (q *Queue[T]) Push(v T)      { q.data = append(q.data, v); q.r++ }
func (q *Queue[T]) Pop() T        { v := q.data[q.l]; q.l++; return v }
func (q *Queue[T]) Front() T      { return q.data[q.l] }
func (q *Queue[T]) Len() int      { return q.r - q.l }
func (q *Queue[T]) IsEmpty() bool { return q.l == q.r }

// 泛型 Set
type Set[T comparable] map[T]struct{}
func NewSet[T comparable]() Set[T]      { return make(Set[T]) }
func (s Set[T]) Add(v T)                { s[v] = struct{}{} }
func (s Set[T]) Remove(v T)            { delete(s, v) }
func (s Set[T]) Has(v T) bool          { _, ok := s[v]; return ok }
```

### 使用 `~` 前缀
```go
type MyInt int
// 约束写 int 不接受 MyInt，写 ~int 则接受
```

---

## 二十一、slices 包（Go 1.21+）

```go
import "slices"

slices.Equal(a, b)           // 比较两个切片是否相等
slices.Clone(s)              // 深拷贝切片（等价于 append([]T(nil), s...)）
slices.Reverse(s)            // 原地反转
slices.Sort(s)               // 升序排序
slices.SortFunc(s, cmp)      // 自定义排序
slices.BinarySearch(s, x)    // 二分查找，返回 (index, found)
slices.Contains(s, x)        // 是否包含
slices.Index(s, x)           // 第一个出现位置，没有返回 -1
slices.Delete(s, i, j)       // 删除 s[i:j]，返回新切片
slices.Insert(s, i, v...)    // 在 i 处插入
slices.Replace(s, i, j, v...) // 替换 s[i:j]
slices.Compact(s)            // 移除相邻重复元素
slices.Max(s), slices.Min(s) // 返回最大值/最小值
```

---

## 二十二、container/list（双向链表）

```go
import "container/list"

l := list.New()
l.PushBack(1)          // 尾部添加
l.PushFront(2)         // 头部添加
back := l.Back()       // 获取尾部元素
front := l.Front()     // 获取头部元素
l.Remove(back)         // 删除指定元素
l.MoveToFront(elem)    // 移到头部
l.MoveToBack(elem)     // 移到尾部

// 遍历
for e := l.Front(); e != nil; e = e.Next() {
    v := e.Value.(int)  // 类型断言
}
```

常用于 **LRU Cache** 类问题。

---

## 二十三、make vs new

```go
// make：只用于 slice/map/chan，返回初始化后的值（非指针）
s := make([]int, 5)  // cap 自动 = len
m := make(map[int]int)

// new：分配零值内存，返回指针
p := new(int)        // *int，值为 0
st := new(Struct)    // *Struct，所有字段为零值

// ⚠️ 以下会 panic：map 需要 make 初始化
var m map[int]int
m[1] = 10  // panic: assignment to entry in nil map
```

---

## 二十四、int 溢出

```go
// Go 的 int 溢出是静默回绕，不会 panic！
var a int32 = 2147483647
a++  // a == -2147483648，无任何警告

// 字符串长度可能很大，len() 返回 int（64位机器上是 int64）
// 刷题时注意取模防止溢出
const MOD = 1_000_000_007
ans := (ans + x) % MOD     // 加法取模
ans := (ans * x) % MOD     // 乘法取模
ans := ((a-b)%MOD + MOD) % MOD  // 减法取模防负数
```

---

## 二十五、fmt 格式化动词（调试用）

```go
fmt.Printf("%v", s)    // 通用格式
fmt.Printf("%+v", s)   // 结构体带字段名输出
fmt.Printf("%#v", s)   // Go 语法格式输出
fmt.Printf("%T", s)    // 类型
fmt.Printf("%d", n)    // 十进制整数
fmt.Printf("%b", n)    // 二进制
fmt.Printf("%x", n)    // 十六进制
fmt.Printf("%s", str)  // 字符串
fmt.Printf("%q", str)  // 带引号的字符串，安全转义
fmt.Printf("%p", ptr)  // 指针地址
fmt.Printf("%t", b)    // bool
```

---

## 二十六、常见陷阱汇总

| 陷阱 | 说明 |
|------|------|
| **range 是值拷贝** | `for _, v := range arr { v = 1 }` 不会修改原数组，需用索引 |
| **闭包捕获循环变量** | `for i := 0; i < n; i++ { go func() { fmt.Println(i) }() }` 全部输出 n，需要 `i` 作为参数传入 |
| **append 可能重新分配** | `s = append(s, x)` 之后，之前的切片引用可能失效，**一定要用返回值** |
| **map 并发不安全** | 并发读写 map 会 fatal error，需加锁或用 sync.Map |
| **nil 切片 vs 空切片** | `var s []int` 是 nil，`s := []int{}` 是空切片，`len` 都是 0，但 JSON 序列化结果不同 |
| **defer 在函数返回时执行** | 参数在 defer 时求值（值拷贝），而非执行时 |
| **for range map 无序** | 每次遍历顺序可能不同 |
| **interface 的 nil 判断** | `var p *int = nil; var i any = p; i == nil` 是 **false**，因为 interface 有类型信息 |
| **大切片作为参数** | 切片 header 很小（24 字节），传参是值拷贝但底层数组共享 |
| **int 默认 32 位还是 64 位** | 取决于平台，64 位机器上是 64 位 |

---

## 二十七、刷题注意事项总结

| 要点 | 说明 |
|------|------|
| 输入用 `bufio.Scanner` | `fmt.Scan` 太慢，大数据量必超时 |
| 输出用 `bufio.Writer` | 大量输出记得 flush |
| 切片是引用类型 | `append` 可能改变底层数组地址，注意传参 |
| map 遍历无序 | 需要有序时先取 keys 再排序 |
| `for range` 是值拷贝 | 修改 v 不会影响原切片 |
| 字符串不可修改 | 需要修改时转 `[]byte` 或 `[]rune` |
| 切片截取共享内存 | 想独立用 copy |
| 闭包捕获变量 | 循环中小心，需要时复制一份 |
| `heap.Pop()` 返回 `any` | 需要类型断言 `.(int)` |
| LeetCode 函数签名 | 不需要处理输入输出，只需实现给定函数 |

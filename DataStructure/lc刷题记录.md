题解：
https://github.com/EndlessCheng/codeforces-go/blob/master/leetcode/SOLUTIONS.md
https://leetcode.cn/problems/shortest-subarray-with-sum-at-least-k/solution/liang-zhang-tu-miao-dong-dan-diao-dui-li-9fvh/

### 模拟
43. 字符串相乘
sol1:做加法(普通竖式)
sol2:做乘法(优化竖式)
https://leetcode.cn/problems/multiply-strings/solutions/188815/gao-pin-mian-shi-xi-lie-zi-fu-chuan-cheng-fa-by-la/
```
想要做出这道题，需要知道一个数学定理：
两个长度分别为n和m的数相乘，长度不会超过n+m
```
### 哈希
128. 最长连续序列

### 堆
1962. 移除石子使总数最小
堆模版
模版1
```go
func minStoneSum(piles []int, k int) (ans int) {
    h := &hp{piles}
    heap.Init(h) // 原地堆化
    for ; k > 0 && piles[0] > 0; k-- {
        piles[0] -= piles[0] / 2 // 直接修改堆顶
        heap.Fix(h, 0)
    }
    for _, x := range piles {
        ans += x
    }
    return
}

type hp struct{ sort.IntSlice }
func (h hp) Less(i, j int) bool { return h.IntSlice[i] > h.IntSlice[j] }
func (h *hp) Push(v any)        { h.IntSlice = append(h.IntSlice, v.(int)) }
func (h *hp) Pop() any {
	a := h.IntSlice
	v := a[len(a)-1]
	h.IntSlice = a[:len(a)-1]
	return v
}

```
模版2
```go
type hp []int
func (h hp) Len() int{
	return len(h)
}
func (h hp) Swap(i, j int){
	h[i],h[j]=h[j],h[i]
}
func (h hp) Less(i, j int) bool{
	return h[i]>h[j]
}
func (h *hp) Push(x any){
	*h=append(*h, x.(int))
}
func (h *hp)Pop() any{
	t:=*h
	x:=t[len(t)-1]
	*h=t[:len(t)-1]
	return x
}
func minStoneSum(piles []int, k int) int {
	h:=&hp{}
	for _,x:=range piles{
		heap.Push(h, x)
	}
	for i:=0; i<k; i++{
		x:=heap.Pop(h).(int)
		heap.Push(h, x-x/2)
	}
	ans:=0
	for _,x:=range *h{
		ans+=x
	}
	return ans
}
```
### 枚举
枚举子数组
需要 O（n^2）
### 二叉树：

124. 二叉树中的最大路径和(第一次没做出来)
https://leetcode.cn/problems/binary-tree-maximum-path-sum/solution/er-cha-shu-zhong-de-zui-da-lu-jing-he-by-ikaruga
b + a + c。
b + a + a 的父结点。
a + c + a 的父结点。
其中情况 1，表示如果不联络父结点的情况，或本身是根结点的情况。
这种情况是没法递归的，但是结果有可能是全局最大路径和。
情况 2 和 3，递归时计算 a+b 和 a+c，选择一个更优的方案返回，也就是上面说的递归后的最优解啦。
另外结点有可能是负值，最大和肯定就要想办法舍弃负值
但是上面 3 种情况，无论哪种，a 作为联络点，都不能够舍弃。

2415. 反转二叉树的奇数层
同时遍历一颗二叉树的左，右，并且交换
```go
	var dfs func(root1 *TreeNode, root2 *TreeNode, isodd bool)
	dfs = func(root1 *TreeNode, root2 *TreeNode, isodd bool){
		if root1==nil||root2==nil{
			return
		}
		if isodd{
			root1.Val,root2.Val=root2.Val,root1.Val
		}
		dfs(root1.Left, root2.Right, !isodd)
		dfs(root1.Right, root2.Left, !isodd)
	}
	dfs(root.Left, root.Right, true)
	return root
```

99. 恢复二叉搜索树(第一次没做出来)
中序遍历，同时记录前面的状态

114. 二叉树展开为链表(第一次能做出来)
思路1. 后序遍历

297. 二叉树的序列化与反序列化
一般是需要前序和中序，或者中序和后序的结果才能保证唯一的一颗二叉树，但如果存储了空指针的信息，只需要一种就可以做。前序，后序，层次均可以做

979. 在二叉树中分配硬币. ---贡献法，后序遍历

合法二叉搜索树

二叉树的最近公共祖先
注意，是看左右两边是否能找到p，q。如果都能找到，由于后序遍历，能保证第一个相交的点是最近公共祖先

```
分类讨论
当前节点是空节点
当前节点是p
当前节点是q

返回当前节点

否则
「左右子树都找到：返回当前节点
只有在左子树找到：返回递归左子树的结果
只有右子树找到：返回递归右子树的结果
左右子树都没有找到：返回空节点
```

二叉搜索树的最近公共祖先

```
分类讨论

当前节点是空节点（需要判断吗？）
p和q都在左子树：返回递归左子树的结果
p和q都在右子树：返回递归右子树的结果

p和q分别在左右子树
当前节点是p
当前节点是 q

返回当前节点
```

1123. 最深叶节点的最近公共祖先


222. 完全二叉树的节点个数
完全二叉树，左子树或者右子树一定有一个满二叉树，可以直接算出来节点

二叉序的前中后序遍历
```cpp
    vector<int> inorderTraversal(TreeNode* root) {
        stack<TreeNode*> s;
        vector<int> res;
        while(!s.empty() || root != nullptr){
            while(root != nullptr){
                s.push(root);
                root = root->left;
            }
			root = s.top();s.pop();
			res.push_back(root->val);
			root = root->right;
            
        }
        return res;
    }
```

```cpp
    vector<int> preorderTraversal(TreeNode* root) {
        stack<TreeNode*> s;
        vector<int> res;
        while(!s.empty() || root != nullptr){
            while(root != nullptr){
                s.push(root);
                res.push_back(root->val);
                root = root->left;
            }
            root = s.top();s.pop();
            root = root->right;
        }
        return res;
    }
```

```cpp
    vector<int> postorderTraversal(TreeNode* root) {
        stack<TreeNode*> s;
        vector<int> res;
        TreeNode* prev = nullptr;
        while(!s.empty() || root !=nullptr){
            while(root != nullptr){
                s.push(root);
                root = root->left;
            }
            root = s.top();s.pop();
            if(root->right == nullptr || root->right == prev){
                res.push_back(root->val);
                prev = root;
                root = nullptr;
            }else{
                s.push(root);
                root = root->right;
            }
        }
        return res;
    }
```

752. 解开密码锁
双向BFS，需要直到起点和终点 

### 图
求子树节点个数
思考每条边的贡献
2477. 到达首都的最少油耗
979. 在二叉树中分配硬币



### 动态规划:
300. 最长递增子序列
动态规划O(n^2)可以优化成O(nlogn) patience sorting-->扑克牌排序
https://leetcode.cn/problems/longest-increasing-subsequence/solutions/14796/dong-tai-gui-hua-she-ji-fang-fa-zhi-pai-you-xi-jia/
类似的题
1671. 得到山形数组的最少删除次数



1911. 最大子序列交替和.   状态的定义，还有选择，注意此时奇数和偶数是状态

1048. 最长字符串链

<<<<<<< HEAD
线性dp
2008. 出租车的最大盈利
2830. 销售利润最大化
=======
区间dp:
dp\[l]\[r], 从左区间到右区间取得的最大值
877. 石子游戏
486. 预测赢家
>>>>>>> origin/main


环形动态规划：
918. 环形子数组的最大和
213. 打家劫舍 II

树形DP
834. 树中距离之和.  主要是要推出来dp的关系
543. 二叉树的直径
2246. 相邻字符不同的最长路径
124. 二叉树中的最大路径和


### 位运算:
&^ 二元运算符的操作结果是“bit clear"  
a &^ b 的意思就是 清零a中，ab都为1的位
sync.Mutex中有应用: state &^ mutexWoken：更新状态，标识不存在抢锁的协程；

X % 2^n = X & (2^n – 1)
2^n表示2的n次方，也就是说，一个数对2^n取模 == 一个数和(2^n – 1)做按位与运算 。
这个应用在golang Map中增量扩容有应用

136. 只出现一次的数字 
350. 两个数组的交集 II
对应进阶问题三，如果内存十分小，不足以将数组全部载入内存，那么必然也不能使用哈希这类费空间的算法，只能选用空间复杂度最小的算法，即解法一。

但是解法一中需要改造，一般说排序算法都是针对于内部排序，一旦涉及到跟磁盘打交道（外部排序），则需要特殊的考虑。归并排序是天然适合外部排序的算法，可以将分割后的子数组写到单个文件中，归并时将小文件合并为更大的文件。当两个数组均排序完成生成两个大文件后，即可使用双指针遍历两个文件，如此可以使空间复杂度最低。

关于外部排序与JOIN，强烈推荐大家看一下 数据库内核杂谈（六）：表的 JOIN（连接）这一系列数据库相关的文章
https://www.infoq.cn/article/6XGx92FyQ45cMXpj2mgZ


1442. 形成两个异或相等数组的三元组数目
xor的前缀和

2429. 最小 XOR
```go
  c1 := bits.OnesCount(uint(num1)) //记录1的个数
  num1 |= num1 + 1  // 最低的 0 变成 1
  num1 &= num1 - 1  // 最低的 1 变成 0
```


### 双指针: 
颜色分类
26. 删除有序数组中的重复项.   AC
80. 删除有序数组中的重复项 II.  AC
264. 丑数II  三指针    AC
15. 三数之和

数学：
丑数I.   AC
189. 轮转数组. 最小公约数或者反转数组 
```java
public int gcd(int x, int y){  
    return y==0?x:gcd(y, x%y);  
}


面试题 16.01. 交换数字
不用临时变量，直接交换a和b的值
sol1: 数学，当成一个数轴,这种方法存在溢出的风险，不安全, eg\[-2147483647, 2147483647\]
int的最大值
int 的最小值。 -2147483648 
int 的最大值。  2147483647
```go
a=b-a
b=b-a
a=b+a
```
sol2: 位运算
```go
a=a^b
b=a^b
a=a^b
```

### 设计数据结构:
面试题 16.25. LRU 缓存--哈希双链表       AC
https://leetcode.cn/problems/lru-cache-lcci/solution/by-nehzil-zt9y/
https://leetcode.cn/problems/lru-cache/solution/yuan-yu-linkedhashmapyuan-ma-by-jeromememory/

LFU缓存.  

### 单调栈:
无论题目变成什么样，请记住一个核心原则：**及时移除无用数据，保证队列/栈的有序性**
只弹出右边的元素是单调栈，如果还要弹出左边元素就是单调队列

单调栈主要用于解决下一个更大元素类的问题，可以想象比身高的场景 
单调栈可以正序遍历，用每个数更新其他数的下一个更大元素
也可以逆序遍历，，对于每个数，找它的下一个更大元素
496. 下一个更大元素 I（单调栈模板题）  AC
503. 下一个更大元素 II     AC

```
整个nums里每个数字都有两条命，每当后面出现一个比他们大的数字的时候就要砍死自己一条命，而那个恰好把自己第二条命砍死的数字就是自己的secondGreater元素。  
所以我们设置两个赛道，胜者组里放的是还有两条命的，败者组里放的是只有一条命的，每当新到一个数字，他就会把两个赛道里的所有自己能砍死的数字都砍掉，如果砍掉的是胜者组里的，那被砍的就会掉到败者组去，如果砍掉的是败者组的，那自己就终结了他们的第二条命，成为了他们的secondGreater元素。  
为了方便让新来的数字大杀四方，我们决定把两个赛道里的数字都从大到小排整齐，这样新来的只要从后往前砍，直到砍不动了就行，砍爽了以后自己直接加入胜者组赛道的队尾接受制裁。  
有幸活到最后的数字，他们的secondGreater元素就是-1。  
以此类推，这题也可以做成thirdGreater。
```

1. 下一个更大元素 IV  双单调栈        AC
2. 每日温度          AC
3. 股票价格跨度          AC
4. 链表中的下一个更大节点        AC
5. 表现良好的最长时间段
6. 商品折扣后的最终价格
7. 132模式
8. 使数组按非递减顺序排列
9. 接雨水. 单调栈，也可以用动态规划，和双指针优化
10. 柱状图中最大的矩形. 单调栈，找左边和右边第一个更小的元素
11. 移掉 K 位数字
12. 去除重复字母
13. 子数组的最小值之和.  单调栈，找左边和右边第一个更小的元素，然后计算区间，注意要避免重复计算子数组
14. 美丽塔 I
2866. 美丽塔 II
16. 子数组范围和


### 纯算法思想
31下一个排列.  AC
556下一个更大元素III.   AC

### 单调队列
无论题目变成什么样，请记住一个核心原则：**及时移除无用数据，保证队列/栈的有序性**

面试题 59-II. 队列的最大值（单调队列模板题）AC
239. 滑动窗口最大值.   AC
1438. 绝对差不超过限制的最长连续子数组.    AC
862. 和至少为 K 的最短子数组
1499. 满足不等式的最大值.  也可以用堆

### 滑动窗口
需要满足单调性z
3. 无重复字符的最长子串
模板题
```go
func lengthOfLongestSubstring(s string) int {
    mp:=make(map[byte]int)
    l:=0
    n:=len(s)
    ans:=0
	//右侧不断更新寻找符合条件的答案
    for r:=0; r<n; r++{
        mp[s[r]]++
        //左边收缩得到最优解
        for ;l<r&&mp[s[r]]>1;l++{
            mp[s[l]]--
        }
        //更新答案
        ans=max(ans, r-l+1)
    }
    return ans
}
```
209. 长度最小的子数组.  AC
1438. 绝对差不超过限制的最长连续子数组.  AC
211. 最大连续1的个数 III.  AC
76. 最小覆盖子串 AC
滑动窗口--不断增加right，直到达到一个可行解，然后left优化
注意有个坑点:
```
Integer是对象  
Integer会缓存频繁使用的数值，  
数值范围为-128到127，在此范围内直接返回缓存值。  
超过该范围就会new 一个对象
要用equals比较
```
632. 最小区间
sol1:预处理后滑动窗口
sol2:排序k个有序链表

1423. 可获得的最大点数
正向思维逆向思维

### 链表
剑指 Offer II 024. 反转链表
剑指 Offer II 027. 回文链表

### 字典树
1233. 删除子文件夹
字典树模版
```go
type Trie struct{
	child map[string]*Trie
	fid int
}

func newTrie() *Trie{
	return &Trie{
		child: map[string]*Trie{},
		fid:   -1,
	}
}

func (this *Trie) insert(fid int, f string){
	ps:=strings.Split(f, "/")
	node:=this
	for _,p:=range ps[1:]{
		if _,ok:=node.child[p];!ok{
			node.child[p]=newTrie()
		}
		node = node.child[p]
	}
	node.fid = fid
}

func (this *Trie) SearchFirst() []int{
	ans:=make([]int, 0)
	var dfs func(root *Trie)
	dfs = func(root *Trie) {
		if root.fid!=-1{
			ans=append(ans, root.fid)
			return
		}
		for _,child:=range root.child{
			dfs(child)
		}
	}
	dfs(this)
	return ans
}
```
### 并查集
1631. 最小体力消耗路径

### 二分
33. 搜索旋转排序数组
153. 寻找旋转排序数组中的最小值
81. 搜索旋转排序数组 II

### 前后缀
1930. 长度为 3 的不同回文子序列
2484. 统计回文子序列数目


### 回溯
剑指 Offer II 079. 所有子集
注意下列代码，回溯过程中，子集会不断地变化，而我们希望在每次添加到结果列表时，都能保存子集的快照，而不是保存对同一个列表对象的引用。如果直接将 path 添加到 ans 中，那么后续对 path 的修改，会导致 ans 列表中的子集也会发生变化。
new ArrayList<>(path) 创建了一个新的 ArrayList 对象，并将当前的 path 内容复制到新对象中。这样，即使 path 在后续的回溯过程中发生了变化，已经添加到 ans 中的子集也不会受到影响。
```java
    public void backtrack(int[] nums, int step){  
        res.add(new LinkedList<>(track));  
//        res.add(track);  
        for(int i = step; i < nums.length; i++){  
            track.add(nums[i]);  
            backtrack(nums, i+1);  
            track.remove(track.size()-1);  
        }  
    }
```

869. 重新排序得到 2 的幂
46. 全排列
47. 全排列 II

78. 子集
```Go
//method1, 选哪个数
func subsets(nums []int) [][]int {
	ans:=make([][]int, 0)
	temp:=make([]int, 0)
	n:=len(nums)
	var backtrack func(idx int)
	backtrack = func(idx int) {
		tans:=make([]int, len(temp))
		copy(tans, temp)
		ans=append(ans, tans)
		if idx>=n{
			return
		}
		for i:=idx; i<n; i++{
			temp=append(temp, nums[i])
			backtrack(i+1)
			temp=temp[0:len(temp)-1]
		}
	}
	backtrack(0)
	return ans
}

//method2:选或者不选
func subsets(nums []int) [][]int {
    n := len(nums)
    ans := make([][]int, 0, 1<<n)
    path := make([]int, 0, n)
    var dfs func(int)
    dfs = func(i int) {
        if i == n {
            ans = append(ans, append([]int(nil), path...)) // 固定答案
            return
        }
        // 不选 nums[i]
        dfs(i + 1)
        // 选 nums[i]
        path = append(path, nums[i])
        dfs(i + 1)
        path = path[:len(path)-1] // 恢复现场
    }
    dfs(0)
    return ans
}
```

90. 子集 II
```go
func subsetsWithDup(nums []int) [][]int {
	sort.Ints(nums)
	n:=len(nums)
	ans:=make([][]int, 0, 2<<n)
	path:=make([]int, 0)
	var backtrack func(idx int)
	vis:=make([]bool, n)
	backtrack = func(idx int) {
		tans:=make([]int, len(path))
		copy(tans, path)
		ans=append(ans, tans)
		for i:=idx; i<n; i++{
			//进行去重
			if i>0&&!vis[i-1]&&nums[i]==nums[i-1]{
				continue
			}
			vis[i]=true
			path=append(path, nums[i])
			backtrack(i+1)
			path=path[0:len(path)-1]
			vis[i]=false
		}
		return
	}
	backtrack(0)
	return ans
}
```

### 多路归并
https://lfool.github.io/LFool-Notes/algorithm/%E5%A4%9A%E8%B7%AF%E5%BD%92%E5%B9%B6%E6%8A%80%E5%B7%A7%E6%80%BB%E7%BB%93.html
23. 合并 K 个升序链表
264. 丑数 II
313. 超级丑数

下面的题既可以多路归并也可以二分做

1. 查找和最小的 K 对数字
2. 第 K 个最小的素数分数
3. 子数组和排序后的区间和
4. 找出第 K 小的数对距离
5. 有序矩阵中的第 k 个最小数组和



剑指 Offer II 080. 含有 k 个元素的组合
46. 全排列（中等）
47. 全排列 II（中等）：思考为什么造成了重复，如何在搜索之前就判断这一支会产生重复；
39. 组合总和（中等）
40. 组合总和 II（中等）
77. 组合（中等）
78. 子集（中等）
90. 子集 II（中等）：剪枝技巧同 47 题、39 题、40 题；
60. 第 k 个排列（中等）：利用了剪枝的思想，减去了大量枝叶，直接来到需要的叶子结点；
93. 复原 IP 地址（中等）
https://leetcode.cn/problems/subsets/solution/c-zong-jie-liao-hui-su-wen-ti-lei-xing-dai-ni-gao-/
https://leetcode.cn/problems/zi-fu-chuan-de-pai-lie-lcof/solution/c-zong-jie-liao-hui-su-wen-ti-lei-xing-dai-ni-ga-4/

37. 解数独    需要知道如何求box内是否有数字冲突
52. N 皇后 II.  需要知道对角线是否冲突
79. 单词搜索

### BFS
BFS可以用来最优解
773. 滑动谜题


### 树状数组
树状数组本质是删减的线段树，适合单点修改，区间查询
前置知识lowbit
非负整数n在二进制表示下最低位 1及其后面的0构成的数值
e.g. lowbit (44) = lowbit ((101100)2) = (100)2 = 4

~n+1=-n （~表示取反）
lowbit (n) =n& (~ n + 1)= n& -n

307. 区域和检索 - 数组可修改
树状数组模版题
```go
type NumArray struct {
	nums []int
	tree []int
}

func lowbit(x int) int{
	return x&(-x)
}

func (this *NumArray) presum (idx int) int{
	res:=0
	for ;idx>0; idx-=lowbit(idx){
		res+=this.tree[idx]
	}
	return res
}

func Constructor(nums []int) NumArray {
	a:=NumArray{
		nums: make([]int, len(nums)),
		tree: make([]int, len(nums)+1),
	}
	for i,x :=range nums{
		a.Update(i, x)
	}
	return a
}


func (this *NumArray) Update(index int, val int)  {
	del:= val-this.nums[index]
	this.nums[index]=val
	for i:=index+1; i<len(this.tree); i+=lowbit(i){
		this.tree[i]+=del
	}
}

func (this *NumArray) SumRange(left int, right int) int {
	return this.presum(right+1)-this.presum(left)
}

```


## 算法八股
### 在线算法和离线算法的区别？
在线算法
在计算机科学中，一个在线算法是指它可以以序列化的方式一个个的处理输入，也就是说在开始时并不需要已经知道所有的输入。相对的，对于一个[离线算法](http://baike.baidu.com/view/2734232.htm)，在开始时就需要知道问题的所有输入数据，而且在解决一个问题后就要立即输出结果。例如，[选择排序](http://baike.baidu.com/view/547263.htm)在排序前就需要知道所有待排序元素，然而[插入排序](http://baike.baidu.com/view/396887.htm)就不必。

因为在线算法并不知道整个的输入，所以它**被迫做出的选择最后可能会被证明不是最优的**，对在线算法的研究主要集中在当前环境下怎么做出选择。对相同问题的在线算法和[离线算法](http://baike.baidu.com/view/2734232.htm)的对比分析形成了以上观点。如果想从其他角度了解在线算法可以看一下 流算法（关注精确呈现过去的输入所使用的内存的量），动态算法（关注维护一个在线输入的结果所需要的[时间复杂度](http://baike.baidu.com/view/104946.htm)）和在线机器学习。

> ​ **简单来说就是：必须即时回答每一个询问，不能等到收到所有询问后再统一处理。**

一个很好的展示在线算法概念的例子是加拿大旅行者问题，这个问题的目标是在一个有权图中以最小的代价到达一个目标节点，但这个有权图中有些边是不可靠的，可能已经被剔除。然而一个旅行者只有到某个边的一个端点时才能确定该边是否已经被移除了。最坏情况下，该问题会变得简单，即所有的不确定的边都被移除该问题将会变成通常的[最短路径](http://baike.baidu.com/view/349189.htm)问题。

离线算法
离线算法设计策略都是基于在执行算法前输入数据已知的基本假设，也就是说，对于一个离线算法，在开始时就需要知道问题的所有输入数据，而且在解决一个问题后就要立即输出结果，通常将这类具有问题完全信息前提下设计出的算法成为离线算法。

如1851. 包含每个查询的最小区间. 离线算法+优先队列


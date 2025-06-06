题解：
https://github.com/EndlessCheng/codeforces-go/blob/master/leetcode/SOLUTIONS.md
https://leetcode.cn/problems/shortest-subarray-with-sum-at-least-k/solution/liang-zhang-tu-miao-dong-dan-diao-dui-li-9fvh/

### 排序
快速排序
```go
import "math/rand"
import "time"
func sortArray(nums []int) []int {
    rand.Seed(time.Now().UnixNano())
    quickSort(nums, 0, len(nums)-1)
    return nums
}
func quickSort(nums []int, l, r int){
    if l>=r{
        return 
    }
    idx:=partition(nums, l, r)
    quickSort(nums, l, idx-1)
    quickSort(nums, idx+1, r)
}
func partition(nums []int, l, r int) int{
    rd:=rand.Int()%(r-l+1)+l
    nums[l],nums[rd]=nums[rd],nums[l]
    pivot:=nums[l]
    for l<r{
        for l<r&&nums[r]>=pivot{
            r--
        }
        nums[l]=nums[r]
        for l<r&&nums[l]<=pivot{
            l++
        }
        nums[r]=nums[l]
    }
    nums[l]=pivot
    return l
}
```
### 数组
41. 缺失的第一个正数
sol1: 原地交换
```go
func firstMissingPositive(nums []int) int {
    n:=len(nums)
    for i:=0; i<n; i++{
	    //nums[i]!=nums[nums[i]-1]条件很重要而不是nums[i]!=i+1,会死循环
        for nums[i]>=1&&nums[i]<=n&&nums[i]!=nums[nums[i]-1]{
            nums[i],nums[nums[i]-1]=nums[nums[i]-1],nums[i]
        }
    }
    for i,x:=range nums{
        if x!=i+1{
            return i+1
        }
    }
    return n+1
}
```
sol2:原地哈希
```go
func firstMissingPositive(nums []int) int {
    n:=len(nums)
    for i,x:=range nums{
        if x<=0{
            nums[i]=(n+1)
        }
    }
    for _,x:=range nums{
        if x>=1&&x<=n&&!(nums[x-1]<0){
            nums[x-1]=-nums[x-1]
        }else if -x>=1&&-x<=n&&!(nums[-x-1]<0){
            nums[-x-1]=-nums[-x-1]
        }
    }
    for i,x:=range nums{
        if x>0{
            return i+1
        }
    }
    return n+1
}
```

189. 轮转数组


### 模拟
43. 字符串相乘
sol1:做加法(普通竖式)
sol2:做乘法(优化竖式)
https://leetcode.cn/problems/multiply-strings/solutions/188815/gao-pin-mian-shi-xi-lie-zi-fu-chuan-cheng-fa-by-la/
```
想要做出这道题，需要知道一个数学定理：
两个长度分别为n和m的数相乘，长度不会超过n+m
```
```go
func multiply(num1 string, num2 string) string {
    if num1=="0"||num2=="0"{
        return "0"
    }
    n:=len(num1)
    m:=len(num2)
    by:=make([]int, n+m)
    for i:=n-1; i>=0; i--{
        for j:=m-1; j>=0; j--{
            idx:=i+j+1
            add:=int(num1[i]-'0')*int(num2[j]-'0')
            by[idx]+=add%10
            carry:=by[idx]/10
            by[idx]=by[idx]%10
            by[idx-1]+=carry+add/10
        }
    }
    idx:=0
    ans:=""
    for ;idx<len(by)&&by[idx]==0;idx++{}
    for ;idx<len(by);idx++{
        ans+=strconv.Itoa(by[idx])
    }
    return ans
}
```
54. 螺旋矩阵
48. 旋转图像
```
上下对称：matrix[i][j] -> matrix[n-i-1][j]，（列不变）
左右对称：matrix[i][j] -> matrix[i][n-j-1]，（行不变）
主对角线对称：matrix[i][j] -> matrix[j][i]，（行列互换）
副对角线对称：matrix[i][j] -> matrix[n-j-1][n-i-1] （行列均变，且互换）
```
旋转90度-->上下对称 + 主对角线对称或者主对角线对称 + 左右对称
```go
func rotate(matrix [][]int)  {
    upDownSym(matrix)
    mainDiagSym(matrix)
    return 
}
func upDownSym(matrix [][]int){
    n:=len(matrix)
    for i:=0; i<n/2; i++{
        for j:=0; j<n; j++{
            matrix[i][j],matrix[n-1-i][j]=matrix[n-1-i][j],matrix[i][j]
        }
    }
}
func mainDiagSym(matrix [][]int){
    n:=len(matrix)
    for i:=0; i<n; i++{
        for j:=i+1; j<n; j++{
            matrix[i][j],matrix[j][i]=matrix[j][i],matrix[i][j]
        }
    }
}
```

50. 购买两块巧克力
动态维护最小值和次小值
```go
func buyChoco(prices []int, money int) int {
	mn1, mn2 := math.MaxInt, math.MaxInt
	for _, p := range prices {
		if p < mn1 {
			mn2 = mn1
			mn1 = p
		} else if p < mn2 {
			mn2 = p
		}
	}
	if mn1+mn2 <= money {
		return money - mn1 - mn2
	}
	return money
}
```

### 前缀和
https://leetcode.cn/problems/subarray-sum-equals-k/solutions/562174/de-liao-yi-wen-jiang-qian-zhui-he-an-pai-yhyf/?envType=study-plan-v2&envId=top-100-liked
560. 和为 K 的子数组
前缀和+hash

### 哈希
128. 最长连续序列

### 堆
1962. 移除石子使总数最小
堆模版
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

215. 数组中的第K个最大元素
sol1:求k最大用小根堆 O(nlogk)
sol2:应用快速排序的partition思想
```go
func findKthLargest(nums []int, k int) int {
	n:=len(nums)
	return quickSelect(nums, 0, n-1, n-k)
}

func quickSelect(nums []int, l, r, k int) int{
	idx:=partition(nums, l, r)
	if idx<k{
		return quickSelect(nums, idx+1, r, k)
	}else if idx>k{
		return quickSelect(nums, l, idx-1, k)
	}
	return nums[idx]
}

func partition(nums []int, l, r int) int{
	rd:=rand.Int()%(r-l+1)+l
	nums[l],nums[rd]=nums[rd],nums[l]
	pivot:=nums[l]
	for l<r{
		for l<r&&nums[r]>=pivot{
			r--
		}
		nums[l]=nums[r]
		for l<r&&nums[l]<=pivot{
			l++
		}
		nums[r]=nums[l]
	}
	nums[l]=pivot
	return l
}
```

295. 数据流的中位数
经典题目大小堆找中位数
```go
//小堆
type minhp []int
func (h minhp) Len() int{
    return len(h)
}
func (h minhp) Swap(i, j int){
    h[i],h[j]=h[j],h[i]
}
func (h minhp) Less(i, j int) bool{
    return h[i]<h[j]
}
func (h *minhp) Push(x any){
    *h=append(*h, x.(int))
}
func (h *minhp) Pop() any{
    t:=*h
    x:=t[len(t)-1]
    *h=t[0:len(t)-1]
    return x
}
//大堆
type maxhp []int
func (h maxhp) Len() int{
    return len(h)
}
func (h maxhp) Swap(i, j int){
    h[i],h[j]=h[j],h[i]
}
func (h maxhp) Less(i, j int) bool{
    return h[i]>h[j]
}
func (h *maxhp) Push(x any){
    *h=append(*h, x.(int))
}
func (h *maxhp) Pop() any{
    t:=*h
    x:=t[len(t)-1]
    *h=t[0:len(t)-1]
    return x
}
type MedianFinder struct {
    lh maxhp
    rh minhp
}
func Constructor() MedianFinder {
    m:=MedianFinder{
        lh:maxhp{},
        rh:minhp{},
    }
    return m
}
func (this *MedianFinder) AddNum(num int)  {
    tlen:=len(this.lh)+len(this.rh)
    if tlen&1==0{
		heap.Push(&this.rh, num)
		x:=heap.Pop(&this.rh).(int)
		heap.Push(&this.lh, x)
    }else{
        //奇数
        heap.Push(&this.lh, num)
        x:=heap.Pop(&this.lh).(int)
        heap.Push(&this.rh, x)
    }
    return 
}
func (this *MedianFinder) FindMedian() float64 {
    tlen:=len(this.lh)+len(this.rh)
    if tlen&1==0{
        return float64(this.lh[0]+this.rh[0])/2
    }
    return float64(this.lh[0])
}
```
### 枚举
枚举子数组
需要 O（n^2）

### 区间问题
252. 会议室
56. 合并区间
57. 插入区间
1288. 删除被覆盖区间
228. 汇总区间

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
```go
func lengthOfLIS(nums []int) int {
    piles:=make([]int, 0)
    for _,x:=range nums{
        idx:=bisearch(piles, x)
        if idx==len(piles){
            piles=append(piles, x)
        }else{
            piles[idx]=x
        }
    }
    return len(piles)
}

func bisearch(nums []int, tar int) int{
    n:=len(nums)
    l,r:=-1,n
    for l+1!=r{
        mid:=l+(r-l)/2
        if tar>nums[mid]{
            l=mid
        }else{
            r=mid
        }
    }
    return r
}
```

类似的题
1671. 得到山形数组的最少删除次数

1911. 最大子序列交替和.   状态的定义，还有选择，注意此时奇数和偶数是状态

1048. 最长字符串链

序列dp
139. 单词拆分
32. 最长有效括号
结合栈和dp


线性dp
2008. 出租车的最大盈利
2830. 销售利润最大化
=======
区间dp:
dp\[l]\[r], 从左区间到右区间取得的最大值
877. 石子游戏
486. 预测赢家

环形动态规划：
918. 环形子数组的最大和
213. 打家劫舍 II

树形DP
834. 树中距离之和.  主要是要推出来dp的关系
543. 二叉树的直径
2246. 相邻字符不同的最长路径
124. 二叉树中的最大路径和

字符串类:
5. 最长回文子串
sol1:dp
sol2:中心扩散
首先往左寻找与当期位置相同的字符，直到遇到不相等为止。
然后往右寻找与当期位置相同的字符，直到遇到不相等为止。
最后左右双向扩散，直到左和右不相等
均为O(n^2)

2707. 字符串中的额外字符
求字符串的子串要O(n^3)
sol1: 哈希+dp O(n^3)
sol2:Trie+dp O(n^2)

1143. 最长公共子序列

2645. 构造有效字符串的最少插入数

记忆化转dp
LCR 166. 珠宝的最高价值

### 贪心
55. 跳跃游戏
45. 跳跃游戏 II
正向跳跃, 更新能跳得最远的位置

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

260.只出现一次的数字 III
https://leetcode.cn/problems/single-number-iii/solutions/2484352/tu-jie-yi-zhang-tu-miao-dong-zhuan-huan-np9d2/

```go
func singleNumber(nums []int) []int {
    two:=0
    for _,x:=range nums{
        two^=x
    }
    ans:=make([]int, 2)
    lowb:=two&(^two+1)
    for _,x:=range nums{
        if x&lowb==0{
            ans[0]^=x
        }else{
            ans[1]^=x
        }
    }
    return ans
}
```

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

面试题 16.01. 交换数字
不用临时变量，直接交换a和b的值
sol1: 数学，当成一个数轴,这种方法存在溢出的风险，不安全, eg\[-2147483647, 2147483647\]
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
287. 寻找重复数
sol1:位运算
sol2:快慢指针
sol3:原地哈希  类似41. 缺失的第一个正数
### 双指针: 
75. 颜色分类
该题为经典的荷兰国旗问题，由于题目本质是要我们将数分成三段
题解如下: 
https://leetcode.cn/problems/sort-colors/solutions/1868577/by-ac_oier-7lwk/?envType=study-plan-v2&envId=top-100-liked
```go
func sortColors(nums []int)  {
    l:=0
    r:=len(nums)-1
    i:=0
    for i<=r{
        if nums[i]==1{
            i++
        }else if nums[i]==0{
            nums[l],nums[i]=nums[i],nums[l]
            l++
            i++
        }else{
            nums[i],nums[r]=nums[r],nums[i]
            r--
        }
    }
    return 
}
```
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
134.加油站

### 单调队列
无论题目变成什么样，请记住一个核心原则：**及时移除无用数据，保证队列/栈的有序性**

239. 滑动窗口最大值
单调队列模板
```go
type MonoQue struct{
	que []int
}
func (q *MonoQue) Push(x int){
	for len(q.que)>0&&x>q.que[len(q.que)-1]{
		q.que=q.que[:len(q.que)-1]
	}
	q.que=append(q.que, x)
}
func (q *MonoQue) PeekMax() int{
	return q.que[0]
}
func (q *MonoQue) PopMax(x int) int{
	maxx:=q.que[0]
	if x==maxx{
		q.que=q.que[1:]
		return maxx
	}
	return -1
}

func maxSlidingWindow(nums []int, k int) []int {
	q:=&MonoQue{que: make([]int, 0)}
	n:=len(nums)
	ans:=make([]int, n-k+1)
	for i:=0; i<k; i++{
		q.Push(nums[i])
	}
	ans[0]=q.PeekMax()
	for i:=1; i<n-k+1;i++{
		end:=i+k-1
		q.PopMax(nums[i-1])
		q.Push(nums[end])
		ans[i]=q.PeekMax()
	}
	return ans
}
```

面试题 59-II. 队列的最大值（单调队列模板题）AC
239. 滑动窗口最大值.   AC
1438. 绝对差不超过限制的最长连续子数组.    AC
862. 和至少为 K 的最短子数组
1499. 满足不等式的最大值.  也可以用堆


### 链表
剑指 Offer II 024. 反转链表
剑指 Offer II 027. 回文链表
160. 相交链表
142. 环形链表 II
```
根据：
1. f=2s （快指针每次2步，路程刚好2倍）
2. f = s + nb (相遇时，刚好多走了n圈）
推出：s = nb
从head结点走到入环点需要走 ： a + nb， 而slow已经走了nb，那么slow再走a步就是入环点了。
如何知道slow刚好走了a步？ 从head开始，和slow指针一起走，相遇时刚好就是a步
```
24. 两两交换链表中的节点
迭代

206. 反转链表
```go
func reverse(root *ListNode) *ListNode{
    if root==nil||root.Next==nil{
        return root
    }
    last:=reverse(root.Next)
    root.Next.Next=root
    root.Next=nil
    return last
}
```
进阶题: 反转链表前 N 个节点
sol: 需要存后继
```go
var successor *ListNode // 后继节点
// 反转以 head 为起点的 n 个节点，返回新的头结点
func reverseN(head *ListNode, n int) *ListNode {
    if n == 1 {
        // 记录第 n + 1 个节点
        successor = head.Next
        return head
    }
    // 以 head.Next 为起点，需要反转前 n - 1 个节点
    last := reverseN(head.Next, n-1)
    head.Next.Next = head
    head.Next = successor
    return last
}
```
92. 反转链表 II, 反转left到right的节点
sol: 先实现反转前N个节点, 然后递归处理, 需要存后继
```go
var successor *ListNode
func reverseN(root *ListNode, n int) *ListNode{
    if n==1{
        successor=root.Next
        return root
    }
    last:=reverseN(root.Next, n-1)
    root.Next.Next=root
    root.Next=successor
    return last
}

func reverseBetween(head *ListNode, left int, right int) *ListNode {
    if left==1{
        return reverseN(head, right)
    }
    last:=reverseBetween(head.Next, left-1,right-1)
    head.Next=last
    return head
}
```
25. K 个一组翻转链表
sol: 先实现反转前N个节点, 然后递归反转k个节点, 不需要存后继
```go
func reverseN(head *ListNode, n int) *ListNode{
    if n==1{
        return head
    }
    last:=reverseN(head.Next, n-1)
    head.Next.Next=head
    head.Next=nil
    return last
}

func reverseKGroup(head *ListNode, k int) *ListNode {
    if head==nil{
        return nil
    }
    cur:=head
    for i:=0; i<k; i++{
        if cur==nil{
            return head
        }
        cur=cur.Next
    }
    last:=reverseN(head, k)
    head.Next=reverseKGroup(cur, k)
    return last
}
```
234. 回文链表
空间O(1)sol: 找到链表中点, 反转后半链表，然后比较
23. 合并 K 个升序链表
sol:heap多路归并
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


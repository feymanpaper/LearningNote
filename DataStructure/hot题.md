### 动态规划
#### 字符串dp
1143. 最长公共子序列
72. 编辑距离
#### 背包问题
#### 股票问题
https://leetcode.cn/circle/discuss/qiAgHn/
dp\[i]\[0]表示第i天不持有股票
dp\[i]\[1]表示第i天持有股票
121. 买卖股票的最佳时机
188的k=1版本
```go
func maxProfit(prices []int) int {
    n:=len(prices)
    dp:=[2][2]int{}
    dp[0][0]=0
    dp[0][1]=-prices[0]
    for i:=1; i<n; i++{
        dp[i&1][0]=max(dp[(i-1)&1][1]+prices[i], dp[(i-1)&1][0])
        dp[i&1][1]=max(dp[(i-1)&1][1], -prices[i])
    }
    return dp[(n-1)&1][0]
}
```
122. 买卖股票的最佳时机 II
188的k=无穷大版本
```go
func maxProfit(prices []int) int {
    n:=len(prices)
    dp:=[2][2]int{}
    dp[0][0]=0
    dp[0][1]=-prices[0]
    for i:=1; i<n; i++{
        dp[i&1][0]=max(dp[(i-1)&1][0],dp[(i-1)&1][1]+prices[i])
        dp[i&1][1]=max(dp[(i-1)&1][1], dp[(i-1)&1][0]-prices[i])
    }
    return dp[(n-1)&1][0]
}
```
123. 买卖股票的最佳时机 III
sol: 188题目 k=2版本
188. 买卖股票的最佳时机 IV
通解问题
需要再增加一个状态k记录最大交易次数
sol1: 时间复杂度O(nk), 空间复杂度O(nk)
```go
func maxProfit(k int, prices []int) int {
    n:=len(prices)
    dp:=make([][][]int, n)
    for i:=0; i<n; i++{
        dp[i]=make([][]int, k+1)
        for j:=0; j<=k; j++{
            dp[i][j]=make([]int, 2)
        }
    }
    dp[0][0][1]=-math.MinInt/2
    for i:=1; i<=k; i++{
        dp[0][i][1]=-prices[0]
    }
    for i:=1; i<n; i++{
        for j:=1; j<=k; j++{
            dp[i][j][0]=max(dp[i-1][j][0],dp[i-1][j][1]+prices[i])
            dp[i][j][1]=max(dp[i-1][j][1],dp[i-1][j-1][0]-prices[i])
        }
    }
    return dp[n-1][k][0]
}
```
sol2:优化空间，注意到(i,k)依赖于(i-1,k), (i-1,k-1)，可以优化空间O(k)
```go
func maxProfit(k int, prices []int) int {
    n:=len(prices)
    dp:=make([][]int, k+1)
    for i:=0; i<=k; i++{
        dp[i]=make([]int, 2)
    }
    dp[0][1]=-math.MinInt/2
    for i:=1; i<=k; i++{
        dp[i][1]=-prices[0]
    }
    for i:=1; i<n; i++{
        for j:=k; j>=1; j--{
            dp[j][0]=max(dp[j][0],dp[j][1]+prices[i])
            dp[j][1]=max(dp[j][1],dp[j-1][0]-prices[i])
        }
    }
    return dp[k][0]
}
```
714. 买卖股票的最佳时机含手续费
sol: k =无穷大带手续费，只需要在买的时候算手续费就好
```go
func maxProfit(prices []int, fee int) int {
    n:=len(prices)
    dp:=make([][2]int, n)
    dp[0][1]=-prices[0]-fee
    for i:=1; i<n; i++{
        dp[i][0]=max(dp[i-1][0], dp[i-1][1]+prices[i])
        dp[i][1]=max(dp[i-1][1], dp[i-1][0]-prices[i]-fee)
    }
    return dp[n-1][0]
}
```
#### 打家劫舍问题 
198. 打家劫舍
```go
func rob(nums []int) int {
    n:=len(nums)
    dp:=make([][2]int, n)
    dp[0][0]=0
    dp[0][1]=nums[0]
    for i:=1; i<n; i++{
        dp[i][0]=max(dp[i-1][0], dp[i-1][1])
        dp[i][1]=max(dp[i-1][0]+nums[i])
    }
    ans:=max(dp[n-1][0], dp[n-1][1])
    //打印路径
    target:=ans
    path:=make([]int, 0)
    for i:=n-1; i>=0; i--{
        if dp[i][1]==target{
            path=append(path, i)
            target-=nums[i]
        }
    }
    fmt.Println(path)
    return ans 
}
```
213. 打家劫舍 II
环形的房子，则答案为两者取其一
```go
func rob(nums []int) int {
    n:=len(nums)
    if n<=1{
        return nums[0]
    }
    return max(robsin(nums, 0, n-2), robsin(nums, 1, n-1))
}

func robsin(nums []int, st, end int) int{
    dp:=make([][2]int, end+1)
    dp[st][0]=0
    dp[st][1]=nums[st]
    for i:=st+1; i<=end; i++{
        dp[i][0]=max(dp[i-1][0], dp[i-1][1])
        dp[i][1]=max(dp[i-1][0]+nums[i])
    }
    return max(dp[end][0], dp[end][1])
}
```
337. 打家劫舍 III
树形dp
```go
func rob(root *TreeNode) int {
    return max(subrob(root))
}

func subrob(root *TreeNode) (int, int){
    if root==nil{
        return 0,0
    }
    lrob,lnot:=subrob(root.Left)
    rrob,rnot:=subrob(root.Right)
    rob:=root.Val+lnot+rnot
    not:=max(lnot, lrob)+max(rnot, rrob)
    return rob, not
}
```
### 二分
#### 搜索旋转数组系列
该系列比较难抽象成红蓝二分，这里给的抽象是l及左边都不满足条件, r及右边都不满足条件

33. 搜索旋转排序数组
注意是每个数是互不相同的

搞懂这个题的精髓在于三个定理
定理一：只有在顺序区间内才可以通过区间两端的数值判断target是否在其中。
定理二：判断顺序区间还是乱序区间，只需要对比 left 和 right 是否是顺序对即可，left <= right，顺序区间，否则乱序区间
定理三：每次二分都会至少存在一个顺序区间
通过不断的用Mid二分，根据定理二，将整个数组划分成顺序区间和乱序区间，然后利用定理一判断target是否在顺序区间，如果在顺序区间，下次循环就直接取顺序区间，如果不在，那么下次循环就取乱序区间
```
将数组一分为二，其中一定有一个是有序的，另一个可能是有序，也能是部分有序。
此时有序部分用二分法查找。无序部分再一分为二，其中一个一定有序，另一个可能有序，可能无序。就这样循环
```
红蓝二分的运用
此处红蓝二分的定义是，l表示的是左边不符合条件的，r表示的是右边不符合条件的
https://leetcode.cn/problems/search-in-rotated-sorted-array/solutions/1018654/bu-yong-ji-mo-ban-de-er-fen-cha-zhao-shi-ytqu/

```go
func search(nums []int, target int) int {
    n:=len(nums)
    l:=-1
    r:=n
    for l+1!=r{
        mid:=l+(r-l)>>1
        if nums[mid]==target{
            return mid
        }else if nums[l+1]<=nums[mid]{
            if nums[l+1]<=target&&target<nums[mid]{
                r=mid
            }else{
                l=mid
            }
        }else{
            if nums[mid]<target&&target<=nums[r-1]{
                l=mid
            }else{
                r=mid
            }
        }
    }
    return -1
}
```

81. 搜索旋转排序数组 II
和33不同之处在于数字有可能重复，那么只需要在nums l+1 == nums mid时让l++即可
时间复杂度在极端情况下, 比如数字全部相等, 且target不是该数字, 会退化成O(n)
```go
func search(nums []int, target int) bool {
    n:=len(nums)
    l:=-1
    r:=n
    for l+1!=r{
        mid:=l+(r-l)>>1
        if nums[mid]==target{
            return true
        }
        if nums[l+1]<nums[mid]{
            if nums[l+1]<=target&&target<nums[mid]{
                r=mid
            }else{
                l=mid
            }
        }else if nums[l+1]>nums[mid]{
            if nums[mid]<target&&target<=nums[r-1]{
                l=mid
            }else{
                r=mid
            }
        }else{
            l++
        }
    }
    return false
}
```

153. 寻找旋转排序数组中的最小值
注意是每个数是互不相同的
依照上一题思路的进阶，如果左边有序，那么最小值为l+1, 如果右边有序，最小值为mid
```go
func findMin(nums []int) int {
    ans:=math.MaxInt/2
    l:=-1
    r:=len(nums)
    for l+1!=r{
        mid:=l+(r-l)>>1
        if nums[l+1]<=nums[mid]{
            ans=min(ans, nums[l+1])
            l=mid
        }else{
            ans=min(ans, nums[mid])
            r=mid
        }
    }
    return ans
}
```

154. 寻找旋转排序数组中的最小值 II
 和153相比存在重复的元素，那么当存在重复元素时，只需要让l++，同时为了避免极端情况下, 全部元素相同，退化成O(n), 因此需要及时更新最小值
```go
func findMin(nums []int) int {
    ans:=math.MaxInt/2
    l:=-1
    r:=len(nums)
    for l+1!=r{
        mid:=l+(r-l)>>1
        if nums[l+1]<nums[mid]{
            ans=min(ans, nums[l+1])
            l=mid
        }else if nums[l+1]>nums[mid]{
            ans=min(ans, nums[mid])
            r=mid
        }else{
            ans=min(ans, nums[mid])
            l++
        }
    }
    return ans
}
```

面试题 10.03. 搜索旋转数组
红蓝二分，l及左边的是不满足条件的, r及右边是不满足条件的
```go
func search(arr []int, target int) int {
    n:=len(arr)
    l:=-1
    r:=n
    for l+1!=r{
	    // l和左边都不满足条件，如果l+1满足，说明是第一个满足的, 返回
        if arr[l+1]==target{
            return l+1
        }
        mid:=l+(r-l)>>1
        // mid满足, 则更新r=mid+1, 表示mid+1及右边的都不满足条件
        if arr[mid]==target{
            r=mid+1
        }else if arr[l+1]<arr[mid]{ //其余的按照正常的处理
            if target>=arr[l+1]&&target<arr[mid]{
                r=mid
            }else{
                l=mid
            }
        }else if arr[l+1]>arr[mid]{
            if target>arr[mid]&&target<=arr[r-1]{
                l=mid
            }else{
                r=mid
            }
        }else{
            l++
        }
    }
    return -1
}
```


#### 难题
4. 寻找两个正序数组的中位数
https://leetcode.cn/problems/median-of-two-sorted-arrays/solutions/210764/di-k-xiao-shu-jie-fa-ni-zhen-de-dong-ma-by-geek-8m/?envType=study-plan-v2&envId=top-100-liked
思路：转化为求两个正序数组中的第k个数， 折半删除
如何求两个正序数组中的第k个数？
两个数组，大小分别为m, n
每次从num1取第k/2个数a, 从num2去取k/2个数b
如果a\<b,
那么比num1中比a大的数有m-k/2, num2中比a大的数有, n-k/2+1, 那么总共比a大的就有
m+n-k+1个，这里面肯定包含第k个数, 因此可以直接去掉a及a之前的数
如果a=b也满足上述情况
```go
func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
    m:=len(nums1)
    n:=len(nums2)
    k:=(m+n)/2
    // 这里的第k是第中位数的数组的下标, 实际上第k个数是第k+1个
    if (m+n)&1==0{
        //偶数
        return (findKth(nums1, nums2, 0, 0, k)+findKth(nums1, nums2, 0, 0, k+1))/2
    }
    return findKth(nums1, nums2, 0, 0, k+1)
}

// 求两个有序数组第k个数, k从1开始而不是0
func findKth(nums1, nums2 []int, ast, bst, k int) float64{
    if ast>=len(nums1){
        return float64(nums2[bst+k-1])
    }
    if bst>=len(nums2){
        return float64(nums1[ast+k-1])
    }
    if k==1{
        return float64(min(nums1[ast], nums2[bst]))
    }
    midVal1:=math.MaxInt/2
    midVal2:=math.MaxInt/2
    if ast+k/2-1<len(nums1){
        midVal1=nums1[ast+k/2-1]
    }
    if bst+k/2-1<len(nums2){
        midVal2=nums2[bst+k/2-1]
    }

    if midVal1<midVal2{
        return findKth(nums1, nums2, ast+k/2, bst, k-k/2)
    }
    return findKth(nums1, nums2, ast, bst+k/2, k-k/2)
}
```
#### 二维二分
240. 搜索二维矩阵 II
从右上角开始O(m+n)
74. 搜索二维矩阵
sol1:从右上角开始O(m+n)，没有利用性质
sol2:两次二分，先二分行，再二分列
### 双指针
#### 42. 接雨水
https://leetcode.cn/problems/trapping-rain-water/solutions/1974340/zuo-liao-nbian-huan-bu-hui-yi-ge-shi-pin-ukwm/?envType=study-plan-v2&envId=top-100-liked
sol1: 前后缀分解, 接到的雨水取决于min(左边最高, 右边最高)-h
sol2:双指针, 前后缀分解的进阶，发现只需要维护双指针，p, q；pre, p表示之前的左边最高, q,suf表示右边的最高, 答案只取决于min(pre, suf)
```go
func trap(height []int) int {
    n:=len(height)
    p:=0
    q:=n-1
    pre:=0
    suf:=0
    ans:=0
    //p==q会好一点，尽管当p==q时, 增量为0
    for p<=q{
        pre=max(pre, height[p])
        suf=max(suf, height[q])
        if pre<suf{
            ans+=(pre-height[p])
            p++
        }else{
            ans+=(suf-height[q])
            q--
        }
        if p==q{
            fmt.Println(p, " ", pre, " ", suf, " ", ans)
        }
    }
    return ans
}
```

### 单调栈/栈模拟
#### 84. 柱状图中最大的矩形
sol:方法一：暴力解法（超时）
具体来说是：依次遍历柱形的高度，对于每一个高度分别向两边扩散，求出以当前高度为矩形的最大宽度多少
O(n^2)时间，O(1)空间
sol2:单调栈
```go
func largestRectangleArea(heights []int) int {
    stk:=make([]int, 0)
    n:=len(heights)
    ans:=0
    for i:=0; i<n; i++{
        for len(stk)>0&&heights[i]<heights[stk[len(stk)-1]]{
            end:=i
            cur:=stk[len(stk)-1]
            stk=stk[:len(stk)-1]
            st:=-1
            if len(stk)>0{
                st=stk[len(stk)-1]
            }else{
	            //为空的话左边界为-1
                st=-1
            }
            //宽度
            w:=(end-st-1)
            ans=max(ans, w*heights[cur])
        }
        stk=append(stk, i)
    }
    //后面栈中仍然可能存在元素, 可以推一个最小的进去求解
    for len(stk)>0{
	    //右边界为n
        end:=n
        cur:=stk[len(stk)-1]
        stk=stk[:len(stk)-1]
        st:=-1
        if len(stk)>0{
            st=stk[len(stk)-1]
        }else{
            st=-1
        }
        w:=end-st-1
        ans=max(ans, w*heights[cur])
    }
    return ans
}
```
#### 栈模拟
224. 基本计算器（困难）
227. 基本计算器II（中等）
https://leetcode.cn/problems/basic-calculator-ii/solutions/2099855/labuladong-ru-he-shi-xian-yi-ge-ji-suan-oh6jc/
```go
func calculate(s string) int {
    n:=len(s)
    index:=0
    var dfs func() int
    dfs = func() int{
        var presig byte ='+'
        stk:=make([]int, 0)
        num:=0
        for index<n{
            c:=s[index]
            index++
            if c>='0'&&c<='9'{
                num=num*10+int(c-'0')
            }
            if c=='('{
                num=dfs()
            }
            if (!(c>='0'&&c<='9')&&c!=' ')||index==n{
                if presig=='+'{
                    stk=append(stk, num)
                }else if presig=='-'{
                    stk=append(stk, -num)
                }else if presig=='*'{
                    top:=stk[len(stk)-1]
                    stk=stk[:len(stk)-1]
                    stk=append(stk, top*num)
                }else if presig=='/'{
                    top:=stk[len(stk)-1]
                    stk=stk[:len(stk)-1]
                    stk=append(stk, top/num)
                }
                num=0
                presig=byte(c)
            }
            if c==')'{
                break
            }
        }
        ans:=0
        for len(stk)>0{
            ans+=stk[len(stk)-1]
            stk=stk[:len(stk)-1]
        }
        return ans
    }
    return dfs()
}
```
394. 字符串解码
```go
func decodeString(s string) string {
    n:=len(s)
    num:=0
    ans:=""
    stk:=make([]string, 0)
    nstk:=make([]int, 0)
    for i:=0; i<n; i++{
        c:=s[i]
        if c>='0'&&c<='9'{
            num=num*10+int(s[i]-'0')
        }else if c=='['{
            nstk=append(nstk, num)
            stk=append(stk, string(c))
            num=0
        }else if c==']'{
            temp:=""
            //注意此刻要反转
            for len(stk)>0&&stk[len(stk)-1]!="["{
                temp+=reverse(stk[len(stk)-1])
                stk=stk[:len(stk)-1]
            }
            stk=stk[:len(stk)-1]
            tempnum:=nstk[len(nstk)-1]
            nstk=nstk[:len(nstk)-1]
            //再反转
            stk=append(stk, strings.Repeat(reverse(temp), tempnum))
        }else{
            stk=append(stk, string(c))
        }
    }
    for i:=0; i<len(stk); i++{
        ans+=stk[i]
    }
    return ans
}

func reverse(s string) string{
    t:=[]byte(s)
    n:=len(t)
    for i:=0; i<n/2; i++{
        t[i],t[n-1-i]=t[n-1-i],t[i]
    }
    return string(t)
}
```

### 二叉树
#### 前中后序遍历
迭代法:
https://www.bilibili.com/video/BV1RP4y1G79Z/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=108d23f95683578313bdaf5d938b5b3d
本质都是先遍历根, 具体的时机不一样
前序, 在向左之前do, 也就是将当前结点push前
cursor负责压节点进栈, 经常处于nil
```go
func preorderTraversal(root *TreeNode) []int {
    stk:=make([]*TreeNode, 0)
    ans:=make([]int, 0)
    cur:=root
    for cur!=nil||len(stk)>0{
        for cur!=nil{
            //do
            ans=append(ans, cur.Val)
            stk=append(stk, cur)
            cur=cur.Left
        }
        top:=stk[len(stk)-1]
        stk=stk[:len(stk)-1]
        cur=top.Right
    }
    return ans
}
```
中序,在向右之前do, 也就是将当前结点pop之后但是向右之前(和前序差不多)
```go
func inorderTraversal(root *TreeNode) []int {
    ans:=make([]int, 0)
    stk:=make([]*TreeNode, 0)
    cur:=root
    for cur!=nil||len(stk)>0{
        for cur!=nil{
            stk=append(stk, cur)
            cur=cur.Left
        }
        top:=stk[len(stk)-1]
        stk=stk[:len(stk)-1]
        //do
        ans=append(ans, top.Val)
        cur=top.Right
    }
    return ans
}
```
后序,向右之后do, 因此需要一个指针指向上次遍历过的节点
```go
func postorderTraversal(root *TreeNode) []int {
    ans:=make([]int, 0)
    stk:=make([]*TreeNode, 0)
    var pre *TreeNode
    cur:=root
    for cur!=nil||len(stk)>0{
        for cur!=nil{
            stk=append(stk, cur)
            cur=cur.Left
        }
        top:=stk[len(stk)-1]
        if top.Right!=nil&&top.Right!=pre{
            cur=top.Right
        }else{
            //do
            ans=append(ans, top.Val)
            pre=top
            stk=stk[:len(stk)-1]
        }
    }
    return ans
}
```
#### 124. 二叉树中的最大路径和
当前root的最大路径和取决于max(lsum, 0) +max(rsum, 0)+root.Val, 返回值是max(max(lsum, 0), max(rsum, 0))+root.Val
```go
func maxPathSum(root *TreeNode) int {
    ans:=math.MinInt/2
    var getSum func(root *TreeNode) int
    getSum = func(root *TreeNode) int{
        if root==nil{
            return 0
        }
        lsum:=getSum(root.Left)
        if lsum<0{
            lsum=0
        }
        rsum:=getSum(root.Right)
        if rsum<0{
            rsum=0
        }
        ans=max(ans, lsum+rsum+root.Val)
        return max(lsum, rsum)+root.Val
    }
    getSum(root)
    return ans
}
```
#### 105. 从前序与中序遍历序列构造二叉树
```go
func buildTree(preorder []int, inorder []int) *TreeNode {
    var build func(pl, pr, il, ir int) *TreeNode
    build = func(pl, pr, il, ir int) *TreeNode{
        if pl>=pr{
            return nil
        }
        val:=preorder[pl]
        root:=&TreeNode{
            Val:val,
        }
        if pl+1==pr{
            return root
        }
        iidx:=-1
        for i:=il; i<ir; i++{
            if inorder[i]==val{
                iidx=i
                break
            }
        }
        //通过确定il, idx, ir的数量关系来计算下一次的pl, pr
        root.Left=build(pl+1, pl+(iidx-il)+1, il, iidx)
        root.Right=build(pl+(iidx-il)+1, pr, iidx+1, ir)
        return root
    }
    return build(0, len(preorder), 0, len(inorder))
}
```
437. 路径总和 III
回溯+前缀和哈希
```go
func pathSum(root *TreeNode, targetSum int) int {
    sum:=0
    mp:=make(map[int]int)
    //cur-pre=0时，默认有一条路径
    mp[0]=1
    ans:=0
    var backtrack func(root *TreeNode)
    backtrack = func(root *TreeNode){
        if root==nil{
            return 
        }
        sum+=root.Val
        //此处更新答案
        ans+=mp[sum-targetSum]
        mp[sum]++
        backtrack(root.Left)
        backtrack(root.Right)
        mp[sum]--
        sum-=root.Val
    }
    backtrack(root)
    return ans
}
```
113. 路径总和 II
回溯
112. 路径总和
回溯
### 链表
#### 环形链表
141. 环形链表
快慢指针
142. 环形链表 II
证明:起始点到环起始点为x, 环起始点到交叉点为y, 交叉点到环起始点z
则(x+y)\*2=x+y+n(y+z)
则x=n(y+z)-y=(n-1)(y+z)+z, 当n=1时，x=z，当n>1时, x=(n-1)(y+z)+z, y+z是多余走的圈数
#### 138. 随机链表的复制
sol: 存一个哈希表，key是oldnode, val是newnode
#### 19. 删除链表的倒数第 N 个结点
sol1: 迭代：快慢指针
sol2:递归: 
```go
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    cnt:=0
    var backtrack func(head *ListNode) *ListNode
    backtrack = func(head *ListNode) *ListNode{
        if head==nil{
            return head
        }
        head.Next=backtrack(head.Next)
        cnt++
        if cnt==n{
            return head.Next
        }
        return head
    }
    return backtrack(head)
}
```
#### 83. 删除排序链表中的重复元素
```go
func deleteDuplicates(head *ListNode) *ListNode {
    dummy:=&ListNode{}
    dummy.Next=head
    pre:=dummy
    cur:=head
    for cur!=nil{
        for cur.Next!=nil&&cur.Val==cur.Next.Val{
            cur=cur.Next
        }
        if pre.Next==cur{
            pre=cur
        }else{
            pre=pre.Next
            pre.Next=cur.Next
            cur.Next=nil
            cur=pre
        }
        cur=cur.Next
    }
    return dummy.Next
}
```
#### 82. 删除排序链表中的重复元素 II
```go
func deleteDuplicates(head *ListNode) *ListNode {
    dummy:=&ListNode{}
    dummy.Next=head
    pre:=dummy
    cur:=head
    for cur!=nil{
        for cur.Next!=nil&&cur.Val==cur.Next.Val{
            cur=cur.Next
        }
        if pre.Next==cur{
            pre=cur
        }else{
            pre.Next=cur.Next
            cur.Next=nil
            cur=pre
        }
        cur=cur.Next
    }
    return dummy.Next
}
```
#### 反转链表系列
206. 反转链表
```go
func reverse(head *ListNode) *ListNode{
    if head==nil||head.Next==nil{
        return head
    }
    last:=reverse(head.Next)
    head.Next.Next=head
    head.Next=nil
    return last
}
```
进阶题: 反转链表前 N 个节点
sol: 后继怎么存是难点
```go
// 反转以 head 为起点的 n 个节点，返回新的头结点
func reverseN(head *ListNode, n int) *ListNode {
	if head==nil||head.Next==nil{
		return head
	}
    if n == 1 {
        // 记录第 n + 1 个节点
        successor = head.Next
        return head
    }
    // 以 head.Next 为起点，需要反转前 n - 1 个节点
    last := reverseN(head.Next, n-1)
    suc:=head.Next.Next
    head.Next.Next = head
    head.Next = suc
    return last
}
```
92. 反转链表 II, 反转left到right的节点
sol: 先实现反转前N个节点, 然后递归处理, 需要存后继
```go
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
### 技巧

#### 31. 下一个排列
https://leetcode.cn/problems/next-permutation/solutions/80560/xia-yi-ge-pai-lie-suan-fa-xiang-jie-si-lu-tui-dao-/?envType=study-plan-v2&envId=top-100-liked
我们还希望下一个数 增加的幅度尽可能的小，这样才满足“下一个排列与当前排列紧邻“的要求。为了满足这个要求，我们需要：
1.在 尽可能靠右的低位 进行交换，需要 从后向前 查找
2.将一个 尽可能小的「大数」 与前面的「小数」交换。比如 123465，下一个排列应该把 5 和 4 交换而不是把 6 和 4 交换
3.将「大数」换到前面后，需要将「大数」后面的所有数 重置为升序，升序排列就是最小的排列。以 123465 为例：首先按照上一步，交换 5 和 4，得到 123564；然后需要将 5 之后的数重置为升序，得到 123546。显然 123546 比 123564 更小，123546 就是 123465 的下一个排列
#### 179. 最大数
sol:先把所有int转成str，排序的时候比较拼接的字符串大小, 最后需要特判nums全为0的情况
#### 128. 最长连续序列
sol1:排序+dp
```go
func longestConsecutive(nums []int) int {
    n:=len(nums)
    if len(nums)<=0{
        return 0
    }
    sort.Slice(nums, func(i, j int) bool{
        return nums[i]<nums[j]
    })
    dp:=make([]int, n)
    dp[0]=1
    ans:=dp[0]
    for i:=1; i<n; i++{
        dp[i]=1
        if nums[i]==nums[i-1]+1{
            dp[i]=dp[i-1]+1
            ans=max(ans, dp[i])
        }else if nums[i]==nums[i-1]{
            dp[i]=dp[i-1]
        }
    }
    return ans
}
```
sol2:并查集
```go
type USet struct{
    count int //连通分量的个数
    par []int //记录父亲节点
    size []int //某个节点的儿子个数
}
func NewUSet(n int) USet{
    par:=make([]int, n)
    for i:=0; i<n; i++{
        par[i]=i
    }
    sz:=make([]int, n)
    for i:=0; i<n; i++{
        sz[i]=1
    }
    return USet{
        count:n,
        par:par,
        size:sz, 
    }
}
func (u *USet) find(x int) int{
    if u.par[x]!=x{
        u.par[x]=u.find(u.par[x])
    }
    return u.par[x]
}
func (u *USet) Union(x, y int){
    px:=u.find(x)
    py:=u.find(y)
    if px==py{
        return
    }
    u.par[px]=py
    u.size[py]+=u.size[px]
    u.count--
    return 
}
func (u *USet) GetMaxConSize() int{
    maxAns:=0
    for i:=0; i<len(u.par); i++{
        if i==u.par[i]{
            maxAns=max(maxAns, u.size[i])
        }
    }
    return maxAns
}
func longestConsecutive(nums []int) int {
    n:=len(nums)
    mp:=make(map[int]int)
    uset:=NewUSet(n)
    for i,x:=range nums{
        if _,ok:=mp[x];ok{
            continue
        }
        if _,ok:=mp[x-1];ok{
            uset.Union(i, mp[x-1])
        }
        if _,ok:=mp[x+1];ok{
            uset.Union(i, mp[x+1])
        }
        mp[x]=i
    }
    return uset.GetMaxConSize()
}
```
sol3:哈希
```go
func longestConsecutive(nums []int) int {
    set:=make(map[int]bool)
    for _,x:=range nums{
        set[x]=true
    }
    ans:=0
    for _,x:=range nums{
        if set[x-1]{
            continue
        }
        idx:=x
        for ;set[idx];idx++{}
        ans=max(ans, idx-x)
    }
    return ans
}
```
### 设计数据结构
#### LRU
自己实现双向链表
```go
type Node struct {
	pre  *Node
	next *Node
	key  int
	val  int
}

type List struct {
	head *Node
	tail *Node
}
type LRUCache struct {
	cap  int
	mp   map[int]*Node
	list *List
	num  int
}

func Constructor(capacity int) LRUCache {
	lru := LRUCache{
		cap: capacity,
		mp:  make(map[int]*Node),
		list: &List{
			head: &Node{
				pre:  nil,
				next: nil,
				val:  -1,
				key:  -1,
			},
			tail: &Node{
				pre:  nil,
				next: nil,
				key:  -1,
				val:  0,
			},
		},
	}
	lru.list.head.next = lru.list.tail
	lru.list.tail.pre = lru.list.head
	return lru
}

func (l *List) PushToHead(cur *Node) {
	cur.next = l.head.next
	cur.pre = l.head
	l.head.next.pre = cur
	l.head.next = cur
}

func (l *List) MoveToHead(cur *Node) {
	cur.pre.next = cur.next
	cur.next.pre = cur.pre
	//
	cur.next = l.head.next
	cur.pre = l.head
	l.head.next.pre = cur
	l.head.next = cur
}

func (l *List) RemoveLast() int {
	last := l.tail.pre
	last.pre.next = last.next
	last.next.pre = last.pre
	last.next = nil
	last.pre = nil
	return last.val
}

func (this *LRUCache) Get(key int) int {
	v, ok := this.mp[key]
	if !ok {
		return -1
	}
	this.list.MoveToHead(v)
	return v.val
}

func (this *LRUCache) Put(key int, value int) {
	v, ok := this.mp[key]
	if ok {
		this.mp[key].val = value
		this.list.MoveToHead(v)
		return
	}
	// 不存在
	node := &Node{
		pre:  nil,
		next: nil,
		key:  key,
		val:  value,
	}
	this.list.PushToHead(node)
	this.mp[key] = node
	this.num++
	if this.num > this.cap {
		lastkey := this.list.tail.pre.key
		this.list.RemoveLast()
		delete(this.mp, lastkey)
		this.num--
	}
}
```
go自带的list实现
```go
type entry struct{
    key int
    val int
}

type LRUCache struct {
    keyToNode map[int]*list.Element
    NodeList *list.List
    cap int
}


func Constructor(capacity int) LRUCache {
    return LRUCache{
        keyToNode:make(map[int]*list.Element),
        NodeList:list.New(),
        cap:capacity,
    }
}


func (l *LRUCache) Get(key int) int {
    v,ok:= l.keyToNode[key]
    if !ok{
        return -1
    }
    l.NodeList.MoveToFront(v)
    return v.Value.(entry).val
}


func (l *LRUCache) Put(key int, value int)  {
    v,ok:=l.keyToNode[key]
    if ok{
        v.Value=entry{
            key:key,
            val:value,
        }
        l.NodeList.MoveToFront(v)
        return 
    }
    node:=l.NodeList.PushFront(entry{
        key:key,
        val:value,
    })
    l.keyToNode[key]=node
    if len(l.keyToNode)>l.cap{
        back:=l.NodeList.Back()
        l.NodeList.Remove(back)
        bkey:=back.Value.(entry).key
        delete(l.keyToNode, bkey)
    }
    return 
}
```
#### LFU
https://leetcode.cn/problems/lfu-cache/solutions/2457716/tu-jie-yi-zhang-tu-miao-dong-lfupythonja-f56h/
```go
type entry struct{
    key int
    value int
    freq int
}
type LFUCache struct {
    //12:51
    keyToNode map[int]*list.Element
    freqToList map[int]*list.List
    minFreq int
    capacity int
}


func Constructor(capacity int) LFUCache {
    return LFUCache{
        keyToNode:make(map[int]*list.Element),
        freqToList:make(map[int]*list.List),
        minFreq:0,
        capacity:capacity,
    }
}

func (l *LFUCache) pushFrontFreq(e entry){
    _,ok:=l.freqToList[e.freq]
    if !ok{
        l.freqToList[e.freq]=list.New()
    }
    l.keyToNode[e.key] = l.freqToList[e.freq].PushFront(e)
}

func (l *LFUCache) Get(key int) int {
    node,ok:=l.keyToNode[key]
    if !ok{
        return -1
    }
    e:=node.Value.(entry)
    lst:=l.freqToList[e.freq]
    lst.Remove(node)
    if lst.Len()==0{
        delete(l.freqToList, e.freq)
        if e.freq==l.minFreq{
            l.minFreq++
        }
    }
    e.freq++
    l.pushFrontFreq(e)
    return e.value
}


func (l *LFUCache) Put(key int, value int)  {
    node,ok:=l.keyToNode[key]
    if ok{
        e:=node.Value.(entry)
        lst:=l.freqToList[e.freq]
        lst.Remove(node)
        if lst.Len()==0{
            delete(l.freqToList, e.freq)
            if e.freq==l.minFreq{
                l.minFreq++
            }
        }
        e.freq++
        e.value=value
        l.pushFrontFreq(e)
        return 
    }
    l.pushFrontFreq(entry{
        key:key,
        value:value,
        freq:1,
    })
    if len(l.keyToNode)>l.capacity{
        lst:=l.freqToList[l.minFreq]
        e:=lst.Back().Value.(entry)
        lst.Remove(lst.Back())
        delete(l.keyToNode, e.key)
        if lst.Len()==0{
            delete(l.freqToList, l.minFreq)
        }
    }
    l.minFreq=1
    return 
}
```
### 字符串
#### 151. 反转字符串中的单词
逆序双指针
```go
func reverseWords(s string) string {
    n:=len(s)
    p:=n-1
    ans:=""
    q:=p
    for p>=0{
        for p>=0&&s[p]==' '{
            p--
        }
        q=p
        for p>=0&&s[p]!=' '{
            p--
        }
        if p!=q{
            ans+=s[p+1:q+1]+" "
        }
    }
    return ans[0:len(ans)-1]
}
```

### 数学
#### 470. 用 Rand7() 实现 Rand10()
题解:
https://leetcode.cn/problems/implement-rand10-using-rand7/solutions/427572/cong-pao-ying-bi-kai-shi-xun-xu-jian-jin-ba-zhe-da/
rand7能得到 \[0,6], 两次得到\[0,48]
如果满足数位于\[1,10]则成功，否则重试
优化: 1,10-11,20-21,30-31,40都可以利用上，取模
```go
func randMToRandN() int{
    k:=0
    num:=1
    for num-1<N{
        k++
        num=num*M
    }
    nn=1
    for nn*N<num{
        nn=nn*N
    }
    for{
        x:=0
        for i:=0; i<k; i++{
            x=x*M+randM()-1
        }
        if x>=1&&x<=nn{
            return x%N+1
        }
    }
    return -1
}
```
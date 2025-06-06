### 动态规划
#### 字符串dp
1143.最长公共子序列
72.编辑距离
```
    for i:=1; i<=m; i++{
        for j:=1; j<=n; j++{
            dp[i][j]=min(dp[i-1][j], dp[i][j-1])+1
            if word1[i-1]==word2[j-1]{
                dp[i][j]=min(dp[i][j], dp[i-1][j-1])
            }else{
                dp[i][j]=min(dp[i][j], dp[i-1][j-1]+1)
            }
        }
    }

当前已经有dp[i-1][j]可以从i-1变成j, 那么就在i上删除1个字符就可以得到j
当前已经有dp[i][j-1]可以从i变成j-1, 那么就在i上增加1个字符就可以得到j
当前已经有dp[i-1][j-1]可以从i-1变成j-1, 那么可以替换1个字符
```
516.最长回文子序列
```go
func longestPalindromeSubseq(s string) int {
    n:=len(s)
    dp:=make([][]int, n)
    for i:=0; i<n; i++{
        dp[i]=make([]int, n)
    }
    for i:=0; i<n; i++{
        dp[i][i]=1
    }
    for i:=n-2; i>=0; i--{
        for j:=i+1; j<n; j++{
            if s[i]==s[j]{
                dp[i][j]=dp[i+1][j-1]+2
            }else{
                dp[i][j]=max(dp[i+1][j], dp[i][j-1])
            }
        }
    }
    return dp[0][n-1]
}
```
115. 不同的子序列
```go
func numDistinct(s string, t string) int {
    m:=len(t)
    n:=len(s)
    dp:=make([][]int, m+1)
    for i:=0; i<=m; i++{
        dp[i]=make([]int, n+1)
    }
    for i:=0; i<=n; i++{
        dp[0][i]=1
    }
    //dp表示t中的前i个字符出现在s前j个字符的个数
    for i:=1; i<=m; i++{
        for j:=1; j<=n; j++{
            if t[i-1]==s[j-1]{
                dp[i][j]=dp[i-1][j-1]+dp[i][j-1]
            }else{
                dp[i][j]=dp[i][j-1]
            }
        }
    }
    return dp[m][n]
}
```
32. 最长有效括号
```go
func longestValidParentheses(s string) int {
    n:=len(s)
    stk:=make([]int, 0)
    dp:=make([]int, n+1)
    ans:=0
    for i:=1; i<=n; i++{
        x:=s[i-1]
        if x=='('{
            stk=append(stk, i-1)
        }else{
            if len(stk)>0{
                t:=stk[len(stk)-1]
                stk=stk[:len(stk)-1]
                dp[i]=dp[t]+i-t
                ans=max(ans, dp[i])
            }
        }
    }
    return ans
}
```
#### 背包问题
#### 股票问题
https://leetcode.cn/circle/discuss/qiAgHn/
```
T[i][0][0] = 0, T[i][0][1] = -Infinity

T[i][k][0] = max(T[i - 1][k][0], T[i - 1][k][1] + prices[i])
T[i][k][1] = max(T[i - 1][k][1], T[i - 1][k - 1][0] - prices[i])
```

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
#### dp记录路径
JZ85 连续子数组的最大和(二), dp记录
```go
func FindGreatestSumOfSubArray( nums []int ) []int {
    // write code here
    n:=len(nums)
    dp:=make([]int, n)
    pre:=make([]int, n)
    ans:=make([]int, 0)
    dp[0]=nums[0]
    pre[0]=-1
    maxVal:=nums[0]
    maxIdx:=0
    for i:=1; i<n; i++{
        if dp[i-1]+nums[i]>=nums[i]{
            dp[i]=dp[i-1]+nums[i]
            pre[i]=i-1
        }else{
            dp[i]=nums[i]
            pre[i]=-1
        }
        if dp[i]>=maxVal{
            maxVal=dp[i]
            maxIdx=i
        }
    }
    for maxIdx!=-1{
        ans=append(ans, nums[maxIdx])
        maxIdx=pre[maxIdx]
    }
    for i:=0; i<len(ans)/2; i++{
        ans[i],ans[len(ans)-1-i]=ans[len(ans)-1-i],ans[i]
    }
    return ans
}
```
#### 剪绳子问题
https://leetcode.cn/problems/jian-sheng-zi-lcof/
```go
func cutRope( n int ) int {
    // write code here
    dp:=make([]int, n+1)
    dp[2]=1
    for i:=3; i<=n; i++{
        for j:=2; j<i; j++{
            temp:=max(j*dp[i-j], j*(i-j))
            dp[i]=max(dp[i], temp)
        }
    }
    return dp[n]
}
```
#### 卡特兰数
96. 不同的二叉搜索树
https://leetcode.cn/problems/unique-binary-search-trees/solutions/550154/96-bu-tong-de-er-cha-sou-suo-shu-dong-ta-vn6x/
```go
func numTrees(n int) int {
    if n==0{
        return 0
    }else if n==1{
        return 1
    }
    dp:=make([]int, n+1)
    dp[0]=1
    dp[1]=1
    for i:=2; i<=n; i++{
        for j:=1; j<=i; j++{
            dp[i]+=dp[j-1]*dp[i-j]
        }
    }
    return dp[n]
}
```

95. 不同的二叉搜索树 II
```go
func generateTrees(n int) []*TreeNode {
    var backtrack func(l, r int) []*TreeNode
    backtrack = func(l, r int) []*TreeNode{
        ans:=make([]*TreeNode, 0)
        if l>r{
            ans=append(ans, nil)
            return ans
        }
        for i:=l; i<=r; i++{
            lchlist:=backtrack(l, i-1)
            rchlist:=backtrack(i+1, r)
            for _,lc:=range lchlist{
                for _,rc:= range rchlist{
                    root:=&TreeNode{
                        Val:i,
                    }
                    root.Left=lc
                    root.Right=rc
                    ans=append(ans, root)
                }
            }
        }
        return ans
    }
    return backtrack(1, n)
}
```
### 二分
#### 搜索旋转数组系列
该系列比较难抽象成红蓝二分，这里给的抽象是直接染色, 注意l表示左边界, r表示右边界
抽象的方法是，l及左边的都不符合条件, r及右边的都不符合条件
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
        //注意此处是l+1<=mid, 因为这个时候是只有一个值的边界情况
        }else if nums[l+1]<=nums[mid]{
	        //注意此处是l+1<=target
            if nums[l+1]<=target&&target<nums[mid]{
                r=mid
            }else{
                l=mid
            }
        }else{
	        //注意此处是target<=nums[r-1]
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
        //注意此时可能满足只有一个值或者元素重复的情况，所以单独处理l++
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
	        //注意此时也需要更新最小值
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
三个指针
75. 颜色分类
```go
func sortColors(nums []int)  {
    n:=len(nums)
    zero:=0
    one:=0
    two:=n-1
    for one<=two{
        if nums[one]==1{
            one++
        }else if nums[one]==0{
            nums[one],nums[zero]=nums[zero],nums[one]
            one++
            zero++
        }else{
            nums[one],nums[two]=nums[two],nums[one]
            two--
        }
    }
    return 
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
#### 402. 移掉 K 位数字
```go
func removeKdigits(num string, k int) string {
    stk:=make([]byte, 0)
    n:=len(num)
    for i:=0; i<n; i++{
        for len(stk)>0&&num[i]<stk[len(stk)-1]&&k>0{
            stk=stk[:len(stk)-1]
            k--
        }
        if num[i]=='0'&&len(stk)==0{
            continue
        }
        stk=append(stk, num[i])

    }
    for k>0&&len(stk)!=0{
        stk=stk[:len(stk)-1]
        k--
    }
    if len(stk)==0{
        return "0"
    }
    return string(stk)
}
```
#### 316. 去除重复字母
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
#### 层序遍历
958. 二叉树的完全性检验
通过层次遍历过程中加入nil节点，并且判断pre\==nil and cur!=nil(这里的pre也包含了上一层的最后一个节点)时返回false, 

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

236. 二叉树的最近公共祖先
一共有五种情况，在纸上画一下
1.root为nil, 返回root
2.root就是p或者q, 那么root p/q就可能在root的子节点，此时root是最近祖先，否则返回给上层节点继续判断，总之这两种情况都是返回root
3.root的左边有答案，右边没有答案，则信赖左边返回的结果
4.root的左边无，右边有答案，信赖右边返回的结果
5.root的左右都有答案, p和q在root的两边, 返回root
#### 二叉树对称性
https://leetcode.cn/problems/shu-de-zi-jie-gou-lcof/solutions/791039/yi-pian-wen-zhang-dai-ni-chi-tou-dui-che-uhgs/
100. 相同的树
```go
func isSame(root1 *TreeNode, root2 *TreeNode) bool{
    if root1==nil&&root2==nil{
        return true
    }
    if root1==nil||root2==nil{
        return false
    }
    if root1.Val!=root2.Val{
        return false
    }
    return isSame(root1.Left, root2.Left)&&isSame(root1.Right, root2.Right)
}
```
572. 另一棵树的子树
```go
func isSubtree(root *TreeNode, subRoot *TreeNode) bool {
    if root==nil||subRoot==nil{
        return false
    }
    if isSame(root, subRoot){
        return true
    }
    return isSubtree(root.Left, subRoot)||isSubtree(root.Right, subRoot)
}
```
剑指 Offer 26. 树的子结构
注意子结构和子树的区别
```go
func isSameStruct(root1 *TreeNode, root2 *TreeNode) bool{
    if root1==nil&&root2==nil{
        return true
    }
    if root2==nil{
        return true
    }
    if root1==nil{
        return false
    }
    if root1.Val!=root2.Val{
        return false
    }
    return isSameStruct(root1.Left, root2.Left)&&isSameStruct(root1.Right, root2.Right)
} 
func isSubStructure(A *TreeNode, B *TreeNode) bool {
    if A==nil||B==nil{
        return false
    }
    if isSameStruct(A, B){
        return true
    }
    return isSubStructure(A.Left, B)||isSubStructure(A.Right, B)
}
```
#### JZ36 二叉搜索树与双向链表
https://leetcode.cn/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/solutions/896127/tu-wen-bing-mao-zui-tong-su-yi-dong-de-t-0adg/
改成双向循环链表问题也不大，pre指向最后一个元素
```go
func Convert( root *TreeNode ) *TreeNode {
    // write code here
    var pre *TreeNode
    var head *TreeNode
    var inorder func(root *TreeNode)
    inorder = func(root *TreeNode){
        if root==nil{
            return 
        }
        inorder(root.Left)
        if pre!=nil{
            pre.Right=root
        }else{
            head=root
        }
        root.Left=pre
        pre=root
        inorder(root.Right)
    }
    inorder(root)
    return head
}
```
JZ37 序列化二叉树与反序列化
```go
func Serialize( root *TreeNode ) string {
    // write code here
    if root==nil{
        return "#"
    }
    ans:=""
    que:=make([]*TreeNode, 0)
    que=append(que, root)
    ans+=strconv.Itoa(root.Val)
    for len(que)>0{
        sz:=len(que)
        for i:=0; i<sz; i++{
            top:=que[0]
            que=que[1:]
            if top.Left!=nil{
                ans+=","
                ans+=strconv.Itoa(top.Left.Val)
                que=append(que, top.Left)
            }else{
                ans+=",#"
            }
            if top.Right!=nil{
                ans+=","
                ans+=strconv.Itoa(top.Right.Val)
                que=append(que, top.Right)
            }else{
                ans+=",#"
            }
        }
    }
    return ans
}
func Deserialize( s string ) *TreeNode {
    // write code here
    if len(s)<=0||s[0]=='#'{
        return nil
    }
    strarr:=strings.Split(s, ",")
    idx:=0
    val,_:=strconv.Atoi(strarr[0])
    root:=&TreeNode{
        Val:val,
    }
    idx++
    que:=make([]*TreeNode, 0)
    que=append(que, root)
    for len(que)>0{
        sz:=len(que)
        for i:=0; i<sz; i++{
            top:=que[0]
            que=que[1:]
            if strarr[idx]=="#"{
                top.Left=nil
            }else{
                val,_:=strconv.Atoi(strarr[idx])
                top.Left=&TreeNode{
                    Val:val,
                }
                que=append(que, top.Left)
            }
            idx++
            if strarr[idx]=="#"{
                top.Right=nil
            }else{
                val,_:=strconv.Atoi(strarr[idx])
                top.Right=&TreeNode{
                    Val:val,
                }
                que=append(que, top.Right)
            }
            idx++
        }
    }
    return root
}
```
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
保留相同的最后一个元素
```go
func deleteDuplicates(head *ListNode) *ListNode {
    dummy:=&ListNode{}
    dummy.Next=head
    cur:=head
    pre:=dummy
    for cur!=nil{
        for cur.Next!=nil&&cur.Val==cur.Next.Val{
            cur=cur.Next
        }
        pre.Next=cur
        pre=cur
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
        return head
    }
    // 以 head.Next 为起点，需要反转前 n - 1 个节点
    last := reverseN(head.Next, n-1)
    suc:=head.Next.Next
    head.Next.Next = head
    head.Next = suc
    return last
}

func reverseN(head *ListNode, n int) *ListNode{
    if head==nil||head.Next==nil{
        return head
    }
    if n==1{
        return head
    }
    last:=reverseN(head.Next, n-1)
    head.Next.Next=head
    head.Next=nil
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
迭代法:
```go
func reverseBetween(head *ListNode, left int, right int) *ListNode {
    if head==nil||head.Next==nil{
        return head
    }
    dummy:=&ListNode{}
    dummy.Next=head
    l:=dummy
    for i:=0; i<left-1; i++{
        l=l.Next
    }
    pre:=l.Next
    cur:=pre.Next
    for i:=0; i<right-left; i++{
        nx:=cur.Next
        cur.Next=pre
        pre=cur
        cur=nx
    }
    l.Next.Next=cur
    l.Next=pre
    return dummy.Next
}
```
25. K 个一组翻转链表
sol: 先实现反转前N个节点, 然后递归反转k个节点, 不需要存后继
```go
func reverseN(head *ListNode, n int) *ListNode{
    if head==nil||head.Next==nil{
        return head
    }
    if n==1{
        return head
    }
    last:=reverseN(head.Next, n-1)
    head.Next.Next=head
    head.Next=nil
    return last
}

func reverseKGroup(head *ListNode, k int) *ListNode {
    if head==nil||head.Next==nil{
        return head
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
变题：改成后序不足k个节点也要反转
```go
func reverseKGroup(head *ListNode, k int) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	cur := head
	cnt := 0
	for i := 0; i < k; i++ {
		cnt++
		if cur == nil {
			return reverseN(head, cnt)
		}
		cur = cur.Next
	}
	last := reverseN(head, k)
	head.Next = reverseKGroup(cur, k)
	return last
}
```
#### 24. 两两交换链表中的节点
```go
func recur(head *ListNode) *ListNode{
    if head==nil||head.Next==nil{
        return head
    }
    last:=recur(head.Next.Next)
    nx:=head.Next
    nx.Next=head
    head.Next=last
    return nx
}
```
#### 328. 奇偶链表
将所有索引为奇数的节点和索引为偶数的节点分别组合在一起
#### 排序奇升偶降链表
https://www.nowcoder.com/practice/3a188e9c06ce4844b031713b82784a2a?tpId=117&tqId=39396&rp=1&ru=/exam/oj&qru=/exam/oj&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26pageSize%3D50%26search%3D207%26tab%3D%25E7%25AE%2597%25E6%25B3%2595%25E7%25AF%2587%26topicId%3D117&difficulty=undefined&judgeStatus=undefined
先奇偶链表, 再反转链表, 再合并两个升序链表
#### 143. 重排链表
先快慢指针找到中点，然后反转链表, 再合并链表
#### 148. 排序链表
快慢指针，然后归并合并
```go
func mergeSortList(head *ListNode) *ListNode{
    if head==nil||head.Next==nil{
        return head
    }
    fast:=head
    slow:=head
    pre:=head
    for fast!=nil&&fast.Next!=nil{
        fast=fast.Next.Next
        pre=slow
        slow=slow.Next
    }
    pre.Next=nil
    a:=mergeSortList(head)
    b:=mergeSortList(slow)
    return mergeList(a, b)
}

func mergeList(head1, head2 *ListNode) *ListNode{
    dummy:=&ListNode{}
    cur:=dummy
    p:=head1
    q:=head2
    for p!=nil||q!=nil{
        if p!=nil&&q!=nil{
            if p.Val<q.Val{
                cur.Next=&ListNode{
                    Val:p.Val,
                }
                p=p.Next
            }else{
                cur.Next=&ListNode{
                    Val:q.Val,
                }
                q=q.Next
            }
        }else if p!=nil{
            cur.Next=&ListNode{
                Val:p.Val,
            }
            p=p.Next
        }else if q!=nil{
            cur.Next=&ListNode{
                Val:q.Val,
            }
            q=q.Next
        }
        cur=cur.Next
    }
    return dummy.Next
}
```
排序链表快排实现
https://leetcode.cn/problems/sort-list/solutions/2400774/ge-chong-pai-xu-suan-fa-jie-jue-mou-pao-9dwmt/
```go

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
### 模拟
498. 对角线遍历
https://leetcode.cn/problems/diagonal-traverse/solutions/1497406/by-lin-shen-shi-jian-lu-k-laf5/
```go
func findDiagonalOrder(mat [][]int) []int {
    n:=len(mat)
    m:=len(mat[0])
    ans:=make([]int, 0)
    for i:=0; i<m+n-1; i++{
        if i&1==0{
            st:=min(i, n-1)
            end:=max(0, i-(m-1))
            for x:=st; x>=end; x--{
                ans=append(ans, mat[x][i-x])
            }
        }else{
            st:=max(0, i-(m-1))
            end:=min(i, n-1)
            for x:=st; x<=end; x++{
                ans=append(ans, mat[x][i-x])
            }
        }   
    }
    return ans
}
```
54.螺旋矩阵
59.螺旋矩阵 II
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
### 贪心
55. 跳跃游戏
45. 跳跃游戏 II
```go
func jump(nums []int) int {
    n:=len(nums)
    if n<=1{
        return 0
    }
    maxIdx:=0
    idx:=0
    ans:=0
    for{
        //如果能跳到终点, 直接跳, 退出计算答案
        maxIdx=idx+nums[idx]
        if maxIdx>=n-1{
            ans++
            break
        }
        // 否则不能跳到终点, 则在剩下可以跳的位置中选能跳最远的
        nextIdx:=-1
        tempMaxIdx:=-1
        for i:=idx+1; i<=maxIdx; i++{
            if i+nums[i]>=tempMaxIdx{
                tempMaxIdx=i+nums[i]
                nextIdx=i
            }
        }
        ans++
        idx=nextIdx
    }
    return ans
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
##### 8.字符串转换整数 (atoi)
只允许用int32不用int64位的解法
核心思想: 在到32位的最大值或者最小值/10时, 预期看看会不会超出
```go
func myAtoi(s string) int {
    var ans int32
    n:=len(s)
    sign:=1
    idx:=0
    // 无效前导空格
    for idx<n&&s[idx] == ' '{
        idx++
    }
    if idx>=n{
        return 0
    }
    // 判断正负
    if s[idx] == '-'{
        idx++
        sign = -1
    }else if s[idx] == '+'{
        idx++
    }
    // 去除前导0
    for idx<n&&s[idx]=='0'{
        idx++
    }
    for idx<n&&(s[idx]>='0'&&s[idx]<='9'){
        ans = ans*10+int32(s[idx]-'0')*int32(sign)
        if sign==1&&ans<0{
            return math.MaxInt32
        }else if sign==-1&&ans>0{
            return math.MinInt32
        }
        idx++
    }
    return int(ans)
}
```

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
#### 43. 字符串相乘
```go
func multiply(num1 string, num2 string) string {
    //15:54
    if num1=="0"||num2=="0"{
        return "0"
    }
    m:=len(num1)
    n:=len(num2)
    ans:=make([]int, m+n)
    for i:=m-1; i>=0; i--{
        for j:=n-1; j>=0; j--{
            x:=int(num1[i]-'0')
            y:=int(num2[j]-'0')
            sum:=ans[i+j+1]+x*y
            carry:=sum/10
            ans[i+j+1]=sum%10
            ans[i+j]+=carry
        }
    }
    idx:=0
    for ; idx<len(ans)&&ans[idx]==0;idx++{}
    res:=""
    for ;idx<len(ans); idx++{
        res+=strconv.Itoa(ans[idx])
    }
    return res
}
```
#### 字符数组实现2的n次方
```go
func getTwoPow(n int) string {
	ans := make([]int, 1000)
	ans[len(ans)-1] = 1
	m := len(ans)
	for i := 0; i < n; i++ {
		carry := 0
		for j := m - 1; j >= 0; j-- {
			sum := ans[j] * 2
			ans[j] = sum%10 + carry
			carry = sum / 10
		}
		fmt.Println(ans)
	}
	idx := 0
	for ; idx < m && ans[idx] == 0; idx++ {
	}
	res := ""
	for ; idx < m; idx++ {
		res += strconv.Itoa(ans[idx])
	}
	return res
}
```
#### 大数相加
#### 大数相减
```go
func substring(s string, t string) string {
	// write code here
    if len(s)<len(t){
        return "-"+substring(t, s)
    }else if len(s)==len(t){
        if s<t{
            return "-"+substring(t, s)
        }else if s==t{
            return "0"
        }
    }
	ans := make([]byte, len(s))
	p := len(s) - 1
	q := len(t) - 1
	borrow := 0
    idx:=len(ans)-1
	for p >= 0 || q >= 0 {
		if p >= 0 && q >= 0 {
			temp := int(s[p]-'0') - int(t[q]-'0') - borrow
			ans[idx]= byte((temp+10)%10+'0')
			if temp < 0 {
				borrow = 1
			} else {
				borrow = 0
			}
			p--
			q--
		} else if p >= 0 {
            temp:=int(s[p]-'0')-borrow
            ans[idx]=byte((temp + 10) % 10+'0')
			if temp < 0 {
				borrow = 1
			} else {
				borrow = 0
			}
            p--
		} else {
            temp:=int(t[q]-'0')-borrow
            ans[idx]=byte((temp + 10) % 10+'0')
			if temp < 0 {
				borrow = 1
			} else {
				borrow = 0
			}
            q--
		}
        idx--
	}
    idx++
    for ;idx<len(ans); idx++{
        if ans[idx]!='0'{
            break
        }
    }
    return string(ans[idx:])
}
```
#### 大数相乘
#### 7. 整数反转
```go
func reverse(x int) int {
    var ans int32
    copy:=x
    for x!=0{
        if ans<math.MinInt32/10||ans>math.MaxInt32/10{
            return 0
        }
        ans=ans*10+int32(x%10)
        x=x/10
    }
    if copy>0&&ans<0{
        return 0
    }
    if copy<0&&ans>0{
        return 0
    }
    return int(ans)
}
```
#### 384.打乱数组
洗牌算法
https://leetcode.cn/problems/shuffle-an-array/solutions/1114959/pythonjavajavascriptgo-xi-pai-suan-fa-by-k7i2/
```go
type Solution struct {
    nums []int
}
func Constructor(nums []int) Solution {
    s := Solution{nums}
    return s
}
func (this *Solution) Reset() []int {
    return this.nums
}
func (this *Solution) Shuffle() []int {
    temp := make([]int, len(this.nums))
    copy(temp, this.nums)
    for i := 0; i < len(temp); i++ {
        idx := rand.Intn(len(temp) - i) + i
        temp[i], temp[idx] = temp[idx], temp[i]
    }
    return temp
}
```
### 图
拓扑排序
207.课程表
210.课程表 II

### 堆
手写堆
```go
//小根堆
type Heap struct{
    nums []int
}
func down(nums []int, idx, n int){
    child:=2*idx+1
    for child<n{
        if child+1<n&&nums[child+1]<nums[child]{
            child++
        }
        if nums[idx]<=nums[child]{
            break
        }
        nums[idx],nums[child]=nums[child],nums[idx]
        idx=child
        child=child*2+1
    }
    return 
}
func up(nums []int, idx int){
    par:=(idx-1)>>1
    for par>=0{
        if nums[par]<=nums[idx]{
            break
        }
        nums[idx],nums[par]=nums[par],nums[idx]
        idx=par
        par=(par-1)>>1
    }
    return 
}
func (h *Heap) Push(x int){
    h.nums=append(h.nums, x)
    up(h.nums, len(h.nums)-1)
}
func (h *Heap) Len() int{
    return len(h.nums)
}
func (h *Heap) Pop() int{
    ans:=h.nums[0]
    n:=len(h.nums)
    h.nums[0],h.nums[n-1]=h.nums[n-1],h.nums[0]
    down(h.nums, 0, n-1)
    h.nums=h.nums[:n-1]
    return ans
}
func (h *Heap) Peek() int{
    return h.nums[0]
}
```
堆go实现接口模版
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
	//注意此处是t[len(t)-1], 因为go底层是先swap, 再调用这个Pop方法
	x:=t[len(t)-1]
	*h=t[:len(t)-1]
	return x
}
```
### 多线程
启动3个线程, 每个线程只打印1个字符A/B/C, 循环10次
```go
func main() {
	c1 := make(chan struct{}, 0)
	c2 := make(chan struct{}, 0)
	c3 := make(chan struct{}, 0)
	cnt := 10
	wg := &sync.WaitGroup{}
	wg.Add(3)
	go func() {
		defer wg.Done()
		for i := 0; i < cnt; i++ {
			<-c1
			fmt.Println("A")
			c2 <- struct{}{}
		}
		close(c1)
	}()
	go func() {
		defer wg.Done()
		for i := 0; i < cnt; i++ {
			<-c2
			fmt.Println("B")
			c3 <- struct{}{}
		}
		close(c2)
	}()
	go func() {
		defer wg.Done()
		for i := 0; i < cnt; i++ {
			<-c3
			fmt.Println("C")
			if i < cnt-1 {
				c1 <- struct{}{}
			}
		}
		close(c3)
	}()
	c1 <- struct{}{}
	wg.Wait()
	fmt.Println("Done")
}

```


个人的思考，锁和信号量能做的，channel都能做，但是channel的性能会差一些
n个协程打印一个数组
```go
func main() {
	n := 10
	nums := make([]int, 0)
	for i := 225; i >= 0; i-- {
		nums = append(nums, i)
	}
	wg := sync.WaitGroup{}
	wg.Add(n)
	chanList := make([]chan struct{}, n)
	for i := 0; i < n; i++ {
		chanList[i] = make(chan struct{}, 0)
	}
	arridx := 0
	//最先开始执行任务的goroutine序号
	firstG := 3
	fmt.Println("start Go")
	for i := 0; i < n; i++ {
		go func(idx int) {
			defer wg.Done()
			for {
				<-chanList[idx]
				if arridx < len(nums) {
					fmt.Println("goroutine", idx, nums[arridx])
					arridx += 1
					chanList[(idx+1)%n] <- struct{}{}
				} else {
					chanList[(idx+1)%n] <- struct{}{}
					fmt.Println("goroutine", idx, "finish")
					break
				}
			}
			//如果是最后一个执行任务的下个协程, 还需要帮助最后一个执行任务的协程退出
			if idx == (len(nums)+firstG)%n {
				<-chanList[idx]
			}
		}(i)
	}
	//最先开始执行任务
	chanList[firstG] <- struct{}{}
	wg.Wait()
	fmt.Println("finish")
}
```
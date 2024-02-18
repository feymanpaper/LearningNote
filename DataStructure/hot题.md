### 动态规划
#### 字符串dp
1143. 最长公共子序列
72. 编辑距离

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

### 单调栈
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

### 技巧
#### 31. 下一个排列
https://leetcode.cn/problems/next-permutation/solutions/80560/xia-yi-ge-pai-lie-suan-fa-xiang-jie-si-lu-tui-dao-/?envType=study-plan-v2&envId=top-100-liked
我们还希望下一个数 增加的幅度尽可能的小，这样才满足“下一个排列与当前排列紧邻“的要求。为了满足这个要求，我们需要：
1.在 尽可能靠右的低位 进行交换，需要 从后向前 查找
2.将一个 尽可能小的「大数」 与前面的「小数」交换。比如 123465，下一个排列应该把 5 和 4 交换而不是把 6 和 4 交换
3.将「大数」换到前面后，需要将「大数」后面的所有数 重置为升序，升序排列就是最小的排列。以 123465 为例：首先按照上一步，交换 5 和 4，得到 123564；然后需要将 5 之后的数重置为升序，得到 123546。显然 123546 比 123564 更小，123546 就是 123465 的下一个排列

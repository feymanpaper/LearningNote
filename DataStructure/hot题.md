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
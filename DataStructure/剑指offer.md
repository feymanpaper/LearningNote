### 数组
#### 原地交换
JZ-数组中重复的数字
```go
func duplicate( numbers []int ) int {
    n:=len(numbers)
    for i:=0; i<n; i++{
        for numbers[i]!=numbers[numbers[i]]{
            if numbers[i]==numbers[numbers[i]]{
                return numbers[i]
            }
            numbers[i],numbers[numbers[i]]=numbers[numbers[i]],numbers[i]
        }
        if i!=numbers[i]{
            return numbers[i]
        }
    }
    return -1
}
```
442. 数组中重复的数据
448. 找到所有数组中消失的数字
41. 缺失的第一个正数
```go
func firstMissingPositive(nums []int) int {
    n:=len(nums)
    for i:=0; i<n; i++{
        for nums[i]>=1&&nums[i]<=n&&nums[i]!=nums[nums[i]-1]{
            nums[i],nums[nums[i]-1]=nums[nums[i]-1],nums[i]
        }
    }
    for i:=0; i<n; i++{
        if i+1!=nums[i]{
            return i+1
        }
    }
    return n+1
}
```

#### 位运算
#### JZ15 二进制中1的个数

#### 二叉树
236. 二叉树的最近公共祖先
235. 二叉搜索树的最近公共祖先

#### 数学
sol1:迭代O(n)
sol2:递归保留结果, O(logn)
```go
func Power( base float64 ,  exponent int ) float64 {
    if exponent==0{
        return 1
    }
    if exponent<0{
        base=1/base
        exponent=-exponent
    }
    //偶数
    if exponent&1==0{
        ans:=Power(base, exponent/2)
        return ans*ans
    }
    //奇数
    ans:=Power(base, exponent/2)
    return ans*ans*base
}
```
JZ65 不用加减乘除做加法
https://leetcode.cn/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/solutions/174464/jin-zhi-tao-wa-ru-he-yong-wei-yun-suan-wan-cheng-j/
```go
func encryptionCalculate(dataA int, dataB int) int {
    for dataB!=0{
        add:=dataA^dataB
        carry:=(dataA&dataB)<<1
        dataA=add
        dataB=carry
    }
    return dataA
}
```

#### 双指针
264. 丑数 II
```go
func nthUglyNumber(n int) int {
    nums:=make([]int, n+1)
    nums[1]=1
    a,b,c:=1,1,1
    for i:=2; i<=n; i++{
        temp:=min(2*nums[a], min(3*nums[b], 5*nums[c]))
        if temp==2*nums[a]{
            a++
        }
        if temp==3*nums[b]{
            b++
        }
        if temp==5*nums[c]{
            c++
        }
        nums[i]=temp
    }
    return nums[n]
}
```
313. 超级丑数
拓展到一般规律
```go
func nthSuperUglyNumber(n int, primes []int) int {
    nums:=make([]int, n+1)
    nums[1]=1
    index:=make([]int, len(primes))
    for i:=0; i<len(index); i++{
        index[i]=1
    }
    for i:=2; i<=n; i++{
        temp:=math.MaxInt
        for j:=0; j<len(primes); j++{
            temp=min(temp, nums[index[j]]*primes[j])
        }
        for j:=0; j<len(primes); j++{
            if nums[index[j]]*primes[j]==temp{
                index[j]++
            }
        }
        nums[i]=temp
    }
    return nums[n]
}
```
JZ21 调整数组顺序使奇数位于偶数前面(二)
快排思想

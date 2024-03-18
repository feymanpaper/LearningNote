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

#### 排序
JZ51 数组中的逆序对
sol:归并排序
```go
func reversePairs(record []int) int {
    ans:=mergeSort(record, 0, len(record)-1)
    fmt.Println(record)
    return ans
}

func mergeSort(nums []int, l, r int) int{
    if l>=r{
        return 0
    }
    mid:=l+(r-l)>>1
    lsum:=mergeSort(nums, l, mid)
    rsum:=mergeSort(nums, mid+1, r)
    midsum:=merge(nums, l, mid, mid+1, r)
    return lsum+midsum+rsum
}

func merge(nums []int, l1, r1, l2, r2 int) int{
    n:=r1-l1+1+r2-l2+1
    p:=l1
    q:=l2
    temp:=make([]int, n)
    idx:=0
    ans:=0
    for p<=r1||q<=r2{
        if p<=r1&&q<=r2{
            if nums[p]<=nums[q]{
                temp[idx]=nums[p]
                p++
            }else{
                ans+=r1-p+1
                temp[idx]=nums[q]
                q++
            }
        }else if p<=r1{
            temp[idx]=nums[p]
            p++
        }else{
            temp[idx]=nums[q]
            q++
        }
        idx++
    }
    for i:=l1; i<=r2; i++{
        nums[i]=temp[i-l1]
    }
    return ans
}
```
315. 计算右侧小于当前元素的个数
此题重点, 可以求左边/右边 小于/大于当前元素的个数
逆序对归并排序类似
https://leetcode.cn/problems/count-of-smaller-numbers-after-self/solutions/1256119/zhen-xiao-bai-jiang-jie-gei-ni-bai-kai-l-78qf/
```go
var index []int
var ans []int
func mergeSort(nums []int, l, r int){
    if l>=r{
        return 
    }
    mid:=l+(r-l)>>1
    mergeSort(nums, l, mid)
    mergeSort(nums, mid+1, r)
    merge(nums, l, mid, mid+1, r)
}
func merge(nums []int, l1, r1, l2, r2 int){
    n:=r2-l2+1+r1-l1+1
    p:=l1
    q:=l2
    idx:=0
    temp:=make([]int, n)
    tempidx:=make([]int, n)
    for p<=r1||q<=r2{
        if p<=r1&&q<=r2{
            if nums[p]>nums[q]{
                ans[index[p]]+=r2-q+1
                temp[idx]=nums[p]
                tempidx[idx]=index[p]
                p++          
            }else{
                temp[idx]=nums[q]
                tempidx[idx]=index[q]
                q++
            }
        }else if p<=r1{
            temp[idx]=nums[p]
            tempidx[idx]=index[p]
            p++
        }else{
            temp[idx]=nums[q]
            tempidx[idx]=index[q]
            q++
        }
        idx++
    }
    for i:=l1; i<=r2; i++{
        nums[i]=temp[i-l1]
        index[i]=tempidx[i-l1]
    }
    return 
}
func countSmaller(nums []int) []int {
    n:=len(nums)
    ans=make([]int, n)
    index=make([]int, n)
    for i:=0; i<n; i++{
        index[i]=i
    }
    mergeSort(nums, 0, len(nums)-1)
    return ans
}
```

JZ61 扑克牌顺子
```go
func checkDynasty(places []int) bool {
    sort.Slice(places, func(i, j int) bool{
        return places[i]<places[j]
    })
    king:=0
    for i,x:=range places{
        if x==0{
            king++
        }else if i>0&&places[i]==places[i-1]{
            return false
        }
    }
    if places[len(places)-1]-places[king]<5{
        return true
    }
    return false
}
```
#### 位运算
JZ15 二进制中1的个数
JZ56 数组中只出现一次的两个数字
https://leetcode.cn/problems/single-number-iii/solutions/2484352/tu-jie-yi-zhang-tu-miao-dong-zhuan-huan-np9d2/
```go
func singleNumber(nums []int) []int {
    two:=0
    for _,x:=range nums{
        two^=x
    }
    ans:=make([]int, 2)
    lowb:=two&(-two)
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
JZ43 整数中1出现的次数（从1到n整数中1出现的次数）
https://leetcode.cn/problems/number-of-digit-one/solutions/1748815/by-baoya_uncle-2hnj/
JZ44 数字序列中某一位的数字
https://leetcode.cn/problems/nth-digit/solutions/1129550/pythonjavajavascriptgo-jian-dan-mo-ni-by-kk3x/
```go
func findNthDigit(n int) int {
    dig:=1
    num:=9
    for n>dig*num{
        n-=dig*num
        dig++
        num*=10
        if math.MaxInt/num<dig{
            break
        }
    }
    first:=1
    for i:=0; i<dig-1; i++{
        first=first*10
    }
    n--
    idx:=n/dig
    subidx:=n%dig
    loc:=first+idx
    str:=strconv.Itoa(loc)
    return int(str[subidx]-'0')
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

剑指 Offer 62. 圆圈中最后剩下的数字
```go
func iceBreakingGame(n, m int) int {
    nums:=make([]int, n)
    for i:=0; i<n; i++{
        nums[i]=i
    }
    idx:=0
    for len(nums)>1{
        idx=(idx+m-1)%len(nums)
        nums=append(nums[:idx], nums[idx+1:]...)
    }
    return nums[0]
}
```

#### 字符串模拟
JZ20 表示数值的字符串
https://leetcode.cn/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/solutions/1461026/by-gong-zhu-zi-zhu-3xvr/
```go
func isNumeric( str string ) bool {
    // write code here
    s:=strings.Trim(str, " ")
    if len(s)==0{
        return false
    }
    numFlag:=false
    dotFlag:=false
    eFlag:=false
    for i:=0; i<len(s); i++{
        if s[i]>='0'&&s[i]<='9'{
            numFlag=true
        }else if s[i]=='.'&&!dotFlag&&!eFlag{
            dotFlag=true
        }else if (s[i]=='e'||s[i]=='E')&&(!eFlag&&numFlag){
            eFlag=true
            numFlag=false
        }else if (s[i]=='+'||s[i]=='-')&&(i==0||s[i-1]=='e'||s[i-1]=='E'){

        }else{
            return false
        }
    }
    return numFlag
}
```
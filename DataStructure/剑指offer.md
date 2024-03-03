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
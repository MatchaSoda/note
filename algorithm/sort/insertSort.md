# 插入排序 insertSort
### 效率
* 平均时间复杂度：O(N^2)
* 最差时间复杂度：O(N^2)
* 空间复杂度：O(1)
* 排序方式：In-place
* 稳定性：稳定
### 逻辑
将数组分为 有序区 和 无序区 。  
遍历无序区中的元素，将其从尾到头的和有序区的元素比较，并插入到合适的位置。
![](https://www.runoob.com/wp-content/uploads/2019/03/insertionSort.gif)
### 代码
```cpp
vector<int> sortArray(vector<int>& nums) {
  for (int i = 1; i < nums.size(); i++) {
    int tmp = nums[i];
    int j;
    for (j = i - 1; j >= 0 && nums[j] > tmp; j--) {
      nums[j + 1] = nums[j];
    }
    nums[j + 1] = tmp;
  }
  return nums;
}
```
### 优化
二分优化，将一个元素和有序区的元素依次比较时，可以使用二分法。总时间复杂度降至O(nlogn)。
```cpp
int binarySearch(vector<int>& nums, int target, int left, int right) {
  while (left <= right) {
    int mid = (right - left) / 2 + left;
    if (target < nums[mid]) {
      right = mid - 1;
    } else if (target > nums[mid]) {
      left = mid + 1;
    } else {
      return mid;
    }
  }
  return left;
}
vector<int> sortArray(vector<int>& nums) {
  for (int i = 1; i < nums.size(); i++) {
    int tmp = nums[i];
    int pos = binarySearch(nums, tmp, 0, i - 1);
    for (int j = i - 1; j >= pos; j--) {
      nums[j + 1] = nums[j];
    }
    nums[pos] = tmp;
  }
  return nums;
}
```
golang版本
```go
func binarySearch(nums []int, target int) int {
	left := 0
	right := len(nums) - 1
	for left <= right {
		mid := (right-left)/2 + left
		if nums[mid] > target {
			right = mid - 1
		} else if nums[mid] < target {
			left = mid + 1
		} else {
			return mid
		}
	}
	return left
}

func sortArray(nums []int) []int {
	for index := 1; index < len(nums); index++ {
		currentValue := nums[index]
		position := binarySearch(nums[0:index-1], currentValue)
		for pendingIndex := index - 1; pendingIndex >= position; pendingIndex-- {
			nums[pendingIndex+1] = nums[pendingIndex]
		}
		nums[position] = currentValue
	}
	return nums
}
```
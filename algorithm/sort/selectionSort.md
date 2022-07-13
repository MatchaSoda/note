# 选择排序 selectionSort
### 性能
* 平均时间复杂度：O(n2)
* 最佳时间复杂度：O(n2)
* 最差时间复杂度：O(n2)
* 空间复杂度：O(1)
* 排序方式：In-place
* 稳定性：不稳定
### 逻辑
将数组分为 有序区 和 无序区 。  
遍历无序区找到最小元素，放在有序区末尾（即和无序区第一个元素交换）。直到所有元素均排序完毕。

![](https://www.runoob.com/wp-content/uploads/2019/03/selectionSort.gif)
### 代码
```cpp
vector<int> sortArray(vector<int>& nums) {
  for (int i = 0; i < nums.size() - 1; i++) {
    int min = i;
    for (int j = i + 1; j < nums.size(); j++) {
      if (nums[min] > nums[j]) {
        min = j;
      }
    }
    swap(nums[i], nums[min]);
  }
  return nums;
}
```
golang版本
```go
func sortArray(nums []int) []int {
	for pendingIndex, _ := range nums[:len(nums)-1] {
		minIndex := pendingIndex
		for currentIndex, currentValue := range nums[pendingIndex+1:] {
			if nums[minIndex] > currentValue {
				minIndex = currentIndex + pendingIndex + 1
			}
		}
		nums[pendingIndex], nums[minIndex] = nums[minIndex], nums[pendingIndex]
	}
	return nums
}
```
### 优化
* 二元选择排序：  
普通的选择排序每次循环只能确定一个元素排序后的位置，我们可以同时记录最大和最小值，放到两边，从而减少循环次数。
* 堆排序：  
堆排序是树形选择排序，具体见堆排序篇。
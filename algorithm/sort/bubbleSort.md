# 冒泡排序 bubbleSort
### 性能
* 平均时间复杂度：O(n2)
* 最佳时间复杂度：O(n)
* 最差时间复杂度：O(n2)
* 空间复杂度：O(1)
* 排序方式：In-place
* 稳定性：稳定
### 逻辑
将数组分为 无序区 和 有序区 。  
从无序区通过交换找出最大值放到有序区前端。
![](https://www.runoob.com/wp-content/uploads/2019/03/bubbleSort.gif)
### 代码
第一层循环，每轮取到当前的最大元素，放到右侧。即每轮确定一个元素的位置，共需要 ``n - 1`` 轮。  
第二层循环，循环前边的无序区，需要减去后边的有序区，有序区元素个数也是第一轮的循环数，即 ``i`` 。
```cpp
vector<int> bubbleSort(vector<int>& nums) {
  int n = nums.size();
  for (int i = 0; i < n - 1; i++) {
    for (int j = 0; j < n - 1 - i; j++) {
      if (nums[j] > nums[j + 1]) {
        swap(nums[j], nums[j + 1]);
      }
    }
  }
  return nums;
}
```
### 优化 
对于数组仅前几个元素无序的情况，上边的算法会遍历固定的 ``n - 1`` 次，实际上我们需要遍历左侧的无序区将元素移动到右边有序区。  
所以我们要记录遍历时最后一次交换的位置，这个位置之后的数据已经有序了，这右边就是有序区。下次遍历到这个位置即可。
```cpp
vector<int> sortArray(vector<int>& nums) {
  int flag = nums.size();
  while (flag > 0) {
    int k = flag;
    flag = 0;
    for (int j = 1; j < k; j++) {
      if (nums[j] < nums[j - 1]) {
        swap(nums[j], nums[j - 1]);
        flag = j;
      }
    }
  }
  return nums;
}
```
golang版本
```go
func sortArray(nums []int) []int {
	lastSwapIndex := len(nums)
	for lastSwapIndex > 0 {
		currentSwapIndex := 0
		for index := 1; index < lastSwapIndex; index++ {
			if nums[index] < nums[index-1] {
				nums[index], nums[index-1] = nums[index-1], nums[index]
				currentSwapIndex = index
			}
		}
		lastSwapIndex = currentSwapIndex
	}
	return nums
}
```
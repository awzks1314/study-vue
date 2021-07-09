#

# 二分查找

二分查找是一种基于比较值和数组中间元素的教科书式算法。

- 如果目标值等于中间元素，则找到目标值
- 如果目标值较小，继续在左侧搜索
- 如果目标值较大，继续在右侧搜索

二分法查找的时间复杂度O(logn)。

### 例题

给定一个n个元素有序的(升序)整形数组`nums`和一个目标值`target`，写一个函数搜索`nums`中的`target`，
如果目标值存在返回下标，否则返回`-1`

**示例1**

	输入: nums = [-1,0,3,5,9,12], target = 9
	输出: 4
	解释: 9 出现在 nums 中并且下标为 4
	
**示例2**

	输入: nums = [-1,0,3,5,9,12], target = 2
	输出: -1
	解释: 2 不存在 nums 中因此返回 -1

**提示：**

	1.你可以假设 nums 中的所有元素是不重复的。
	2.n 将在 [1, 10000]之间。
	3.nums 的每个元素都将在 [-9999, 9999]之间。
	
### 解题思路

```js
var search = function(nums, target) {
	//设置左右点边界点
	let left = 0, right = nums.length - 1;
	while(left <= right){
		//设置中间值,防止数值过大溢出
		let mid = Math.floor((left + right)/2)
		if(nums[mid] === target) return mid
		//else if(nums[mid] < target) left = mid + 1
		//else right = mid - 1
		left = nums[mid] < target ? mid + 1 : left;
		right = nums[mid] < target ? right : mid - 1;
	}
	return -1
}
```

### 例题

你是产品经理，目前正在带领一个团队开发新的产品。不幸的是，你的产品最新版本没有通过质量检测。
由于每个版本都是基于之前的版本开发的，所以错误版本之后的所有版本都是错的。

假设你有n个版本`[1,2,...,n]`,你想找出导致之后所有版本出错的第一个错误版本。

你可以通过调用 `bool isBadVersion(version) `接口来判断版本号 version 是否在单元测试中出错。实现一个函数来查找第一个错误的版本。你应该尽量减少对调用 API 的次数。

**示例 1：**

	输入：n = 5, bad = 4
	输出：4
	解释：
	调用 isBadVersion(3) -> false 
	调用 isBadVersion(5) -> true 
	调用 isBadVersion(4) -> true
	所以，4 是第一个错误的版本。
	
**示例 2：**

	输入：n = 1, bad = 1
	输出：1

### 解决思路

```js
var solution = function(isBadVersion) {
	return function(n) {
		let left = 0, right = n - 1;
		while(left <= right) {
			//设置中间值
			let mid = Math.floor((left + right)/2)
			if(isBadVersion(mid)) {
				//如果中间值值错误版本，那个mid前肯定是错误版本
				right = mid - 1
			}else{
				left = mid + 1
			}
		}
		return left;
	}
}
```

### 例题

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。
如果目标值不存在与数组中，返回它将会被按顺序插入的位置。

假设数组中无重复元素。

示例 1:

	输入: [1,3,5,6], 5
	输出: 2
示例 2:

	输入: [1,3,5,6], 2
	输出: 1
示例 3:

	输入: [1,3,5,6], 7
	输出: 4
示例 4:

	输入: [1,3,5,6], 0
	输出: 0

### 解题思路

```js
var searchInsert = function(nums, target) {
	let left = 0, right = nums.length - 1;
	//特殊条件判断
	if(nums[right] < target) {
		return right + 1;
	}
	if(nums[0] > target) {
		return 0;
	}
	while(left <= right) {
		//设中间值
		let mid = Math.floor((right + left) /2);
		if(nums[mid] === target) return mid;
		else if (nums[mid] < target) left = mid + 1;
		else right = mid - 1;
	}
	return left;
}
```



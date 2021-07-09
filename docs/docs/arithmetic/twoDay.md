#

# 双指针

### 例题

给你一个按`非递减顺序`排序的整数数组`nums`，返回`每个数字的平方`
组成的新数组，要求也按`非递减顺序`排序。

**示例 1：**

	输入：nums = [-4,-1,0,3,10]
	输出：[0,1,9,16,100]
	解释：平方后，数组变为 [16,1,0,9,100]
	排序后，数组变为 [0,1,9,16,100]
	
**示例 2：**

	输入：nums = [-7,-3,2,3,11]
	输出：[4,9,9,49,121]

### 解题思路

##### 分析

新建空数组，用双指针法，左右指针进行比较，那个元素的绝对值最大，就从头插入
空数组，若左指针绝对值大于右指针绝对值，则左指针插曲空数组，左指针右移一步；
若右指针绝对值大于左指针绝对值，则右指针插入数组，右指针左移一步。

```js
var sortedSquares = function(nums) {
	let left = 0, right = nums.length - 1,arr = [];
	while(left <= right) {
		if(Math.abs(nums[left]) > Math.abs(nums[right])) {
			//左指针大于右指针
			arr.unshift(nums[left] * nums[left])
			left ++
		}else {
			//右指针大于左指针
			arr.unshift(nums[right] * nums[right])
			right--
		}
	}
	return arr;
}
```

### 例题

给定一个数组，将数组中的元素向右移动`k`个位置，其中`k`是非负数。

**示例 1:**

	输入: nums = [1,2,3,4,5,6,7], k = 3
	输出: [5,6,7,1,2,3,4]
	解释:
	向右旋转 1 步: [7,1,2,3,4,5,6]
	向右旋转 2 步: [6,7,1,2,3,4,5]
	向右旋转 3 步: [5,6,7,1,2,3,4]
**示例 2:**

	输入：nums = [-1,-100,3,99], k = 2
	输出：[3,99,-1,-100]
	解释: 
	向右旋转 1 步: [99,-1,-100,3]
	向右旋转 2 步: [3,99,-1,-100]


### 解题思路

##### 第一种分析

新建数组保存，遍历数组，每个元素放置`(index + k) mod nums.length`的位置。

如nums = [1,2,3,4,5,6,7], k = 3,arr = []。将元素`1`放置arr[(0+3)%7]处
元素`5`在arr[(4+3)%7]即arr[0]

```js
var rotate = function(nums, k) {
	let arr = [],len = nums.length;
	for(let i = 0 ; i< len ; i++) {
		arr((i + k ) % len) = nums[i]
	}
	
	for(let j = 0; j<  arr.length ; j++) {
		nums[j] = arr[j]
	}
}
```

##### 第二种

先整体翻转，翻转为[7,6,5,4,3,2,1],再翻转[5,6,7,4,3,2,1]，
再翻转[5,6,7,1,2,3,4]

```js
//定义翻转函数

var reserve = function(nums,start,end) {
	while(start < end) {
		let template = nums[start]
		nums[start] = nums[end]
		nums[end] = template
		start ++
		end --
	}
}
var rotate = function(nums, k) {
	//先求余数，放置k大于nums,length
	k %= nums.length
	
	//翻转全部
	reserve(nums,0,nums.length - 1)
	//翻转左侧,数量k - 1，表示下标
	reserve(nums,0,k - 1)
	//最后翻转右侧
	reserve(nums,k,nums.length - 1)
}
```

##### 第三种

利用splice，剪切之后等到剪切的元素，在插入

```js
var rotate =  function(nums, k) {
	nums.splice(0,0,...nums.splice(-(k %= nums.length),k))
}
```



### 例题

给定一个数组`nums`，编写一个函数将所有`0`移动到数组的末尾，同时保持非零元素的相对顺序。

示例:

	输入: [0,1,0,3,12]
	输出: [1,3,12,0,0]

### 解题思路

##### 双指针+冒泡排序

设置双指针，left，right，初始值为0。right快指针，left慢指针。当右指针探测到非0元素时，
与左指针交换，同时左右指针都加一；若探测到0时，则右指针加1

```js
var moveZeroes = function(nums) {
	let left = 0, right = 0 ,len = nums.length - 1;
	while(right <= len) {
		if(nums[right]){
			let tem = nums[left]
			nums[left] = nums[right]
			nums[right] = tem
			left++
		}
		right++
	}
}
```

### 例题

给定一个已按照**升序排列**的整数数组`numbers`，请你从数组中找出两个数满足相加之和等于目标数`target`。

函数应该以长度为`2`的整数数组的形式返回这两个数的下标值。`numbers`的下标**从1开始计数**。

你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素。

**示例 1：**

	输入：numbers = [2,7,11,15], target = 9
	输出：[1,2]
	解释：2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。
**示例 2：**

	输入：numbers = [2,3,4], target = 6
	输出：[1,3]
**示例 3：**

	输入：numbers = [-1,0], target = -1
	输出：[1,2]	


 ### 解题思路

二分指针，设置左右指针，若左右指针值相加等于目标值，则找到唯一解；
若小于目标值，则左指针加一；若大于目标值，则右指针减一；

```js
var twoSum = function(numbers, target) {
	let left = 0, len = numbers.length - 1;
	while(left <= len){
		let sum = numbers[left] + numbers[len]
		if(sum === target) return [left, len]
		else if(sum > target) len --
		else left++
	}
}
```
 

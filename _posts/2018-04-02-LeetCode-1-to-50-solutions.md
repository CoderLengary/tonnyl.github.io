---
layout: post # needs to be post
title: LeetCode 1 - 50 解法 (Go, Java, JavaScript, Kotlin, Python, Swift)
summary: Let's LeetCode!
featured-img: design
categories: [LeetCode]
---

在文章正式开始之前, 首先有几点需要特别说明:

<!--more-->

+ 本文所有代码都托管在 **Windary: https://github.com/TonnyL/Windary**, 如果你发现了错误, 请通过 issue 或其他方式指出, 但是暂时 **不** 接受 Pull Request;
+ 我提供的算法仅供参考, 并不代表最佳的解法(有的解法可能爆机, 有的解法可能刚好踩线 AC);
+ 文章使用 `Go` 作为主要描述语言, 同时也会在每一道题的末尾附加其他语言描述的解法的地址及其 AC 情况(选用 Go 是因为她的代码量最少);
+ 所有的解法都有相应单元测试用例:
	- Go: 测试用例文件与解法同级位于同级目录下, 例如解法文件 `TwoSum.go` 的路径为 `Windary/Golang/TwoSum/TwoSum.go`, 则测试用例文件 `TwoSum_test.go` 的路径为 `Windary/Golang/TwoSum/TwoSum_test.go`, 所有的测试用例文件均以 **解法文件名 + _test** 的形式命名;
	- Java: 测试用例与解法文件均位于 `src` 目录下, 例如解法文件 `TwoSum.java` 的路径为 `Windary/Java/src/TwoSum.java`, 则测试用例文件 `TwoSumTest.java` 的路径为 `Windary/Java/src/TwoSumTest.java`, 所有的测试文件均以 **解法文件名 + Test** 的形式命名;
	- JavaScript: 测试用例与解法文件为同一个文件, 解法文件在前, 测试用例在后;
	- Kotlin: 与 Java 代码组织形式一致;
	- Python: 测试用例文件与解法文件均位于 `Python` 目录下, 例如解法文件 `TwoSum.py` 的路径为 `Windary/Python/TwoSumTest.py`, 则测试用例文件的路径为 `Windary/Python/TwoSumTest.py`, 所有的测试文件均以 **解法文件名 + Test** 的形式命名;
	- Swift: 所有的解法文件位于 `Windary/Swift/LeetCode/LeetCode/` 目录下, 所有的测试用例文件位于 `Windary/Swift/LeetCode/LeetCodeTests/` 目录下, 测试用例文件以 **解法文件名 + Tests** 的形式命名.


## [1. Two Sum](https://leetcode.com/problems/two-sum/description/)
Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

Example:

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

------

## [1. 两数之和](https://leetcode-cn.com/problems/two-sum/description/)
给定一个整数数列，找出其中和为特定值的那两个数。

你可以假设每个输入都只会有一种答案，同样的元素不能被重用。

示例:

```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

思路:

循环遍历, 这种方法时最容易想到的. 时间复杂度为 O(n2).

解法(Go):

```go
func twoSum(nums []int, target int) []int {
	
	numsLength := len(nums)
	for i := 0; i < numsLength; i++ {
		for j := i + 1; j < numsLength; j++ {
			if target == (nums[i] + nums[j]) {
				return []int{i, j}
			}
		}
	}
	
	return []int{0, 0}
}
```

其他解法:

上面的解法还可以改进, 可以先使用快速排序(时间复杂度 O(nlogn)), 然后双指针从前后往中间扫描. 但是题目要求是返回数组下标, 所以还需要额外的空间存放下标信息.

另外一种方法则是通过 Hash 的方式. 从前往后扫描一遍, 将数及下标存放到 Map 中, 再次扫描即可. 时间复杂度为 O(n).

AC 情况:

| # | Title  | Go | Java | JavaScript | Kotlin | Python | Swift |  Difficulty |
| :-------------: | :-------------: | :-------------: | :-------------: | :-------------: | :-------------: | :-------------: | :-------------: | :-------------:|
| 1 | [Two Sum](https://leetcode.com/problems/two-sum/) | [√](https://github.com/TonnyL/Windary/tree/master/Golang/TwoSum/TwoSum.go) | [√](https://github.com/TonnyL/Windary/tree/master/Java/src/TwoSum.java) | [√](https://github.com/TonnyL/Windary/tree/master/JavaScript/src/TwoSum.js) | [√](https://github.com/TonnyL/Windary/tree/master/Kotlin/src/TwoSum.kt) | [√](https://github.com/TonnyL/Windary/tree/master/Python/TwoSum.py)  | [√](https://github.com/TonnyL/Windary/tree/master/Swift/LeetCode/LeetCode/TwoSum.swift) | Easy |
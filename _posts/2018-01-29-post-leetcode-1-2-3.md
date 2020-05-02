---
layout: post
title: "Leetcode-1-2-3"
date: 2018-01-29 07:49:08
image: 'http://res.cloudinary.com/degzyaemb/image/upload/c_scale,w_750/v1517212452/sheldon1.png'
description: Leetcode1、2、3解法与心得
category: 'leetcode'
tags:
- leetcode
- algorithm
- data structure
twitter_text:
introduction: 这章将介绍Leetcode1、2、3刷题的解法和心得。让你更加深入理解map、set和list的用法。
---

## LeetCode 1 Two Sum
**Problem** : Given an array of integers, return indices of the two numbers such that they add up to a specific target. You may assume that each input would have exactly one solution, and you may not use the same element twice.

**Example** :
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].


**Solution** : 通过描述可以看出题目将给出两个参数（分别为一个数组和一个目标数），需要在数组中找出满足相加等于目标数的一对值。

刚接触oj的小白看到这道题也会觉得很简单，通过两个for循环判断相加是否等于target即可，但是这道题有更好的解法，就是通过一个 `hashmap` 保存数组的值和下标即可通过一次遍历来实现，即 `HashMap<Integer(value), Integer(key)>`。

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        // 用于保存数组中的值和下标
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++){
            // 如果找到就可以直接返回两个数组下标
            if (map.containsKey(target - nums[i]))
                return new int[]{map.get(target - nums[i]), i};
            map.put(nums[i], i);
        }
        return new int[]{};
    }
}
```


## LeetCode 2 Add Two Numbers

**Problem** : You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their
nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Example** :

*Input*: (2 -> 4 -> 3) + (5 -> 6 -> 4)

*Output*: 7 -> 0 -> 8

*Explanation*: 342 + 465 = 807.

**Solution** : 之前在Lintcode上遇到这道题，当时的思路是将传入的两个List转化为数字，再通过简单的相加后转换为List。但是写完发现无法通过，因为还存在 `(1) + (9 -> 9 -> 9 -> 9)`
的这种两个List不平衡的情况，后来看了答案才发现是直接对两个List相同位置的值进行相加(遇到大坑才能印象深刻啊)。

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        int i = 0;
        ListNode result = new ListNode(0);
        ListNode l = result;
        while(l1 != null && l2 != null) {
            int tmp = l1.val + l2.val;
            if (i == 1) {
                tmp += i;
                i = 0;
            }
            i = tmp / 10;

            l.next = new ListNode(tmp % 10);
            l1 = l1.next;
            l2 = l2.next;
            l = l.next;
        }
        while(l1 != null) {
            int tmp = l1.val;
            if(i != 0) {
                tmp += 1;
                i = 0;
            }
            i = tmp / 10;

            l.next = new ListNode(tmp % 10);
            l1 = l1.next;
            l = l.next;
        }

        while(l2 != null) {
            int tmp = l2.val;
            if(i != 0) {
                tmp += 1;
                i = 0;
            }
            i = tmp / 10;
            l.next = new ListNode(tmp % 10);
            l2 = l2.next;
            l = l.next;
        }

        if (i != 0)
            l.next = new ListNode(1);

        return result.next;
    }
}
```


## LeetCode 3 Longest Substring Without Repeating Characters

**Problem** : Given a string, find the length of the longest substring without repeating characters.

**Example** :

Given `"abcabcbb"`, the answer is `"abc"`, which the length is 3.

Given `"bbbbb"`, the answer is `"b"`, with the length of 1.

Given `"pwwkew"`, the answer is `"wke"`, with the length of 3. Note that the answer must be a substring, `"pwke"` is a subsequence and not a substring.

**Solution** : 这道题的解法其实就是初始化 `int max`，遍历字符串的每个下标，将不重复的值插入到 `HashSet<Character>` 中，当遇到不同值的时候修改`max`的值即可。

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        Set<Character> set = new HashSet<>();
        int max = 0;
        for (int i = 0; i < s.length(); i++) {
            int j = i;
            // 如果遇到set中不重复的，就写入set中
            while(j < s.length() && !set.contains(s.charAt(j))) {
                set.add(s.charAt(j));
                j++;
            }
            // 将max修改成max和set.size()中最大的那个
            max = Math.max(set.size(), max);
            // 将set清空
            set.clear();
        }
        return max;
    }
}
```


-----
其实数据结构真的是个好东西，它将比较繁琐的题目解法简化成容易理解便于操作的形式。

更重要的是： *面试的时候要考啊！不好好学真是不行!*
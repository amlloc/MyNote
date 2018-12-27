---
title: LeetCode
date: 2018-12-10 21:13:45
categories: codenote
tags: Algorithm
---

LeetCode进阶之路
<!--more-->

## 初级题目

### 数组问题

#### 从排序数组中删除重复项

```c++
    int removeDuplicates(vector<int>& nums) {
        if(nums.size() == 0)
            return 0;
        int k = 1;
        for(int i = 1; i < nums.size(); i++){
            if (nums[i] == nums[i - 1])
                continue;
            nums[k++] = nums[i];
        }
        
        return k;
    }
```

#### 买卖股票的最佳时机

- 解读

  典型的贪心算法，当明天比今天低价的时候将股票卖出即可。

- 代码

  ```c++
  int maxProfit(vector<int>& prices) {
          
          int result = 0;
          for(int i = 0; i < prices.size() - 1; i++){
              if( prices[i] < prices[i + 1]){
                  result = result + (prices[i + 1] - prices[i]);
              }
          }
          return result;
      }
  ```

#### 旋转数组

给定一个数组，将数组中的元素向右移k位，其中k是非负数

- 说明
  - 尽可能想出更多的解决方案，至少有三种不同的方法可以解决这个问题。
  - 要求使用空间复杂度为 O(1) 的原地算法。

- 解读
  [参考这篇博客](https://blog.csdn.net/biezhihua/article/details/79535021)

  使用`i = (i + k) % n`的思路，每次调换元素后，后一个元素调换都是基于上一个位置。如：

  ```
  1,2,3,4,5,6,7,8  n = 8, k = 2
  ```

  则执行流程如下：

  ```
  //奇数位旋转完毕
  1
  1,2,3,4,5,6,7,8
  
      3 
  1,2,1,4,5,6,7,8
  
          5    
  1,2,1,4,3,6,7,8
  
              7
  1,2,1,4,3,6,5,8
  
  
  7,2,1,4,3,6,5,8
  
  //偶数位旋转开始
    2
  7,2,1,4,3,6,5,8
  
        4
  7,2,1,2,3,6,5,8
        
            6
  7,2,1,2,3,4,5,8
  
                8
  7,2,1,2,3,4,5,6
  
  7,8,1,2,3,6,5,6
  
  ```

- 解题如下：

  ```c++
  class Solution
  {
    public:
      void rotate(vector<int> &nums, int k)
      {
          if (nums.size() == 0 || (k %= nums.size()) == 0)
          {
              return;
          }
          int length = nums.size();
          int start = 0;
          int i = 0;
          int cur = nums[i];
          int cnt = 0;
  gengxin
          while (cnt++ < length)
          {
              i = (i + k) % length;
              int t = nums[i];    //t用于记录转换前的数字
              nums[i] = cur;      //旋转
              if (i == start)  //用于处理奇偶位旋转问题
              {
                  ++start;     //start更新
                  ++i;         //index更新
                  cur = nums[i];  //因为更新了奇偶位，用于转换的数据也需要更新
              }
              else
              {
                  cur = t;        //将上一次被转换的位置的数据用于转换下次数据
              }
          }
      }
  };
  ```

#### 存在重复

- 给定一个整数数组，判断是否存在重复元素。

  如果任何值在数组中出现至少两次，函数返回 true。如果数组中每个元素都不相同，则返回 false。

- **示例 1:**

  ```
  输入: [1,2,3,1]
  输出: true
  ```

- 解读

  - 可以利用set的特性来解决这个问题
  - 可先排序，然后再判断是否有连续index具备相同的元素

- 解题如下：

  ```C++
  #include <set>
  class Solution {
  public:
      bool containsDuplicate(vector<int>& nums) {
          
          bool flag = false;
          set<int> set_int;
          
          for(int i = 0; i < nums.size(); i++){
              set_int.insert(nums[i]);
          }
          
          if(set_int.size() != nums.size()){
              flag = true;
          }
          
          return flag;
          
      }
  };
  ```

  #### 只出现一次的数字

  给定一个**非空**整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

- **说明：**

  你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

- **示例 1:**

  ```
  输入: [2,2,1]
  输出: 1
  ```

- **解读**

  两个相同的数字进行异或后得到0

- **解题如下**

  ```C++
  class Solution {
  public:
      int singleNumber(vector<int>& nums) {
          
          int tmp = 0;
          for(int i = 0; i < nums.size(); i++){
              
              tmp = tmp ^ nums[i];
              
          }
          
          return tmp;
          
          
      }
  };
  ```

  #### 两个数组的交集 II

  给定两个数组，编写一个函数来计算它们的交集。

- **示例 1:**

  ```
  输入: nums1 = [1,2,2,1], nums2 = [2,2]
  输出: [2,2]
  ```

- **说明：**

  - 输出结果中每个元素出现的次数，应与元素在两个数组中出现的次数一致。
  - 我们可以不考虑输出结果的顺序。

- **进阶:**

  - ￼
    搜索
    20:03

    ￼
    一家子￼
    未来	如果给定的数组已经排好序呢？你将如何优化你的算法？

  - 如果 *nums1* 的大小比 *nums2* 小很多，哪种方法更优？

  - 如果 *nums2* 的元素存储在磁盘上，磁盘内存是有限的，并且你不能一次加载所有的元素到内存中，你该怎么办？

- **解读**

  可以使用map的特性，值对具备相同性质的元素进行筛选，利用`value`记录元素出现的次数

- **解题如下**

  ```C++
  #include <map>
  
  class Solution {
  public:
      vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
          
          map<int, int> num2count1;
          vector<int> result;
          
          for(int i = 0;i < nums1.size(); i++){
              
              if(num2count1.find(nums1[i]) != num2count1.end()){
                  num2count1[nums1[i]]++;//每出现一次+1
              }else{
                  num2count1[nums1[i]] = 1;//初次出现则初始化value
              }
          }
          
          for(int i = 0; i < nums2.size(); i++){
              
              if(num2count1.find(nums2[i]) != num2count1.end()){
                  
                  result.push_back(nums2[i]);
                  num2count1[nums2[i]]--;//将一个元素放入result
                  
                  if(num2count1[nums2[i]] == 0) num2count1.erase(num2count1.find(nums2[i]));//如果元素被放置完，则删除该元素
              }
          }
          
          return result;
      }
  };
  ```

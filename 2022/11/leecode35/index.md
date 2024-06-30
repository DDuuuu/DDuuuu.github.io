# LeeCode-35-数组-搜索插入位置


# 题目 

[题目链接](https://leetcode.cn/problems/search-insert-position/)

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 O(log &lt;sup&gt;n&lt;/sup&gt;) 的算法。

示例 1:

```ini
输入: nums = [1,3,5,6], target = 5
输出: 2
```

示例 2:

```ini
输入: nums = [1,3,5,6], target = 2
输出: 1
```

示例 3:

```ini
输入: nums = [1,3,5,6], target = 7
输出: 4
```


提示:

&#43; 1 &lt;= nums.length &lt;= 10&lt;sup&gt;4&lt;/sup&gt;
&#43; -10&lt;sup&gt;4&lt;/sup&gt; &lt;= nums[i] &lt;= 10&lt;sup&gt;4&lt;/sup&gt;
&#43; nums 为 无重复元素 的 升序 排列数组
&#43; -10&lt;sup&gt;4&lt;/sup&gt; &lt;= target &lt;= 10&lt;sup&gt;4&lt;/sup&gt;

# 解题

## 二分

思考，排序数组，无重复，升序，规定时间复杂度，好像只有二分了，递归是否可以，因为递归也是Log&lt;sup&gt;n&lt;/sup&gt;



```C#
public int Search(int[] nums, int target)
{
    //[left,right]
    var left = 0;
    var right = nums.Length - 1;
    var mid = 0;
    while (left &lt;= right)
    {
        mid = (left &#43; right) &gt;&gt; 1;
        if (nums[mid] == target)
        {
            left = mid;
            break;
        }
        else if (target&lt;nums[mid])
        {
            right = mid - 1;
        }
        else
        {
            left = mid &#43; 1;
        }
    }
    return left;
}

public int SearchPlus(int[] nums, int target)
{
    //[right,left)
    var left = 0;
    var right = nums.Length;
    var mid = 0;
    while (left != right)
    {
        mid = (left &#43; right) &gt;&gt; 1;
        if (nums[mid] == target)
        {
            left = mid;
            break;
        }
        else if (target &lt; nums[mid])
        {
            right = mid;
        }
        else
        {
            left = mid &#43; 1;
        }
    }
    return left;
}
```

## 递归

```C#
public int SearchPro(int[] nums, int target)
{
    //[right,left]
    var left = 0;
    var right = nums.Length - 1;
    var mid = 0;

    int InnerCore(int[] nums, int target, int left, int right)
    {
        if (left &lt;= right)
        {
            mid = (left &#43; right) &gt;&gt; 1;
            if (nums[mid] == target)
            {
                return mid;
            }
            else if (target &lt; nums[mid])
            {
                return InnerCore(nums, target, left, mid-1);
            }
            else
            {
                return InnerCore(nums, target, mid &#43; 1, right);
            }
        }
        return left;
    }
    return InnerCore(nums,target,left,right);
}

public int SearchProPlus(int[] nums, int target)
{
    //[right,left)
    var left = 0;
    var right = nums.Length;
    var mid = 0;

    int InnerCore(int[] nums, int target, int left, int right)
    {
        if (left != right)
        {
            mid = (left &#43; right) &gt;&gt; 1;
            if (nums[mid] == target)
            {
                return mid;
            }
            else if (target &lt; nums[mid])
            {
                return InnerCore(nums, target, left, mid);
            }
            else
            {
                return InnerCore(nums, target, mid &#43; 1, right);
            }
        }
        return left;
    }

    return InnerCore(nums, target, left, right);
}
```

# 网络解析

```C#
public int SearchInsert(int[] nums, int target) {
    int searchIdx;
    int searchLower = 0, searchHigher = nums.Length - 1;
    int middle = 0;
    while (searchLower &lt;= searchHigher)
    {
        middle = searchLower &#43; (searchHigher - searchLower) / 2;
        if (nums[middle] == target)
        {
            return middle;
        }
        if (nums[middle] &gt; target)
        {
            searchHigher = middle - 1;
        }
        else
        {
            searchLower = middle &#43; 1;
        }
    }

    if (target &lt;= nums[middle]) return middle;
    if (target &gt; nums[middle]) return middle &#43; 1;
    return -1;
}
```

```C#
public class Solution {
    public int SearchInsert(int[] nums, int target) {
        
        return Rank(nums,0,nums.Length - 1,target);
    }

    public int Rank(int[] arr,int left,int right,int target)
    {        
        if(target &lt; arr[left]) return left;
        if(target &gt; arr[right]) return right &#43; 1;
        
        int mid = (left &#43; right)/2;
        if(target == arr[mid]) return mid;

        if(target&lt;arr[mid]) return Rank(arr,left,mid-1,target);
        else return Rank(arr,mid&#43;1,right,target);

    }
}
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/11/leecode35/  


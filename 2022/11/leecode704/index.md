# LeeCode-704-数组-二分查找


# 题目

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。


示例 1:

```ini
输入: nums = [-1,0,3,5,9,12], target = 9
输出: 4
解释: 9 出现在 nums 中并且下标为 4
```

示例 2:

```ini
输入: nums = [-1,0,3,5,9,12], target = 2
输出: -1
解释: 2 不存在 nums 中因此返回 -1
```

提示:

1. 你可以假设 `nums` 中的所有元素是不重复的。
2. `n` 将在 `[1, 10000]`之间。
3. `nums` 的每个元素都将在 `[-9999, 9999]`之间。

# 解题

如果使用遍历，最好情况O(1)，最坏情况O(n)。

使用二分法是O(logn)。

尝试使用二分，二分肯定有上下限和中间值，通过中间值和目标值比较，不断缩小上下限最后得到结果。

&#43; 从中考虑While的判断是什么？

&#43; 上下限的范围怎么划定？

初版：

```C#
public int Search(int[] nums, int target)
{
    var maxindex = nums.Length-1;
    var minindex = 0;
    var index = (maxindex &#43; minindex) / 2;
    while (maxindex != minindex)
    {
        if (target == nums[index])
        {
            break;
        }
        if (target &lt; nums[index])
        {
            maxindex = index;
        }
        else
        {
            minindex = index&#43;1;
        }
        index = (maxindex &#43; minindex) / 2;
    }
    if (target == nums[index]) return index;
    return -1;
}
```

测试用例通过了，但是有效率不高，有很多重复代码，改进一下

```C#
public int SearchPro(int[] nums, int target)
{
    var maxindex = nums.Length - 1;
    var minindex = 0;
    var midindex=0;
    while (minindex&lt;=maxindex)
    {
        midindex = (maxindex &#43; minindex) / 2;
        if (target == nums[midindex])
        {
            return midindex;
        }
        if (target &lt; nums[midindex])
        {
            maxindex = midindex - 1;
        }
        else
        {
            minindex = midindex &#43; 1;
        }
    }
    return -1;
}
```

最佳答案

```C#
public int StandardSearch(int[] nums, int target)
{
    int left = 0, right = nums.Length - 1;
    while (left &lt;= right)
    {
        int mid = left &#43; (right - left) / 2;
        if (nums[mid] == target)
        {
            return mid;
        }
        if (nums[mid] &gt; target)
        {
            right = mid - 1;
        }
        else
        {
            left = mid &#43; 1;
        }
    }
    return -1;
}
```

对比：

想一下变量用left和right应该更简介一点。mid在循环体中声明，每次会在栈中创建新的临时变量。

# 网络解析

target 是在一个在左闭右闭的区间里，**也就是[left, right]** ，这种就是上面的标准写法

- while (left &lt;= right) 要使用 &lt;= ，因为left == right是有意义的，所以使用 &lt;=
- if (nums[middle] &gt; target) right 要赋值为 middle - 1，因为当前这个nums[middle]一定不是target，那么接下来要查找的左区间结束下标位置就是 middle - 1

target 是在一个在左闭右开的区间里，**也就是[left, right)** 

- while (left &lt; right)，这里使用 &lt; ,因为left == right在区间[left, right)是没有意义的
- if (nums[middle] &gt; target) right 更新为 middle，因为当前nums[middle]不等于target，去左区间继续寻找，而寻找区间是左闭右开区间，所以right更新为middle，即：下一个查询区间不会去比较nums[middle]

```C#
public int SearchPlus(int[] nums, int target)
{
    var maxindex = nums.Length;
    var minindex = 0;
    var midindex = 0;
    while (minindex &lt; maxindex)
    {
        midindex = (maxindex &#43; minindex) / 2;
        if (target == nums[midindex])
        {
            return midindex;
        }
        if (target &lt; nums[midindex])
        {
            maxindex = midindex;
        }
        else
        {
            minindex = midindex&#43;1;
        }
    }
    return -1;
}
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/11/leecode704/  


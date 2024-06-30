# LeeCode-34-数组-查找第一个和最后一个位置


### 题目 

[题目链接](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

给你一个按照非递减顺序排列的整数数组 nums，和一个目标值 target。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 target，返回 [-1, -1]。

你必须设计并实现时间复杂度为 O(log n) 的算法解决此问题。

示例 1：

```ini
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]
```

示例 2：

```ini
输入：nums = [5,7,7,8,8,10], target = 6
输出：[-1,-1]
```

示例 3：

```ini
输入：nums = [], target = 0
输出：[-1,-1]
```


提示：

&#43; 0 &lt;= nums.length &lt;= 10&lt;sup&gt;5&lt;/sup&gt;
&#43; -10&lt;sup&gt;9&lt;/sup&gt; &lt;= nums[i] &lt;= 10&lt;sup&gt;9&lt;/sup&gt;
&#43; nums 是一个非递减数组
&#43; -10&lt;sup&gt;9&lt;/sup&gt; &lt;= target &lt;= 10&lt;sup&gt;9&lt;/sup&gt;

### 解题

二分法类似题，解题方案和前面两个相似

存在的变化，有重复数据，需要返回两个位置

尝试使用二分，找到值的序号后，向左，向右定位到上下限。

```C#
public int[] Search(int[] nums, int target)
{
    //[left，right]
    var left = 0;
    var right = nums.Length - 1;
    var result = new int[] {-1, -1};
    var mid = 0;
    while (left &lt;= right)
    {
        mid = (left &#43; right) &gt;&gt; 1;
        if (nums[mid] == target)
        {
            var min = mid;
            while (--min&gt;=left)//确定下限，使用遍历
            {
                if(nums[min] != target)
                    break;
            }

            result[0] = min &#43; 1;
            var max = mid;
            while (&#43;&#43;max &lt;= right)//确定上限，使用遍历
            {
                if (nums[max] != target)
                    break;
            }

            result[1] = max-1;
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
    return result;
}
```

如果数组中的重复项比较多，中间使用遍历可以优化为二分，同时判定目标值是否在数组中也改成递归进行统一。

```C#
public int[] SearchPlus(int[] nums, int target)
{
    var result = new int[] { -1, -1 };
    if (nums.Length == 0) return result;
    //[left，right]
    var left = 0;
    var right = nums.Length - 1;
    var mid = 0;
    //遍历定位左右限，type 0:返回下限序号，1返回上限序号 ,2找到即返回,
    //返回[left right,index]
    int[] InnterSearch(int[] nums, int target, int left, int right, int type)
    {
        if (left &lt;= right)
        {
            mid = (left &#43; right) &gt;&gt; 1;
            if (type == 2 &amp;&amp; target == nums[mid])
            {
                return new[] {left, right, mid};
            }
            else if ( target &lt; nums[mid]||(type == 0 &amp;&amp; target == nums[mid]))
            {
                right = mid - 1;
            }
            else if (target &gt; nums[mid] || (type == 1 &amp;&amp; target == nums[mid]))
            {
                left = mid &#43; 1;
            }
            return InnterSearch(nums, target, left, right, type);
        }

        if (type == 2)//不在数组中
        {
            return new[] {left, right, -1};
        }

        var index = type==1 ? left-1 : right&#43;1;

        return new[] {left, right, nums[index] == target ? index : -1};
    }

    var indexarray = InnterSearch(nums, target, left, right, 2);//数组中存在
    if (indexarray[2] == -1) return result;
    result[0]= InnterSearch(nums, target, indexarray[0], indexarray[2], 0)[2];//找到下限 [left,mid]中二分
    result[1] = InnterSearch(nums, target, indexarray[2], indexarray[1], 1)[2];//找到上限 [mid,right]中二分
    return result;
}
```

InnterSearch方法承担了三个职责，目标值是否在数组中，寻找上限，寻找下限。导致内部实现用了很多的判定，看上去有点乱，通过字典封装判断逻辑，一目了然，如果以后需求变化，改动字典即可，尽可能满足开放封闭原则。

```C#
public int[] SearchPlusPro(int[] nums, int target)
{
    var result = new int[] { -1, -1 };
    if (nums.Length == 0) return result;
    //[left，right]
    var left = 0;
    var right = nums.Length - 1;
    var mid = 0;
    //遍历定位左右限，type 0:返回下限序号，1返回上限序号 ,2找到即返回,
    //返回[left right,index]
    var searchtypes = new Dictionary&lt;int, Func&lt;int[], int, int, int,int, int[]&gt;&gt;()
    {
        {//下限
            0, (_nums, _target, _left, _right,_mid) =&gt;
            {
                if (_target &lt;= _nums[_mid])
                {
                    _right = _mid - 1;
                }
                else //target&gt;nums[mid]
                {
                    _left = _mid &#43; 1;
                }
                return new []{_left,_right,-1};
            }
        },
        {//上限
            1, (_nums, _target, _left, _right,_mid) =&gt;
            {
                if (_target &lt; _nums[_mid])
                {
                    _right = _mid - 1;
                }
                else //target&gt;=nums[mid]
                {
                    _left = _mid &#43; 1;
                }
                return new []{_left,_right,-1};
            }
        },
        {//找到立即返回
            2, (_nums, _target, _left, _right,_mid) =&gt;
            {
                if (_target == _nums[_mid])
                {
                    return new[] {_left, _right, _mid};
                }
                else if ( _target &lt; _nums[_mid])
                {
                    _right = _mid - 1;
                }
                else 
                {
                    _left = _mid &#43; 1;
                }
                return new  []{_left,_right,-1};
            }
        }
    };

    int[] InnterSearch(int[] nums, int target, int left, int right, int type)
    {
        if (left &lt;= right)
        {
            mid = (left &#43; right) &gt;&gt; 1;
            var temresult = searchtypes[type](nums, target, left, right, mid);
            if (temresult[2] != -1) return temresult;
            return InnterSearch(nums, target, temresult[0], temresult[1], type);
        }
        return new[] {left, right, -1};
    }

    var indexarray = InnterSearch(nums, target, left, right, 2);//数组中存在
    if (indexarray[2] == -1) return result;
    result[0] = InnterSearch(nums, target, indexarray[0], indexarray[2], 0)[0];//找到下限 [left,mid]中二分
    result[1] = InnterSearch(nums, target, indexarray[2], indexarray[1], 1)[1];//找到上限 [mid,right]中二分
    return result;
}
```

### 网络解析

网络解析有以下几种方式：

&#43; 对数组使用两次二分查找上限和下限。
&#43; 先判断目标在数组中，再使用二分查找上下限

```C#
public class Solution {
    public int[] SearchRange(int[] nums, int target)
    {
        if (nums.Length == 0)
            return new int[] { -1, -1 };
        int l = binarySearch(nums, target, true);
        int r = binarySearch(nums, target, false) - 1;
        if (l &lt;= r &amp;&amp; r &lt; nums.Length &amp;&amp; nums[l] == target &amp;&amp; nums[r] == target)
            return new int[] { l, r };
        return new int[] { -1, -1 };
    }
    int binarySearch(int[] nums,int target,bool lower)
    {
        int n = nums.Length;
        int l = 0, r = n - 1;
        while (l &lt;= r)
        {
            int m = (l &#43; r) / 2;
            if (nums[m] &gt; target || (lower &amp;&amp; nums[m] &gt;= target)) 
            {
                r = m - 1;
                n = m;
            }
            else
                l = m &#43; 1;
        }
        return n;
    }
}
```

```C#
public class Solution {
    public int[] SearchRange(int[] nums, int target) {
        int[] res = new int[2];
        int left=0;
        int right = nums.Length-1;
        while(left&lt;= right)
        {
           int mid = left&#43;(right-left)/2;
            if(nums[mid] &lt;target)
            {
                left = mid&#43;1;
            }else if(nums[mid] &gt;target)
            {
                right = mid-1;
            }else if (nums[mid]==target)
            {
                right = mid-1;
            }
        }
        if(left==nums.Length)
        {
            res[0] = -1;
        }else
        {
        res[0] = nums[left]==target?left:-1;
        }

          left=0;
         right = nums.Length-1;
        while(left&lt;= right)
        {
          int  mid = left&#43;(right-left)/2;
            if(nums[mid] &lt;target)
            {
                left = mid&#43;1;
            }else if(nums[mid] &gt;target)
            {
                right = mid-1;
            }else if (nums[mid]==target)
            {
                left = mid&#43;1;
            }
        }
        if(left-1 &lt;0)
        {
            res[1] = -1;
        }else
        {
        res[1] = nums[left-1]==target?left-1:-1;
        }
        return res;
    }
}
```


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/11/leecode34/  


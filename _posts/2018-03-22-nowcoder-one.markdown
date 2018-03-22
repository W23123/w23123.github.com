---
layout: post
title: 二维数组中查找
date: 2018-03-22 00:00:20 +0300
description: 二维数组中查找
img: i-rest.jpg # Add image post (optional)
tags: [算法]
---

#### 题目
在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

解法1
```java
public boolean Find(int target, int [][] array) {
        for (int i = 0; i < array.length; i++) {
            for (int j = 0; j < array[i].length; j++) {
                if (array[i][j] > target) {
                    break;
                } else if (array[i][j] < target) {
                    continue;
                }
                return true;
            }
        }
        return false;
    }
```
这里其实还是不够高效的，当array[i][j]>target时，对应下一行的array[i+1][j]>target也是成立的，所以在遍历的时候j的循环条件不应该为j < array[i].length，而是 j< 上次break时j的值。

故而新的解法如下(可能会有些复杂)

解法二
```java
public boolean Find ( int target, int[][] array){
            int i = 0;
            int m = -1;
            while (i < array.length) {
                int k = m == -1 ? array[i].length : m, j = 0;
                while (j < k) {
                    if (array[i][j] > target) {
                        m = j;
                        break;
                    } else if (array[i][j] < target) {
                        j++;
                        continue;
                    }
                    return true;
                }
                if (m == 0) {
                    return false;
                }
                i++;
            }
            return false;
        }
```
这种算法也不是最好的，因为已经给了排序的。在一个排好序的数组，查找一个值，理应用二分法最好。

实现如下

```java
public boolean Find(int target, int [][] array) {
        int i = 0;
        int m = -1;
        while (i < array.length) {
            int k = m == -1 ? array[i].length : m, j = 0;
            while (j < k) {
                int mid = (j + k - 1) / 2;
                if (array[i][mid] > target) {
                    k = mid;
                } else if (array[i][mid] < target) {
                    j = mid + 1;
                } else {
                    return true;
                }
            }
            m = k;
            if (m == 0) {
                return false;
            }
            i++;
        }
        return false;
    }
```
基本上久实现到这了，如果以后还有不错的实现，会更新的
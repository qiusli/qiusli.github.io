---
layout: post
title: "快速排序(Quicksort)"
date: 2013-08-23 21:57
comments: true
categories: 
---
Quick Sort即快速排序，它是对归并排序的一种补充，采用的也是分治策略。基本思想是在整个数组中选择第一个数作为一个基准，然后分别从第二个数向后和最后一个数向前开始扫描。如果在向后扫描的过程中如果遇到比基准大的数字(我们这里假设默认为升序排列)，同时在从后向前扫描的过程中遇到比基准小的数字，那么这两个数字就交换。这个过程一直持续下去直到从左扫描的下标大于从右开始扫描的下标作为结束。最后，把基准和右边下标指向的数交换，这样基准左边的数字都小于该基准，基准右边的数字都大于它。不断地递归这个过程就是快速排序。我们可以看看下面这个图帮助理解：
<!-- more -->
{% img /images/quick_sort-1.png %}
快速排序和归并排序的主要区别在于，归并排序是先递归再合并，而快速排序是先分解再递归。下面是分解的过程：
``` java
private static int partition(String[] a, int low, int high) {
    String value = a[low];
    int i = low + 1;
    int j = high;
    while (true) {
        while (a[i].compareTo(value) < 0 && i < high) {
            i++;
        }
        while (a[j].compareTo(value) > 0 && j > low) {
            j--;
        }
        if (i >= j)
            break;
        exchange(a, i, j);
    }
    exchange(a, low, j);
    return j;
}
```
Partition一次之后，基准的左边都小于它，右边都大于它。

下面是分解后递归的过程：
``` java
private static void sort(String[] a, int low, int high) {
    if (low >= high)
        return;
    int j = partition(a, low, high);
    sort(a, low, j - 1);
    sort(a, j + 1, high);
}
```
我们可以看出，这其实是在对每一个小分组都执行第一张图里所示的过程。    

最后整个程序如下：
``` java
public class QuickSort {
    private static String[] a = {"Q", "U", "I", "C", "K", "S", "O", "R", "T"};

    public static void main(String[] args) {
        sort(a, 0, a.length - 1);
        System.out.println(Arrays.toString(a));
    }

    private static void sort(String[] a, int low, int high) {
        if (low >= high)
            return;
        int j = partition(a, low, high);
        sort(a, low, j - 1);
        sort(a, j + 1, high);
    }

    private static int partition(String[] a, int low, int high) {
        String value = a[low];
        int i = low + 1;
        int j = high;
        while (true) {
            while (a[i].compareTo(value) < 0 && i < high) {
                i++;
            }
            while (a[j].compareTo(value) > 0 && j > low) {
                j--;
            }
            if (i >= j)
                break;
            exchange(a, i, j);
        }
        exchange(a, low, j);
        return j;
    }

    private static void exchange(String[] a, int i, int j) {
        String temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
}
```
---
title: Go 1.19 引入新型排序算法 pdqsort
cover_image: /img/cover.png
abbrlink: 8f95714c
date: 2023-04-03 03:46:22
---

经典排序算法

Insertion Sort 插入排序

Time complexity

| Best | Avg | Worst |
| O(n) | O(n^2) | O(n^2) |

优点

最好情况时间复杂度为 O(n)

缺点

平均和最坏情况时间复杂度高达O(n^2)

Quick Sort 快速排序

•选定一个 pivot•使用 pivot 分割序列，分成元素比 pivot 大和元素比 pivot 小两个序列，不断重复Time complexity

| Best | Avg | Worst |
| O(nlogn) | O(nlogn) | O(n^2) |

> 最好情况： pivot 为中位数
> 
> 最坏情况： pivot 为最大/小值
> 
> 优点

平均情况时间复杂度为 O(nlogn)

缺点

最坏情况时间复杂度 O(n^2)

Heap Sort 堆排序

• 构造一个大顶堆• 将根节点（最大元素）交换到最后一个位置，使堆大小 -1，调整整个堆，如此反复Time complexity

| Best | Avg | Worst |
| O(nlogn) | O(nlogn) | O(nlogn) |

优点

最坏情况时间复杂度为 O(nlogn)

缺点

最好情况时间复杂度为 O(nlogn)

实际场景 Benchmark

|  | Best | Avg | Worst |
| Insertion Sort | O(n) | O(n^2) | O(n^2) |
| Quick Sort | O(nlogn) | O(nlogn) | O(n^2) |
| Heap Sort | O(nlogn) | O(nlogn) | O(nlogn) |

• 插入排序平均和最坏情况都是 O(n^2)，性能不好• 快速排序整体性能处于中间• 堆排序性能比较稳定，各种情况均稳定为 O(nlogn)序列元素排列情况

• 完全随机 (random)• 有序/逆序 (sorted/reverse)• 元素重复度较高 (mod8)在此基础上，还需要根据序列长度划分(16/128/1024)

random

• 插入排序在短序列中速度最快• 快速排序在其他情况中速度最快• 堆排序速度与最快算法差距不大sorted

• 插入排序在序列已经有序的情况下最快

结论

• 所有短序列和元素有序情况下,插入排序性能最好• 在大部分的情况下，快速排序有较好的综合性能• 几乎在任何情况下，堆排序的表现都比较稳定pdqsort 算法

pdqsort (pattern-defeating-quicksort) 是一种不稳定的混合排序算法，它的不同版本被应用在C++ BOOST、Rust 以及Go 1.19中。它对常见的序列类型做了特殊的优化，使得在不同条件下都拥有不错的性能。

Version 1

• 对于短序列 (<=24) 我们使用插入排序• 其他情况，使用快速排序 (选择首个元素作为 pivot) 来保证整体性能• 当快速排序表现不佳时 (limit==0)，使用堆排序来保证最坏情况下时间复杂度仍然为 O(nlogn)> 当最终 pivot 的位置离序列两端很接近时 (距离小于 length/8) 判定其表现不佳，当这种情况的次数达到 limit (即 bits.Len(length))时，切换到堆排序。
> 
> Version 2

如何改进？

• 尽量使得 QuickSort 的 pivot 为序列的中位数 -> 改进 choose pivot

根据序列长度的不同，来决定选择策略

优化Pivot的选择

• 短序列 (<=8)，选择固定元素

• 中序列 (<=50)，采样三个元素，median of three

• 长序列 (>50)，采样九个元素，median of medians

同时，pivot 的这种采样方式使得我们有探知序列当前状态的能力

采样的元素都是逆序排列 -> 序列可能已经逆序 -> 翻转整个序列

采样的元素都是顺序排列 -> 序列可能已经有序 -> 使用插入排序

> 插入排序实际使用 partiallnsertionSort，即有限制次数的插入排序
> 
> 

Final version

还有一种场景，元素重复较多的场景。

如果两次 partition 生成的 pivot 相同，即 partition 进行了无效分割，此时认为 pivot 的值为重复元素。

优化：当检测到此时的 pivot 和上次相同时（发生在 leftSubArray)，使用 partitionEqual 将重复元素排列在一起，减少重复元素对于 pivot 选择的干扰。

其他优化：pivot 选择策略表现不佳时，随机交换元素。

避免一些极端情况使得 QuickSort 总是表现不佳，以及一些黑客攻击情况。

Reference

[1] sort: use pdqsort · Issue #50154 · golang/go

    https://github.com/golang/go/issues/50154

[2] Pattern-defeating Quicksort

    https://arxiv.org/abs/2106.05123

[3] go/zsortinterface.go at master · golang/go

    https://github.com/golang/go/blob/master/src/sort/zsortinterface.go


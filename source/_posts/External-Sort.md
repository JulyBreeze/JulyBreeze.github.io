---
title: External Sort
date: 2019-02-12 11:35:11
tags: sort
---
Internal sort，是能够在计算机的内存中直接完成排序任务的算法，主要有：
bubber sort  
quick sort  
heap sort  
merge sort  

如果数据量太大，无法一次性装入内存，只能放在外存储器中(通常是硬盘)。 
External Sort, O(nlogn):  
1)局部排序 
先读入能够放在内存中的数据量，将排序输出到一个临时文件中。最终获得多个有序的临时文件。  
2)归并  
将这些临时文件合并成一个大的有序文件。合并时，分别从每个临时文件中取得m大小的数据，放入内存，内存留出部分的输出缓冲区。  
在内存里归并这些数据，并把结果放入缓冲区。当缓冲区满时，把数据写入外部文件中。清空缓冲区。
当来自i文件的数据被处理完毕，就从i文件中继续读入下一堆数据，直到i文件为空。



If both nums1 and nums2 are so huge that neither fit into the memory, sort them individually (external sort), then read 2 elements from each array at a time in memory, record intersections.
---
title: C++ STL istream_iterator、ostream_iterator
toc: true
date: 2018-08-01 18:06:13
---
# C++ STL istream_iterator、ostream_iterator

今天看一道算法题里面有这么一句：


    	int arr[] = { 7, 5, 6, 4 };
    	vector<int> vec(arr, arr + 4);
    	copy(vec.begin(), vec.end(), ostream_iterator<int>(cout, " "));


以前好像没有看到过 ostream_iterator 的使用，因此要总结下。





# 相关

1. [STL中 istream_iterator和 ostream_iterator的基本用法](https://www.cnblogs.com/VIPler/p/4367308.html)

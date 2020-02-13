---
title: 算法 最小的K个数
toc: true
date: 2018-06-11 08:14:54
---

# 相关






  1.


[CodingInterviews](https://github.com/gatieme/CodingInterviews)







## 可以补充进来的






  * **从最小堆开始后面的几种方法都没有仔细看，需要看下。**


  * **这里遇到堆相关的了，要整理一下堆相关的一些东西，很多都不清楚。**





* * *





# INTRODUCTION






  * aaa




# 题目




## **题目描述**


输入 n 个整数，找出其中最小的 K 个数。

例如输入 4,5,1,6,2,7,3,8 这 8 个数字， 则最小的 4 个数字是 1,2,3,4 。


##




# 题目解答


有八种方法，分别如下：




  * 方法一：排序


  * 方法二：选择或者交换排序


  * 方法三：最小堆


  * 方法四：快速排序的分治划分（中位数作为枢轴）


  * 方法五：快速排序的分治划分（随机枢轴）


  * 方法六：线性排序


  * 方法七：最小堆与优先队列


  * 方法八：提取最小堆的元素


OK，我们依次看下：


## 方法一：排序


看到说要求一个序列中最小的 K 个数，那么按照惯有的思维方式，我们可以直接先对这个序列从小到大排序，然后输出前面的最小的 K 个数即可。

至于选取什么样的排序方法，第一时间应该想到的是快速排序，我们知道，快速排序平均时间复杂度为 O(nlogn) ，然后再遍历输出序列中前 K 个元素即可，总的时间复杂度为 O(nlogn + K) = O(nlogn)。


## 方法二：选择或者交换排序


再进一步想想，其实题目并没有要求要查找的 K 个数，甚至是后面的 n-K 个数是有序的，既然这样，咱们又何必对所有的 n 个数都进行排序呢？是的。

这个时候，我们可以这样：




  * 我们遍历这 n 个数，把最先遍历到的 K 个数存入大小为 K 的数组之中，对这 K 个数，利用选择或交换排序，找到 K 个数中的最大数 Kmax ( Kmax 为这 K 个元素的数组中最大的元素 )，这个操作使用的时间为 O(K)（你应该知道，插入或选择排序查找操作需要 O(K) 的时间），


  * 然后我们再继续遍历后 n-K 个数，每个 x 与 Kmax 比较：如果 x< Kmax ，则 x 代替 Kmax ，并再次重新找出 K 个元素的数组中的最大元素 Kmax'；如果 x>Kmax ，则不更新数组。


按照这样，每次更新和不更新数组所用的时间为 O(K) 或 O(0)，而整趟下来，总的时间复杂度平均下来为：nO(K) = O(nK)。**可见，K 比较小的时候，这个比排序法要快一些的。**


## 方法三：最小堆


当然，还有更好的办法：

原理其实与上面的第 2 个方案一致，不过这里我们维护的是一个 K 个元素的大顶堆，我们先用这个大顶堆存储最先遍历到的 K 个数，并假设它们即是最小的 K 个数（建堆需要 O(K) ），然后我们继续遍历数列，每次遍历一个元素 x，与堆顶元素比较，除非小于大顶堆的最大值，否则不更新堆。

由于堆的性质，因此这样下来，总费时 O(K+(n-K)logK) = O(nlogK) 。**要确认下。关于堆 很多都不清楚，需要好好总结下。**

这个方法得益于在堆中，查找等各项操作时间复杂度均为 logK。


## 方法四：快速排序的分治划分（中位数作为枢轴）


我们可以使用一种类似快速排序的划分方法：




  * 我们把 N 个数存储在数组 S 中，再从数组中随机选取一个数 X（随机选取枢纽元，可做到线性期望时间 O(N)的复杂度），把数组划分为 Sa 和 Sb 两部分，且 Sa<= X <=Sb。**这个划分怎么实现的？**


    * 如果要查找的 K 个小的元素小于 Sa 中的元素个数，则返回 Sa 中较小的 K 个元素，


    * 否则返回 Sa 中 K 个小的元素 + Sb中小的 K-|Sa|个元素。





像上述过程一样，这个运用类似快速排序的 partition 的快速选择 Select 算法寻找最小的 K 个元素，在最坏的情况下亦能做到 O(N) 的复杂度。

不过值得一提的是，这个快速选 Select 算法是选择数组中 “中位数的中位数” 作为枢纽元，而非随机选择枢纽元。


## 方法五：快速排序的分治划分（随机枢轴）


Randomized-Select，每次都是随机选择数列中的一个元素作为主元，在 O(n)的时间内找到第 K 小的元素，然后遍历输出前面的 K 个小的元素。如果能的话，那么总的时间复杂度为线性期望时间：O(n+k) = O(n)（当 n 比较小时）；


## 方法六：线性排序


线性时间的排序，即计数排序，时间复杂度虽能达到 O(n)，但是，限制条件太多了，不常用；


## 方法七：最小堆与优先队列


”可以用最小堆初始化数组，然后取这个优先队列前 k 个值。复杂度为 O(n)+k_O(logn)“。意思是针对整个数组序列建立最小堆，建堆所用时间为 O(n)，然后取堆中的前 k 个数，即总的时间复杂度为：O(n+k_logn)。


## 方法八：提取最小堆的元素


与上述思路 7 类似，不同的是在对元素数组原地建立最小堆 O(n)后，然后提取 K 次，但是每次提取时，换到顶部的元素只需要下移顶多 K 次就足够了，下移次数逐次减少（而上述思路 7 每次提取都需要 logn，所有提取 K 次，思路 7 需要 K*logn，而本思路 8 只需要 K^2）；



完整代码如下：


    #include <iostream>
    #include <vector>
    #include <algorithm>
    #include <cstring>
    using namespace std;

    class Solution {
    protected:
    	vector<int> m_res;
    public:

    	vector<int> GetLeastNumbers_Solution(vector<int> numbers, int k) {
    		m_res.clear();

    		if (numbers.size() == 0 || numbers.size() < k) {
    			return m_res;
    		}

    		cout << "****************  方法 1：排序   ***************" << endl;
    		m_res.clear();
    		LeastKNumbers_BySort(numbers, k);
    		PrintResVector();


    		cout << "*********  方法 2：把 numbers 中的数转化为 index   ********" << endl;
    		m_res.clear();
    		LeastKNumbers_ByCountSort(numbers, k);
    		PrintResVector();

    		cout << "********* 方法 3：采用选择排序法, K趟找出前 K 个数字   ********" << endl;
    		m_res.clear();
    		LeastKNumbers_BySelectSort(numbers, k);
    		PrintResVector();


    		//
    		//        m_res.clear( );
    		//        LeastKNumbers_ByBubbleSort(numbers, k);




    		return m_res;
    	}
    	void PrintResVector() {
    		for each (int var in m_res) {
    			cout << var << " ";
    		}
    		cout << endl;
    	}
    	//直接排序后输出前 K 个数字
    	vector<int> LeastKNumbers_BySort(vector<int> numbers, int k) {
    		sort(numbers.begin(), numbers.end());//先进行排序
    		for (int i = 0; i < k; i++) {
    			m_res.push_back(numbers[i]);//把最小的几个数直接放到 m_res里面
    		}
    		return m_res;
    	}
    	//把 numbers 中的数转化为 index
    	vector<int> LeastKNumbers_ByCountSort(vector<int> numbers, int k) {
    		int i, count;
    		int num[1000];
    		memset(num, '\0', 1000);
    		//把 numbers 里的数作为 num 的 index ，然后把这个地方设定为 1
    		for (i = 0; i < numbers.size(); i++) {
    			num[numbers[i]] = 1; //把这个数对应的 index 地方的数设置为 1
    		}

    		//OK，既然已经设定好了，那么最小的 k 个 index 就是 numbers里面的最小的 k 个数。
    		for (i = 0, count = 0; i < 1000 && count < k; i++) {
    			if (num[i] != 0) {
    				count++;
    				m_res.push_back(i);//将统计到的 index 存放到 m_res 里面 , 这就是 numbers 里面最小的几个数
    			}
    		}
    		return m_res;
    	}


    	//采用选择排序法, K趟找出前 K 个数字
    	//由于选择排序每趟结束后前 i 个数字都有序，因此 K 趟即可找出前 K 小的数字
    	vector<int> LeastKNumbers_BySelectSort(vector<int> numbers, int k) {
    		int i, j, index;
    		int length = numbers.size() - 1;
    		// 循环每趟排序
    		for (i = 0; i < k/*length*/; i++) {
    			index = i;
    			for (j = i + 1; j < length; j++) {
    				if (numbers[j] < numbers[index]) {
    					index = j;                 // 找到当前极值元素的下标
    				}
    			}
    			if (index != i) {
    				swap(numbers[i], numbers[index]);     // 将极值保存到应该填入的位置
    			}
    			m_res.push_back(numbers[i]);

    			cout << "pos = " << index << ", num = " << numbers[i] << endl;
    			cout << "when " << i << " select sort, the least " << i << " numbers is sorted" << endl;

    			for (int pos = 0; pos <= i; pos++) {
    				cout << numbers[pos] << " ";
    			}
    			cout << endl;
    		}
    		return m_res;
    	}

    	///  采用冒泡排序法, K趟找出前 K 个数字
    	vector<int> LeastKNumbers_ByBubbleSort(vector<int> numbers, int k) {
    		cout << endl << "line " << __LINE__ << " in function : " << __FUNCTION__ << endl << endl;

    		int j/*控制每趟循环*/, i/*控制相邻数据的比较循环*/;
    		// 排序过程
    		int length = numbers.size();
    		for (i = 0; i < k/*length - 1*/; i++)        // 共计进行 length-1趟循环
    		{
    			for (j = length - i - 1; j > 0; j--)   // 每趟循环比较 length-1-j次
    			{
    				if (numbers[j - 1] > numbers[j])    // 如果当前的元素与后一个元素不满足排序规则
    				{
    					swap(numbers[j - 1], numbers[j]);
    				}

    			}
    			m_res.push_back(numbers[i]);

    			cout << "pos = " << i << ", num = " << numbers[i] << endl;
    			cout << "when " << i << " bubble sort, the least " << i << " numbers is sorted" << endl;

    			for (int pos = 0; pos <= i; pos++) {
    				cout << numbers[pos] << " ";
    			}
    			cout << endl;
    		}

    		return m_res;
    	}



    	vector<int> GetLeastNumbers_ByFindKth(vector<int> numbers, int k) {
    		int kth;
    		vector<int> res;
    		for (int i = 0; i < k; i++) {
    			kth = FindKth(numbers, 0, numbers.size() - 1, i);
    			cout << i << " th is " << kth << endl;
    			res.push_back(kth);
    		}
    		return res;
    	}
    	//待查找元素所在的数组 left 数据起始元素的位置 right 数据终止元素的位置 k 第 K 大的元素
    	int FindKth(vector<int> &numbers, int left, int right, int k) {
    		int res;
    		int pivotIndex = left + 1;

    		if (left == right) {
    			cout << "left == right" << numbers[left] << endl;
    			return numbers[left];
    		}
    		pivotIndex = Partition(numbers, left, right);

    		if (pivotIndex < k)          //  当前查找到的比第 K 个数小
    		{
    			// 第 K 大的的在 pivot 之前, 在[pivotIndex + 1, right]之间查找
    			cout << "K is in [" << pivotIndex + 1 << ", " << right << "]" << endl;
    			return FindKth(numbers, pivotIndex + 1, right, k);
    		}
    		else if (pivotIndex > k)     //  当前查找到的比第 K 个数大
    		{
    			// 第 K 小的在 pivot 之前, 在[left, pivotIndex - 1]之间查找
    			cout << "K is in [" << left << ", " << pivotIndex - 1 << "]" << endl;
    			return FindKth(numbers, left, pivotIndex - 1, k);
    		}
    		else      //  这里返回的是其位置
    		{
    			/// return pivotIndex; ///  error
    			cout << "pivotIndex == k, " << numbers[pivotIndex] << endl;
    			return numbers[pivotIndex];
    		}
    	}
    	//快速排序的划分函数, 返回枢轴(left下标元素)在排序的数组中应该所处的位置, 即下标为 left 的元素是第几大的元素
    	int Partition(vector<int> &numbers, int left, int right) {
    		int i = left, j = right;

    		///  我们选择第一个元素作为基准
    		///  这个也可以随机选择
    		int pivotIndex = left, pivotNum = numbers[pivotIndex];

    		cout << "pivotNum = " << pivotNum << endl;
    		while (i < j) {
    			while (i < j && numbers[j] >= pivotNum) {
    				cout << "[" << i << ", " << j << "] " << numbers[j] << " >= " << pivotNum << endl;
    				j--;
    			}
    			cout << "now we find i = " << i << ", posJ = " << j << ", num = " << numbers[j] << " < " << pivotNum << endl;;
    			numbers[i] = numbers[j];        // 将找到的那个比枢轴小的数，放在枢轴左侧 I 的位置
    			//swap(numbers[i], numbers[j]); // 也可以使用交换, 但是没有必要, 因为枢轴的位置并没有最后确定

    			while (i < j && numbers[i] <= pivotNum) {
    				cout << "[" << i << ", " << j << "] " << numbers[i] << " <= " << pivotNum << endl;
    				i++;
    			}

    			cout << "now we find j = " << i << ", posI = " << j << ", num = " << numbers[i] << " > " << pivotNum << endl;;
    			numbers[j] = numbers[i];        // 将找到的那个比枢轴大的数，放在枢轴右侧 J 的位置
    			//swap(numbers[i], numbers[j]); // 也可以使用交换, 但是没有必要, 因为枢轴的位置并没有最后确定
    		}

    		numbers[i] = pivotNum;              //  最后的位置 i == j 就是枢轴的位置


    		cout << pivotNum << " 's post is" << i << endl;
    		for (int pos = left; pos <= right; pos++) {
    			cout << numbers[pos] << " ";
    		}
    		cout << endl;


    		return i;
    	}




    	//竟然可以这样
    	class greater_class {
    	public:
    		bool operator()(int a, int b) {
    			return a > b;
    		}
    	};
    	vector<int> LeastKNumbers_ByMinHeap(vector<int> numbers, int k) {
    		vector<int> res;

    		if (numbers.size() == 0 || numbers.size() < k) {
    			return res;
    		}
    		make_heap(numbers.begin(), numbers.end(), greater_class());

    		for (int i = 0; i < k; i++) {
    			//  最小的元素在栈顶
    			cout << numbers[0] << " ";
    			res.push_back(numbers[0]);

    			///  一下两种操作均可以
    			// [1]  --  清除它, 然后重新排序堆
    			//numbers.erase(numbers.begin( ));
    			//sort_heap(numbers.begin( ), numbers.end( ));

    			// [2]  --  当然从堆出弹出这个栈顶元素
    			pop_heap(numbers.begin(), numbers.end(), greater_class());   // 弹出一个元素后，剩下的又重建了 heap，仍保持 heap 的性质
    			numbers.pop_back();         // vector 删除末尾元素
    		}

    		return res;
    	}

    };


    int main() {
    	int arr[] = { 4, 5, 1, 6, 2, 7, 13, 8 };
    	std::vector<int> vec(arr, arr + 8);
    	Solution solu;
    	solu.GetLeastNumbers_Solution(vec, 4);
    	return 0;
    }


输出：


    ****************  方法 1：排序   ***************
    1 2 4 5
    *********  方法 2：把 numbers 中的数转化为 index   ********
    1 2 4 5
    ********* 方法 3：采用选择排序法, K趟找出前 K 个数字   ********
    pos = 2, num = 1
    when 0 select sort, the least 0 numbers is sorted
    1
    pos = 4, num = 2
    when 1 select sort, the least 1 numbers is sorted
    1 2
    pos = 2, num = 4
    when 2 select sort, the least 2 numbers is sorted
    1 2 4
    pos = 4, num = 5
    when 3 select sort, the least 3 numbers is sorted
    1 2 4 5
    1 2 4 5


**从方法 3 开始没有看，而且原来的代码跟上面的讲解也是不是很对应的，要自己都整理一下。**















* * *





# COMMENT




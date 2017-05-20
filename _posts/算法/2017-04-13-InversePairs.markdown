---
layout: post
title:  "数组中的逆序对(归并排序)"
categories: 算法
---
### [数组中的逆序对](https://www.nowcoder.com/practice/96bd6684e04a44eb80e6a68efc0ec6c5?tpId=13&tqId=11188&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

### 题目描述
在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007  
### 输入例子
&emsp; 1,2,3,4,5,6,7,0  
### 输出例子
&emsp; 7  
按照最朴素的算法，就是遍历数组，把当前元素i和它后面的元素j想比较，如果i>j，则逆序对的个数加1。代码如下：  

	public class Solution {
	    public int InversePairs(int [] array) {
	        int count = 0;
	        for(int i = 0; i < array.length; i++){
	            for(int j = i + 1; j < array.length; j++){
	                if(array[i] > array[j])
	                    count = (count + 1) % 1000000007;
	            }
	        }
	        return count;
	    }
	}

运行超时...  
参考《剑指offer》,采用归并排序思想来查找逆序对,主要思想是把一段数组分成两段，两段数组内元素都是升序的。比如：  
<table>
<tr><th style="width: 200px;">第一段</th><th style="width: 200px;">第二段</th></tr>
<tr align="center"><td>1,  2,  4,  5</td><td>3,  6,  7,  8</td></tr>
</table>
合并的时候生成新数组，第一段4和第二段3比较，4比3大，那么第一段里包括4在内后面的数都可以和3组成逆序对。  
AC代码如下：  

	public class Solution {
	     
	    private int count = 0;
	     
	    public int[] mergeSort(int[] arr, int low, int high){
	        if (low == high)
	            return new int[]{arr[low]};
	        int mid = (low + high) >> 1;
	        int[] arr1 = mergeSort(arr, low, mid);
	        int[] arr2 = mergeSort(arr, mid + 1, high);
	        int[] res = new int[arr1.length + arr2.length];
	        int i = 0, j = 0, k = 0;
	        while(i < arr1.length && j < arr2.length){
	            if(arr1[i] <= arr2[j]){
	                res[k++] = arr1[i++];
	            }else{
	                res[k++] = arr2[j++];
	                count += arr1.length - i;
	                count = count % 1000000007;
	            }
	        }
	        if(i >= arr1.length){
	            System.arraycopy(arr2, j, res, k, arr2.length - j);
	        }
	        if(j >= arr2.length){
	            System.arraycopy(arr1, i, res, k, arr1.length - i);
	        }
	        return res;
	    }
	     
	    public int InversePairs(int [] array) {
	        int[] res = mergeSort(array, 0, array.length - 1);
	        return count;
	    }
	}

#### 归并排序
下面来说一下归并排序，jdk1.7之前Arrays.sort()使用的是归并排序，源代码：

	public static void sort(Object[] a) {
		Object[] aux = (Object[])a.clone();
		mergeSort(aux, a, 0, a.length, 0);   
	}  

jdk1.7后使用的是归并排序和Timsort, 源代码：  

	public static void sort(Object[] a) {
		if (LegacyMergeSort.userRequested)
		    legacyMergeSort(a);
		else
		    ComparableTimSort.sort(a);
	}
	
	private static void legacyMergeSort(Object[] a) {
	    Object[] aux = a.clone();
	    mergeSort(aux, a, 0, a.length, 0);
	}

mergeSort()源码如下：

	private static void mergeSort(Object[] src,
	                              Object[] dest,
	                              int low,
	                              int high,
	                              int off) {
	    int length = high - low;
	
	    // Insertion sort on smallest arrays
	    if (length < INSERTIONSORT_THRESHOLD) {
	        for (int i=low; i<high; i++)
	            for (int j=i; j>low &&
	                     ((Comparable) dest[j-1]).compareTo(dest[j])>0; j--)
	                swap(dest, j, j-1);
	        return;
	    }
	
	    // Recursively sort halves of dest into src
	    int destLow  = low;
	    int destHigh = high;
	    low  += off;
	    high += off;
	    int mid = (low + high) >>> 1;
	    mergeSort(dest, src, low, mid, -off);
	    mergeSort(dest, src, mid, high, -off);
	
	    // If list is already sorted, just copy from src to dest.  This is an
	    // optimization that results in faster sorts for nearly ordered lists.
	    if (((Comparable)src[mid-1]).compareTo(src[mid]) <= 0) {
	        System.arraycopy(src, low, dest, destLow, length);
	        return;
	    }
	
	    // Merge sorted halves (now in src) into dest
	    for(int i = destLow, p = low, q = mid; i < destHigh; i++) {
	        if (q >= high || p < mid && ((Comparable)src[p]).compareTo(src[q])<=0)
	            dest[i] = src[p++];
	        else
	            dest[i] = src[q++];
	    }
	}


JDK源码中对(2路)归并排序进行了修改，当分段的长度小于INSERTIONSORT_THRESHOLD(默认为7)时，采用直接插入排序。  
归并排序使用的是分治的思想，将复杂的问题分而治之，一个问题可以分成两个或多个子问题，每个子问题和它们的父问题是解决起来可以是同类方案。所以一般采用递归的方式。 

#### 树状数组
此题还可以采用树状数组的方式来计算逆序对，贴上牛客网讨论区马客(Mark)的代码：

	#define lb(x) ((x) & -(x))
	typedef long long ll;
	class BIT{
	public:
		int n;
		vector<int> a;
		BIT(intn) : n(n), a(n + 1, 0){}
		void add(int i, int v){
			for(; i <= n; i += lb(i))
				a[i] += v;
		}
		ll sum(int i){
			int r = 0;
			for(; i; i -= lb(i))
				r += a[i];
			return r;
		}
	};
	
	class Solution {
	public:
		int InversePairs(vector<int> d){
			int n = d.size();
			vector<int> t(d);
			sort(t.begin(), t.end());	//排序
			t.erase(unique(t.begin(), t.end()), t.end());	//去除重复元数（当然这题没有重复元素）
			for(inti = 0; i < n; ++i)	//离散化
				d[i] = lower_bound(t.begin(), t.end(), d[i]) - t.begin() + 1;
			BIT bit(t.size());
			ll ans = 0;
			for(inti = n - 1; i >= 0; --i){
				ans += bit.sum(d[i] - 1);
				bit.add(d[i], 1);
			}
			returnans % 1000000007;
		}
	};


树状数组的详细介绍参见：
<a href="http://blog.jobbole.com/96430/" target="_blank">夜深人静写算法（3）：树状数组</a>

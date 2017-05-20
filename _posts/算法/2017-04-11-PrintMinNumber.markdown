---
layout: post
title:  "把数组排成最小的数(字符串排序)"
categories: 算法
---

### [把数组排成最小的数](https://www.nowcoder.com/practice/8fecd3f8ba334add803bf2a06af1b993?tpId=13&tqId=11185&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)
输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。  
**思路**：  
如果把所有的数字全排序，会有n!种情况，时间复杂度太大。  
考虑先将问题简单特殊化，比如数组中只包含2个元素{32， 321}，那存在两种组合情况32321和32132,只用比较32321和32132的大小。整数转化为字符串，也就是比较"32321"和"32132"的大小。显然"32132"比较小，数组的顺序为{321, 32}。当数组中再增加一个数字3时，3可以插入的位置情况是<font color="red">3</font>32132, 321<font color="red">3</font>32, 32132<font color="red">3</font>，判断它们大小的过程实际上是按照上述的比较条件，将"3"与"321","32"进行比较。  
以此类推，当数组个数为n时，问题转化为特定判断条件排序问题。<font color="red">判断条件为：比较(元素1元素2)和(元素2元素1)的大小。</font>

	import java.util.ArrayList;
	import java.util.Collections;
	import java.util.Comparator;
	
	public class MinNumber {
		public String PrintMinNumber(int [] numbers) {
			ArrayList<String> list = new ArrayList<String>();
			for(int i = 0; i < numbers.length; i++){
				list.add(String.valueOf(numbers[i]));
			}
			Collections.sort(list, new Comparator<String>(){
				@Override
				public int compare(String o1, String o2) {
					String str1 = o1 + o2;
					String str2 = o2 + o1;
					return str1.compareTo(str2);
				}		
			});
			String str = "";
			for(String item : list){
				str += item;
			}
			return str;
	    }
	}



这里比较有意思的一段代码是：  

	Collections.sort(list, new Comparator<String>(){
				@Override
				public int compare(String o1, String o2) {
					String str1 = o1 + o2;
					String str2 = o2 + o1;
					return str1.compareTo(str2);
				}	
			});

Collections.sort()的源码如下：  
	
	public static <T> void sort(List<T> list, Comparator<? super T> c) {
		Object[] a = list.toArray();
		Arrays.sort(a, (Comparator)c);
		ListIterator i = list.listIterator();
		for (int j=0; j<a.length; j++) {
		    i.next();
		    i.set(a[j]);
		}
	}


经常发现有List<? super T>、Set<? extends T>的声明，是什么意思呢？<? super T>表示包括T在内的任何T的父类，<? extends T>表示包括T在内的任何T的子类。<br> Collections.sort()实际是调用的Arrays.sort(),Arrays.sort()采用的排序方法是归并排序和Timsort。  
让人疑惑的是Comparator是一个接口, new Comparator()意味着创建一个接口，显然是不合理的，接口是不能被直接实例化的。那为什么这里可是这样用呢？  
参考《JAVA编程思想》和[http://www.cnblogs.com/nerxious/archive/2013/01/25/2876489.html](http://www.cnblogs.com/nerxious/archive/2013/01/25/2876489.html)，有一种匿名内部类的东西。 
 
>匿名内部类也就是没有名字的内部类  
>正因为没有名字，所以匿名内部类只能使用一次，它通常用来简化代码编写  
>但使用匿名内部类还有个前提条件：必须继承一个父类或实现一个接口  

new Comparator<String>(){...}将对象的创建与类的定义结合在一起，这个类是匿名的，也就是没有名字的。看起来似乎是你正要创建一个Comparator接口的实例化对象，却发现没有还没有定义实现Comparator类。上述匿名内部类的用法是下述形式的简化：  

	class MyComparator extends Object implements Comparator<String>{
		@Override
		public int compare(String o1, String o2) {
			String str1 = o1 + o2;
			String str2 = o2 + o1;
			return str1.compareTo(str2);
		}
	}
	
	Comparator<String> c = new MyComparator();
	Collections.sort(list, c);

Comparator接口中包含两个方法：  
int compare(T o1, T o2);  
boolean equals(Object obj);  
之所以equals()没有被显示的Override，是因为所有的类都继承了Object, Object类定义了equals()的实现。  

### 策略模式
Collections.sort(list, c);使用了策略模式的思想。
>程序设计的基本目标是“将不变的事物与会发生变化的事物相分离”  

而这里不变的是通用的排序算法Collections.sort(), 变化的是各种对象的比较方式。通过使用策略模式，可以将“会变化的代码”封装在单独的类中(策略对象)。可以将不同的策略对象传递给相同的代码，这些代码使用不同的策略来完成其算法。

	class ComparatorA extends Object implements Comparator<String>{
		@Override
		public int compare(String o1, String o2) {
			...
		}
	}
	
	class ComparatorB extends Object implements Comparator<String>{
		@Override
		public int compare(String o1, String o2) {
			...
		}
	}
	
	Comparator<String> cA = new ComparatorA();
	Comparator<String> cB = new ComparatorB();
	Collections.sort(list, cA);

这样可以改变sort()的第二个参数改变不同的比较策略。  

***
参考资料：  
[Java ArrayList的不同排序方法](http://www.importnew.com/17211.html)  
[java中的匿名内部类总结](http://www.cnblogs.com/nerxious/archive/2013/01/25/2876489.html)

---
layout: post
title:  "How Many Tables"
categories: 算法
---
### Problem Description
Today is Ignatius' birthday. He invites a lot of friends. Now it's dinner time. Ignatius wants to know how many tables he needs at least. You have to notice that not all the friends know each other, and all the friends do not want to stay with strangers.

One important rule for this problem is that if I tell you A knows B, and B knows C, that means A, B, C know each other, so they can stay in one table.

For example: If I tell you A knows B, B knows C, and D knows E, so A, B, C can stay in one table, and D, E have to stay in the other one. So Ignatius needs 2 tables at least.
 

### Input
The input starts with an integer T(1<=T<=25) which indicate the number of test cases. Then T test cases follow. Each test case starts with two integers N and M(1<=N,M<=1000). N indicates the number of friends, the friends are marked from 1 to N. Then M lines follow. Each line consists of two integers A and B(A!=B), that means friend A and friend B know each other. There will be a blank line between two cases.
 

### Output
For each test case, just output how many tables Ignatius needs at least. Do NOT print any blanks.
 

### Sample Input
2  
5 3  
1 2  
2 3  
4 5  

5 1  
2 5   

### Sample Output
2  
4  

[How Many Tables](http://acm.hdu.edu.cn/showproblem.php?pid=1213)

题意为：如果A和B是朋友，B和C也是朋友，那么A和C也是朋友。以此类推，问有多少组朋友集合。

问题转化为存在图G=(V, E), 若V1和V2是朋友，则在他们之间建立一条边，最后求有多少个连通子图。此类问题通常用并查集（Union-Find Set）解决。在算法导论中称为不相交集合数据结构(disjoint-set data structure)。
> 集合中的每个元素是由一个对象表示的。设x表示一个对象，支持一下操作：
><ol>
><li>MAKE-SET(x):建立一个新的集合，其唯一成员(因而其代表)就是x,因为各集合是不相交的，故x没有在其他集合中出现过。</li>
><li>UNION(x, y):将包含x和y的动态集合(比如说Sx和Sy)合并为一个新的集合(即两个集合的并集)，经过此操作后，所得的集合代表可以是Sx U Sy中的任何成员，但在很多Union实现中，都选择Sx或Sy的代表作为新的代表。</li>
><li>FIND-SET(x):返回一个指针，指向包含x的(唯一集合代表)。</li>
></ol>


并查集中这三种典型操作的时间复杂度都是线性的。

{% highlight java linenos%}

	import java.util.Scanner;
	public class Main {
	    private static int[] set = new int[1001];
	    public static void main(String args[])
	    {
	        Scanner sc = new Scanner(System.in);
	        int t = sc.nextInt();
	        while(t-- != 0)
	        {
	            int n = sc.nextInt();   //n为朋友的个数
	            int m = sc.nextInt();   //m为关系数
	            int count = 0;          //组数
				//MAKE-SET，将每个节点初始为一个集合
	            for(int i = 1; i < n+1; i++)
	                set[i] = i;
	            while(m-- != 0)
	            {
	                int a = sc.nextInt();
	                int b = sc.nextInt();
	                union(a, b);
	            }
	            for (int i = 1; i < n+1; i++){
	                if(set[i] == i)
	                    count ++;
	            }
	            System.out.println(count);
	        }
	    }
	    
		//UNION
	    public static void union(int a, int b)
	    {
	        int ra = find(a);
	        int rb = find(b);
	        if (ra < rb)
	            set[rb] = ra;
	        if (ra > rb)
	            set[ra] = rb;
	    }
	    
		//FIND-SET
	    public static int find(int x)
	    {
	        int r = x;
	        while(set[r] != r)
	            r = set[r];
	        return r;
	    }
	}
{% endhighlight %}

其中set[x] = y 表示x的父节点为y, 若set[x] = x，则x为根节点，即代表元。

![](https://github.com/Chiryen/chiryen.github.io/blob/master/img/%E5%B9%B6%E6%9F%A5%E9%9B%86%E8%B7%AF%E5%BE%84%E5%8E%8B%E7%BC%A9.png?raw=true)

当树的深度过大时，查找的开销会变大。采用路径压缩后，所有的节点set[x] = root, 其实这里用parent[x]更合适。路径压缩就是在FIND时，让x及其祖先节点都指向根节点。

{% highlight java linenos%}
	public static int find(int x)
	{
	    int r = x;
	    while(set[r] != r)
	        r = set[r];
	    int i = x, j;
	    while(i != r)
	    {
	        j = set[i];
	        set[i] = r;
	        i = j;
	    }
	    return r;
	}
{% endhighlight %}
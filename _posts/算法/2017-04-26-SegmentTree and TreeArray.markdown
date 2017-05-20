---
layout: post
title:  "线段树与树状数组"
categories: 算法
---
### [Color the ball](http://localhost:4040/jekyll/update/2017/04/26/SegmentTree-and-TreeArray.html)

#### Problem Description
N个气球排成一排，从左到右依次编号为1,2,3....N.每次给定2个整数a b(a <= b),lele便为骑上他的“小飞鸽"牌电动车从气球a开始到气球b依次给每个气球涂一次颜色。但是N次以后lele已经忘记了第I个气球已经涂过几次颜色了，你能帮他算出每个气球被涂过几次颜色吗？
 

#### Input
每个测试实例第一行为一个整数N,(N <= 100000).接下来的N行，每行包括2个整数a b(1 <= a <= b <= N)。
当N = 0，输入结束。


#### Output
每个测试实例输出一行，包括N个整数，第I个数代表第I个气球总共被涂色的次数。

  
#### Sample Input
3  
1 1  
2 2  
3 3  
3  
1 1  
1 2  
1 3  
0  
  
#### Sample Output
1 1 1  
3 2 1

线段数解法：

	import java.util.Scanner;
	
	class Node{
		int left;
		int right;
		int count;
		Node(int left, int right, int count){
			this.left = left;
			this.right = right;
			this.count = count;
		}
	}
	
	public class HDU1556 {
		
		public static Node[] arr = new Node[3 * 100005];
	
		public static void build(Node[] arr, int root, int begin, int end){
			arr[root] = new Node(begin, end, 0);
			if(begin == end)
				return ;
			int mid = (begin + end) / 2;
			build(arr, root * 2, begin, mid);
			build(arr, root * 2 + 1, mid + 1, end);
		}
		
		public static void update(Node[] arr, int root, int begin, int end){
			if(arr[root].left == begin && arr[root].right == end){
				arr[root].count ++;
				return;
			}
			int mid = (arr[root].left + arr[root].right) / 2;
			if(end <= mid){
				update(arr, root * 2, begin, end);
			}else if(begin > mid){
				update(arr, root * 2 + 1, begin, end);
			}else{
				update(arr, root * 2, begin, mid);
				update(arr, root * 2 + 1, mid + 1, end);
			}
		}
		
		public static void toSum(Node[] arr, int root, int[] sum){
			for(int i = arr[root].left; i <= arr[root].right; i++){
				sum[i] += arr[root].count;
			}
			if(arr[root].left == arr[root].right)
				return ;
			toSum(arr, root * 2, sum);
			toSum(arr, root * 2 + 1, sum);
		}
		
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			Scanner sc = new Scanner(System.in);
			int n;
			while(true){
				n = sc.nextInt();
				if(n == 0)
					break;
	
				build(arr, 1, 1, n);
				for(int i = 0; i < n; i++){
					int a = sc.nextInt();
					int b = sc.nextInt();
					update(arr, 1, a, b);
				}
				int[] sum = new int[n + 1];
				toSum(arr, 1, sum);
				for(int i = 1; i < n; i ++)
					System.out.print(sum[i] + " ");
				System.out.println(sum[n]);
			}
		}
	
	}



树状数组解法：

	import java.util.Scanner;
	
	public class HDU1556 {
		
		private static int n; 
		private static int[] c;
		
		public static int lowbit(int x){
			return x & -(x);
		}
		
		public static void update(int x, int p){
			while(x <= n){
				c[x] += p;
				x += lowbit(x);
			}
		}
		
		public static int sum(int k){
			int ans = 0;
			while(k > 0){
				ans += c[k];
				k -= lowbit(k);
			}
			return ans;
		}
		
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			Scanner sc = new Scanner(System.in);
			while(sc.hasNext()){
				n = sc.nextInt();
				if(n == 0)
					break;
				c = new int[n+1];
				int a, b;
				for(int i = 1; i <= n; i ++){
					a = sc.nextInt();
					b = sc.nextInt();
					update(a, 1);
					update(b + 1, -1);
				}
				for(int i = 1; i < n; i++){
					System.out.print(sum(i) + " ");
				}
				System.out.println(sum(n));
			}
		}
	
	}

树状数组有4种经典模型，1、PUIQ模型；2、IUPQ模型；3、逆序模型；4、二分模型。参见[夜深人静写算法（3）：树状数组](http://blog.jobbole.com/96430/),此题运用的是IUPQ模型，<font color = "red">更新区间，单点求值</font>。IUPQ模型是PUIQ模型的转化。  
  
博主-IAMACER-做了如下解释：  
>我们假设sigma(r，i)表示r数组的前i项和，调用一次的复杂度是log2(i)。
>设原数组是a[n]，差分数组c[n]，c[i]=a[i]-a[i-1]，那么明显地a[i]=sigma(c,i)。  
>如果想要修改a[i]到a\[j](比如+v)，只需令c[i]+=v,c[j+1]-=v。

其中第三行解释的不是很清楚，我做一下补充。  
修改前：  
c[i] = a[i] - a[i-1]  
c[i+1] = a[i+1] - a[i]  
c[i+2] = a[i+2] - a[i+1]  
.....  
c[j] = a[j] - a[j-1]  
c[j + 1] = a[j + 1] - a[j]  
  
修改后：  
c[i] + v = (a[i] + v) - a[i-1]  
c[i+1] = (a[i+1] + v) - (a[i] + v)  
c[i+2] = (a[i+2] + v) - (a[i+1] + v)  
.....  
c[j] = (a[j] + v) - (a[j-1] - v)  
c[j+1] - v = a[j+1] - (a[j] + v)
  
***
参考资料：  
[http://blog.jobbole.com/96430/](http://blog.jobbole.com/96430/)  
[http://blog.csdn.net/fsahfgsadhsakndas/article/details/52650026](http://blog.csdn.net/fsahfgsadhsakndas/article/details/52650026)  
[http://blog.csdn.net/b735098742/article/details/52198579](http://blog.csdn.net/b735098742/article/details/52198579)

---
layout: post
title:  "数组中只出现一次的数字（位运算）"
categories: 算法
---
### [数组中只出现一次的数字](https://www.nowcoder.com/practice/e02fdb54d7524710a7d664d082bb7811?tpId=13&tqId=11193&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)
### 题目描述
一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。

比较容易想到的解法：遍历数组，用HashMap保存每个数出现的次数。虽然AC了但是不够Cool。

	public void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        for(int i = 0; i < array.length; i++) {
        	if(map.containsKey(array[i])) {
        		map.put(array[i], map.get(array[i]) + 1);
        	}else {
        		map.put(array[i], 1);
        	}
        }
        boolean isFirst = true;
        for(Map.Entry<Integer, Integer> entry : map.entrySet()) {
        	if(entry.getValue() == 1) {
        		if(isFirst) {
	        		num1[0] = entry.getKey();
	        		isFirst = false;
        		}
        		if(!isFirst) {
        			num2[0] = entry.getKey();
        		}
        	}
        }
    }
  
  
《剑指Offer》上的解法，利用异或运算的性质，两个相同的数进行异或运算结果为0。如果数组中出现一次的数字只有一个，那对整个数组进行异或运算的结果就是它。当出现一次的数字有两个时，问题变得不好处理。  
实际上是先将数组分成两组，每组中只包含一个只出现一个的数字，其他都是出现两次的次数。怎样将数组分成这样的两组呢？对整个数组进行异或运算后结果为resultExclusiveOR=num1[0]^num2[0]。找到resultExclusiveOR二进制位数低位起第一个为1的位数indexOf1。这是num1[0]与num2[0]的不同点。按照第indexOf1是否为1，把array[]分成两组，这样每组中就只用一个只出现一次的数字。

	public void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {
		if(array.length == 0)
			return;
		int resultExclusiveOR = 0;
		for (int i = 0; i < array.length; i++) {
			resultExclusiveOR ^= array[i];
		}
		int indexOf1 = FindFirstBitIs1(resultExclusiveOR);
		for (int i = 0 ; i < array.length; i++) {
			if (IsBit1(array[i], indexOf1))
				num1[0] ^= array[i];
			else
				num2[0] ^= array[i];
		}
	}
	
	public int FindFirstBitIs1(int num) {
		int indexBit = 0;
		while(num != 0 && ((num & 1) == 0)) {
			num = num >> 1;
			indexBit ++;
		}
		return indexBit;
	}
	
	public boolean IsBit1(int num, int index) {
		num = num >> index;
		return (num & 1) == 0 ? true : false;
	}

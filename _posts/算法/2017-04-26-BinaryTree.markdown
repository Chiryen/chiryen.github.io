---
layout: post
title:  "二叉树操作"
categories: 算法
---

	//二叉树结构
	public class TreeNode {
		int val = 0;
	    TreeNode left = null;
	    TreeNode right = null;
	    public TreeNode(int val) {
	        this.val = val;
	    }
	}


### 二叉树深度
	{% highlight java %}
	public int TreeDepth(TreeNode root) {
	    if(root == null)
	    	return 0;
	    int ld = TreeDepth(root.left);
	    int rd = TreeDepth(root.right);
	    return Math.max(ld, rd) + 1;
	}
	{% endhighlight %}


### 平衡二叉树
	public class Solution {
	    
	    private class Holder {
			int depth;
		}
		
		public boolean IsBalanced(TreeNode root, Holder h) {
			if(root == null) {
				h.depth = 0;
				return true;
			}
			Holder left = new Holder();
			Holder right = new Holder();
	        if(IsBalanced(root.left, left) && IsBalanced(root.right, right)) {
	        	h.depth = Math.max(left.depth, right.depth) + 1;
	        	int diff = Math.abs(left.depth - right.depth);
	        	if(diff > 1) {
	        		return false;
	        	}else {
	        		return true;
	        	}
	        }else {
	        	return false;
	        }
		}
		
		public boolean IsBalanced_Solution(TreeNode root) {
			return IsBalanced(root, new Holder());
	    }
	}
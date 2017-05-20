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
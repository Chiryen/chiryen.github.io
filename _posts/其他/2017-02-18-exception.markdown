---
layout: post
title:  "异常处理"
date:   2017-02-18 11:45:41 +0800
categories: 其他
---

# C++ 异常处理
项目中平台需要调用qt的plugin,我最初使用getenv()获取环境变量中的qt5路径,代码如下：

	char* qt_path = getenv("QT5_DIR");
	strcat(qt_path, "\\plugins");
	QCoreApplication::addLibraryPath(qt_path);

后来平台软件迁移到另一台机器，软件运行时崩溃。后来发现是应用机器上没有装qt,更没有环境变量QT5_DIR,坑啊...
根本原因还是自己代码写的不鲁棒，以后要写代码注意异常处理。修改代码：

	char* qt_path = getenv("QT5_DIR_");
	if(qt_path)
	{
		strcat(qt_path, "\\plugins");
		QCoreApplication::addLibraryPath(qt_path);
	}
	else
	{
		cout<<"QT5_DIR not exist"<<endl;
	}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

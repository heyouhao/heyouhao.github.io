---
layout: post
title: eclipse 标记任务
date: 2019-08-01 13:25:51
comments: true
tags: 
  - eclipse
---
### eclipse Task Tags:

**TODO** -用来提醒该标识处的代码有待返回继续编写、更新或者添加。该标签通常在注释块的源文件顶部。
**FIXME** -该标签用来提醒你代码中存在稍后某个时间需要修改的部分
**XXX** -需要改进的功能
<!--more-->

### 自定义任务标记：（如：HEYH）
window-->preferences-->java-->compiler-->tags
![eclipse添加标记](http://www.heyouhao.cn/images/7902846489382879.jpg)

eclipse 项目中添加HEYH示例：

```java
public static boolean isPositiveNum(String numStr){   
	String pattern = "^\\+?[1-9][0-9]*$";  
	// HEYH 测试标记
	Pattern p = Pattern.compile(pattern);  
	Matcher m = p.matcher(numStr);  
	return m.matches();  
} 
```

单击 window — show View — Tasks 打开任务视图,显示

![eclipse添加标记2](http://www.heyouhao.cn/images/7903270230596121.jpg)
### 管理任务
可以右键添加(Add Task)、删除(Remove Task)
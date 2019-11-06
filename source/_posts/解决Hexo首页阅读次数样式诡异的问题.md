---
title: 解决Hexo首页阅读次数样式诡异的问题
date: 2018-11-16 10:19:43
tags: [hexo]
categories: 博客
---
## 前言

  今晚在完善阅读统计的功能是时候发现一个很诡异的问题，我们一起来探讨一下。上一篇文章大概有阐述阅读统计功能的搭建，所有这里我就不多说了。  
  利用leancloud接完统计功能后，当前这个hexo版本我会发现首页的阅读数量的样式有点诡异，大致如下：  
  两种情况： 1：*阅读次数::9:9*   :2：*阅读次数::99*   
  实际上这个时候我们的阅读量只有9而已，正常来说应该这样显示：  *阅读次数：9*
  于是我忍不住摁住F12一探究竟,大致可以知道阅读次数那一块的元素是js直接赋值的，所以我就去找了一下阅读统计的部分的相关js，我比较粗暴，直接定位全局搜索一波 *leancloud_visitors*  相关的文件
  这是我们可以找到一个路径为 *themes\next\layout\_third-party\analytics\lean-analytics.swig* 的文件，打开发现就是这部分js处理leancloud的阅读数量统计。



 <!-- more -->

### 贴一波阅读次数统计的代码(LeanCloud提供支持)

  ``` JavaScript 看到function showTime 我相信你已经很明白了
  {% if theme.leancloud_visitors.enable %}
  
    {# custom analytics part create by xiamo #}
   <script src="https://cdn1.lncld.net/static/js/av-core-mini-0.6.1.js"></script>
   <script>AV.initialize("{{theme.leancloud_visitors.app_id}}", "{{theme.leancloud_visitors.app_key}}");</script>
   <script>
   function showTime(Counter) {
      console.warn('~欢迎光临我的博客 Email:binzhizhu@gmail.com ')
   	var query = new AV.Query(Counter);
   	$(".leancloud_visitors").each(function() {
   		var url = $(this).attr("id").trim();
   		query.equalTo("url", url);
   		query.find({
   			success: function(results) {
                  console.warn('--这里是阅读统计代码-- by --leancloud--')
   				if (results.length == 0) {
   					var content = $(document.getElementById(url)).text() + ': 0';
   					$(document.getElementById(url)).text(content);
   					return;
   				}
   				for (var i = 0; i < results.length; i++) {
   					var object = results[i];
   					var content = $(document.getElementById(url)).text() + object.attributes.time;
   					$(document.getElementById(url)).text(content);
   				}
   			},
   			error: function(object, error) {
   				console.log("Error: " + error.code + " " + error.message);
   			}
   		});
  
   	});
   }
  
  ```

  上述代码中的 for循环，看起来是没任何问题。但我hexo g -d 到GitHubPage 的时候，我再for循环里面console.log('obj',object),打印了一下当前文章的所有属性，发现同一篇文章会打印两次，所有才会出现上面所说的 *阅读次数::99*的情况。相当于第一次循环time是9，循环了两次拼接了起来于是变成*阅读次数::99*  
  所有我 直接在for循环的最后加了一句  


 ```
  return;
  
 ```

 ### 真.阅读统计代码段

  这么做实际上很简单，类似于做一层兼容了，确保不会重复循环，修复了之后发现真的没有问题了，阅读次数显示正常了，美滋滋，这也是一种收获。顺便也贴一下其他阅读统计事件的代码片段吧： 
   ``` 这里是统计部分 
   function addCount(Counter) {
   	var Counter = AV.Object.extend("Counter");
   	url = $(".leancloud_visitors").attr('id').trim();
   	title = $(".leancloud_visitors").attr('data-flag-title').trim();
   	var query = new AV.Query(Counter);
   	query.equalTo("url", url);
   	query.find({
   		success: function(results) {
   			if (results.length > 0) {
   				var counter = results[0];
   				counter.fetchWhenSave(true);
   				counter.increment("time");
   				counter.save(null, {
   					success: function(counter) {
   						var content = $(document.getElementById(url)).text() + ': ' + counter.get('time');
   						$(document.getElementById(url)).text(content);
   					},
   					error: function(counter, error) {
   						console.log('Failed to save Visitor num, with error message: ' + error.message);
   					}
   				});
   			} else {
   				var newcounter = new Counter();
   				newcounter.set("title", title);
   				newcounter.set("url", url);
   				newcounter.set("time", 1);
   				newcounter.save(null, {
   					success: function(newcounter) {
   					    console.log("newcounter.get('time')="+newcounter.get('time'));
   						var content = $(document.getElementById(url)).text() + ': ' + newcounter.get('time');
   						$(document.getElementById(url)).text(content);
   					},
   					error: function(newcounter, error) {
   						console.log('Failed to create');
   					}
   				});
   			}
   		},
   		error: function(error) {
   			console.log('Error:' + error.code + " " + error.message);
   		}
   	});
   }
   $(function() {
   	var Counter = AV.Object.extend("Counter");
   	if ($('.leancloud_visitors').length == 1) {
   		addCount(Counter);
   	} else if ($('.post-title-link').length > 1) {
   		showTime(Counter);
   	}
   });
   </script>
  {% endif %}

   ```

 这篇博客写得我有点疲惫了，已经深夜十分了，准备入睡啦，明天还要起来搬砖呢，晚安各位，希望能够帮助到大家吧，我也是刚学习Hexo自己搭建博客。  
 但有趣的一点是我们可以在这些开源的框架或者资源里面肆意的玩弄代码，hexo其实就是提供给大家开源开发的，看着文档接服务就好了，一大推的第三方服务已经Api。


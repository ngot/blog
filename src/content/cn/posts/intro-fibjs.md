---
title: "认识 fibjs"
date: 2014-11-02
---

# 认识fibjs

fibjs开源至今差不多半年了，越来越多的人知道fibjs。响马在前些日子的[南京源创会](http://city.oschina.net/nanjing/event/172750)上做的fibjs分享，更是让fibjs推广进入了一个高潮，就连[外国友人](http://baoz.cn/498330)都开始密切关注fibjs的动态。下面，我们就来探讨一下fibjs的一些问题。
### fibjs是什么？
- fibjs不是前端开发框架，不同于Jquery，Angular等运行在浏览器的JS框架，fibjs运行在服务端。
- fibjs不是Node.js的一个包，和NPM里面的fiber扩展包也没有关系。
- fibjs是基于协程和V8，运用C++语言开发的JS运行平台，和Node.js一样，都是服务端JS环境。
- fibjs由[西祠胡同](http://xici.net)创始人，[孢子社区](http://baoz.cn)创始人，[响马](http://weibo.com/p/1005052041028560/home?from=page_100505&mod=TAB#place)历时两年多开发完成。
### 又一个轮子？

很多人会问，既然已经有了Node.js，为什么还要再造fibjs这个大轮子？难道只是为了造轮子而造轮子吗？

其实，实际原因不是这样。要说fibjs的诞生，不得不说说[孢子社区](http://baoz.cn)的开发史。

在早期阶段，孢子社区的后端是运用[响马](http://weibo.com/p/1005052041028560/home?from=page_100505&mod=TAB#place)开发的VBS运行环境开发的，后来考虑到前后端代码复用，方便招聘开发人员等原因，决定后端转向JS平台。当时，不选择Node.js的原因是认为异步开发模式不是一个适合大规模部署的方式，会给开发和维护带来很大问题。

既然，Node.js不是一个我们认为的好的选择，那就自己造轮子吧。最初的技术选型(详情参见[点我](http://baoz.cn/460567))考虑了，v8，JavaScriptCore 和 SpiderMonkey三个JS引擎。

最终选择了 v8 作为基础核心。原因是：
- 支持多线程重入，协程的堆栈本质是线程，所以要支持协程必须要支持多堆栈重入；
- 不支持多线程并发，虽然 isolate 支持多堆栈现场，但是 isolate 内部却为无锁环境，因此不能接受多线程同时运行，必须在一个线程退出后，另一线程才可进入；

在选择了V8后(不是Node.js用了V8，咱就要用V8，选择什么都是有原因)，再开发协程环境并和V8结合工作，再补上其他基础模块，就是fibjs了。

总的来说，造轮子最初目的是为了自身满足需求。
### fibjs名字由来？

fib是fiber的简称，fiber就是纤程的意思。JS就是Javascript语言。连起来就是用fiber技术构建起来的JS平台，简单而又直白。
### fibjs logo含义
- [![fibjs](https://raw.githubusercontent.com/ngot/blog/master/imgs/fibjs.jpg)](http://baoz.cn/fibjs)
- 去除回调，去除层层的 }}}}}}}}}}}}}}}}}}}}}}}}};
- Javscript
- 跨平台，支持Mac OSX, Linux, FreeBSD, Windows
### fibjs特点
- 同步编写异步代码
  
  node.js的回调写法，肯定很多人见识过，层层回调简直就是项目的灾难。虽然，可以通过Asyc，Promise，Generator等手段，在形式上简化回调写法，但是本质上没有变，始终无法靠直觉写出简洁优美的代码。还是少废话，直接看代码。
  
  我们来看一个文件_异步_读取的例子：
  
  Node.js CallBack版本
  
  ```
      var fs = require("fs");
      fs.readFile('file', function(err, data) {
          if (err) throw err;
          console.log(data.toString());
      });
  
  ```
  
  采用CO库改进的Node.js版本
  
  ```
      var fs = require("fs");
      var co = require("./co");
  
      function read(file) {
          return function(fn) {
              fs.readFile(file, function(err, data) {
                  if (err) return fn(err);
                  fn(null, data);
              });
          }
      }
  
      co(function *() {
          var a = yield read('file');
          console.log(a.toString());
      })();
  
  ```
  
  fibjs版本
  
  ```
      var fs = require("fs");
      try {
          var file = fs.readFile('file');
          console.log(file);
      } catch (e) {
          console.log(e.number);
      }
  
  ```
  
  从上面的代码对比，可以看出，fibjs的同步写法非常简洁，而且可以利用try catch来捕获异常，而node.js必须依赖回调来处理异步，就算采用了Generator，在代码简洁和错误处理上Node.js还是没有fibjs来的简单明了。
- 高性能，整体比Node.js快接近8倍
  
  相比较Node.js，fibjs具有明显的性能优势，测试案例：
  - [web服务对比测试](http://baoz.cn/494881)
  - [fibjs VS node.js With Mongodb](http://baoz.cn/499353)
- 前后统一语言
  这个优点还是非常诱人的，前端和后端不需要跨语言开发，许多代码库可以共享，更有利于开发人员往全栈方向发展。
### fibjs发展史
- 2012年2月，项目启动
- 2012年9月，开始在孢子社区生产环境试运行
- 2013年初，向外公开fibjs信息
- 2014年5月，项目推送Github，彻底开源
- 2014年10月，南京源创会，首次开讲，引起业界轰动
- 还在继续...
### 应用案例
- [孢子社区](http://baoz.cn)
- [咕咚运动](http://www.codoon.com/)
- [舜飞科技](http://www.sunteng.com/)
- [会鸽](http://www.eventdove.com/)
- 以及其他，我还不知道的公司
### 如何安装fibjs
- 如果你是Mac OSX用户，现在可以直接
  
  ```
      brew install fibjs
  ```
  
  感谢[@艾斯昆](http://weibo.com/aisk?nick=艾斯昆)帮忙提交到[Homebrew](http://brew.sh)
- 如果你是有NPM环境的Linux或者Mac用户，可以
  
  ```
      npm install fibjs -g
  ```
  
  感谢[笔者](http://weibo.com/1751144103/profile?rightmod=1&wvr=6&mod=personinfo)(也就是写这篇文字的家伙，并且他还偷懒不支持windows)的贡献
- 如果以上两点条件都不具备或者安装不成功，还可以直接点击下面链接下载笔者编译好的可执行程序来体验。
  - [OSX](http://f.ngot.me/fibjs/osx/fibjs)
  - [Linux x64](http://f.ngot.me/fibjs/linux/x64/fibjs)
  - [Linux x86](http://f.ngot.me/fibjs/linux/x86/fibjs)
  - [windows x64](http://f.ngot.me/fibjs/windows/x64/fibjs.exe)
  - [windows x86](http://f.ngot.me/fibjs/windows/x86/fibjs.exe)
  
  注意OSX和Linux用户，直接下载的文件不具有执行权限，执行：
  
  ``````
  ```
      chmod 777 fibjs
  ```
  就能够正常运行了。
  ``````
- 如果，以上三步都失败了(够悲催的)，那就到[fibjs主页](https://github.com/xicilion/fibjs)下载代码，手动编译。
### 文档翻译计划

因为fibjs在外国友人中得名气越来越大，英文文档的需求日益迫切，所以[响马](http://weibo.com/p/1005052041028560/home?from=page_100505&mod=TAB#place)开展了fibjs文档的翻译工作，文档项目地址：[https://github.com/xicilion/fibjs_docs](https://github.com/xicilion/fibjs_docs)，欢迎认领翻译。
### 参考资料
- [正式介绍fibjs](http://baoz.cn/460567)
- [http://fibjs.org](http://fibjs.org)
- [JavaScript 既是单线程又是异步的，请问这二者是否冲突，以及有什么区别？](http://baoz.cn/384147)
- [fibjs Logo 的原型](http://baoz.cn/369503)
- [关于fibjs 引入npm 的包](http://baoz.cn/451664)
- [fibjs添加模块载入检测机制](http://baoz.cn/458295)
- [为什么采用fiber而不用Generator](http://baoz.cn/458889)
- [fibjs异步IO](http://baoz.cn/384146)
- [源创会Slide](http://baoz.cn/498326)

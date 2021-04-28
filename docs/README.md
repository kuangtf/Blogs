<br>

<p align="center">
    <img width="280px" src="https://gitee.com/kuangtf/blogImage/raw/master/img/title.jpg" >
</p>

</br>
<br>


|          算法          |         操作系统          |        网络&nbsp;        |          数据库          |          Java          |          必备框架          |          系统设计           |          面向对象          |      &nbsp;工具       | 后记                   |
| :--------------------: | :-----------------------: | :----------------------: | :----------------------: | :--------------------: | :------------------------: | :-------------------------: | :------------------------: | :-------------------: | ---------------------- |
| [🤖 ](README?id=🤖-算法) | [🎮](README?id=🎮-操作系统) | [🎨  ](README?id=🎨-网络 ) | [📑 ](README?id=📑-数据库) | [🍵 ](README?id=🍵-Java) | [🔥 ](README?id=🔥-必备框架) | [👷 ](README?id=👷-系统设计 ) | [🦄 ](README?id=🦄-面向对象) | [🔨](README?id=🔨-工具) | [📞 ](README?id=📞-后记) |

</br>

---

💡 **「关于」**

- 🎓 博主渣渣一个，旨在用于记录学习过程中的所思所想，并力图构建一个完善的知识体系!

- 🙏 由于本人水平有限，仓库中的知识点来自本人原创、视频、书籍、博客等，非原创均已标明出处（或在参考资料中列出），如有遗漏或发现文章错误及排版问题，请提 issue 或 PR！

- 🔮 [Github 在线阅读](https://kuangtf.github.io/Blogs/#/README.md) | [Github 仓库地址](https://github.com/kuangtf/Blogs)

## 🤖 算法

#### 数据结构

- [红黑树](https://www.jianshu.com/p/e136ec79235c)
- [平衡二叉树](https://zhuanlan.zhihu.com/p/56066942)

#### 算法

- [Leetcode 题解](https://leetcode-cn.com/u/kuangtf/)
- [剑指 Offer 题解](https://leetcode-cn.com/problemset/lcof/)
- [有哪些散列算法](https://cloud.tencent.com/developer/article/1676797)
- 十大排序算法

## 🎮 操作系统

#### 操作系统

- 进程通信的方式
- 进程同步机制
- 进程调度算法有哪些
- 磁盘的寻道算法
- 孤儿进程和僵尸进程
- [什么是用户线程和守护线程](https://www.jianshu.com/p/b51da027bfe1)

#### Linux

- 常用的Linux命令
- Linux磁盘管理
- Linux文件与目录管理

## 🎨 网络 

- [计算机网络](https://blog.csdn.net/weixin_46156200/category_10731223.html?spm=1001.2014.3001.5482)
- [get与post请求的区别](https://www.huaweicloud.com/articles/5c532839eeb5e1cd4665f48b0a581eb3.html)
- [Tcp和udp区别](https://juejin.cn/post/6857707137797292046)
- 长连接和短连接
- [Cookie和session的区别？](https://www.cnblogs.com/ityouknow/p/10856177.html)
- 典型的HTTP攻击手段
- 粘包和拆包
- ping的原理
- 三次握手和四次挥手
- HTTPS协议
- TCP保证可靠传输的机制
- 输入一个URL回车发生了什么

## 📑 数据库

### MySQL

- [Mysql底层数据结构，b树和b+树的区别 ，聚簇索引和非聚簇索引的区别](https://www.huaweicloud.com/articles/ce5c6f4d1d60cf7f82f2db6e215555d3.html)
- [myISAM和innodb的区别](https://www.huaweicloud.com/articles/bb2e4d7dcc7d849df919a88289c9d74c.html)
- 索引在哪种情况下会失效
- [最左原则?联合索引?](https://blog.csdn.net/u013568373/article/details/93891531#:~:text=%E6%9C%80%E5%B7%A6%E5%8C%B9%E9%85%8D%E5%8E%9F%E5%88%99%E7%9A%84%E6%88%90%E5%9B%A0%20MySQL,%E5%BB%BA%E7%AB%8B%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95%E7%9A%84%E8%A7%84%E5%88%99%E6%98%AF%E8%BF%99%E6%A0%B7%E7%9A%84%EF%BC%8C%E5%AE%83%E4%BC%9A%20%E9%A6%96%E5%85%88%E6%A0%B9%E6%8D%AE%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95%E4%B8%AD%E6%9C%80%E5%B7%A6%E8%BE%B9%E7%9A%84%E3%80%81%E4%B9%9F%E5%B0%B1%E6%98%AF%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%AD%97%E6%AE%B5%E8%BF%9B%E8%A1%8C%E6%8E%92%E5%BA%8F%EF%BC%8C%E5%9C%A8%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%AD%97%E6%AE%B5%E6%8E%92%E5%BA%8F%E7%9A%84%E5%9F%BA%E7%A1%80%E4%B8%8A%EF%BC%8C%E5%86%8D%E5%AF%B9%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95%E4%B8%AD%E5%90%8E%E9%9D%A2%E7%9A%84%E7%AC%AC%E4%BA%8C%E4%B8%AA%E5%AD%97%E6%AE%B5%E8%BF%9B%E8%A1%8C%E6%8E%92%E5%BA%8F%EF%BC%8C%E4%BE%9D%E6%AD%A4%E7%B1%BB%E6%8E%A8%20%E3%80%82)
- [分库解决了什么问题?分表解决了什么问题?](https://segmentfault.com/a/1190000023914691)
- 可以说下数据库范式吗？
- 数据库锁有哪些？
- 数据库的乐观锁和悲观锁?
- [MVVC](https://juejin.cn/post/6871046354018238472)
- 分布式事务
- [讲讲数据库表怎么设计的](https://blog.csdn.net/kw023781/article/details/103002794)
- 数据库的主从复制和保证一致性
- 事务的两阶段提交
- MySQL慢查询
- 什么是sql注入，如何防止sql注入？
- sql优化和索引优化

### Reids

- Redis底层数据结构
- 淘汰过期键的策略
- 缓存穿透 缓存雪崩 如何避免 
- redis单线程模型
- 为什么 Redis 这么快
- 布隆过滤器
- 保证数据库和缓存的一致性
- Redis如何实现分布式锁，及其原理
- [redis内存模型](https://zhuanlan.zhihu.com/p/293040974)

## 🍵 Java

#### Java 基础

- [解决hash冲突有哪些方法？](https://zhuanlan.zhihu.com/p/29520044)
- [动态代理和反射](https://bbs.huaweicloud.com/blogs/195516)
- [CGLIB和JDK代理](https://www.cnblogs.com/carpenterlee/p/8241042.html)

#### 集合

- HashMap底层原理
- ConcurrentHash底层原理
- ArrayList 和 LinkedList有什么区别

#### 并发

- [ThreadLocal](https://zhuanlan.zhihu.com/p/34406557)
- [不得不说的Java“锁”事](https://tech.meituan.com/2018/11/15/java-lock.html)
- [AQS的实现原理](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)
- [synchronized](https://zhuanlan.zhihu.com/p/29866981)
- ReentrantLock
- volatile
- [java线程池](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
- [CAS](https://www.jianshu.com/p/eac466494477) 
- conditions

#### JVM

- 常见的垃圾收集算法
- 类加载过程
- 运行时数据区

## 🔥 必备框架

#### Spring

- spring 事务传播机制

#### SpringMVC

- SpringMVC的执行流程

#### Mybatis

- mybatis缓存

#### Spring Boot

- Spring Boot 启动流程
- 自动装配原理

## 👷 系统设计 

- [系统设计基础]()
- [分布式]()
- [集群]()
- [攻击技术]()
- [缓存]()
- [消息队列]()

## 🦄 面向对象

- [面向对象思想]()
- [设计模式](https://blog.csdn.net/weixin_46156200/category_10769699.html?spm=1001.2014.3001.5482)

## 🔨 工具 

- [Git]()
- [Docker]()

## 📞 后记

#### 公众号

- 文章会第一时间在公众号推送哟

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/gong.jpg" style="zoom:30%;" />

#### 联系我

- 有什么问题也可以添加我的微信，记得备注来意：格式 （学校或公司 - 姓名或昵称 - 来意）

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/wei.jpg" style="zoom:30%;" />

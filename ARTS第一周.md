2019/3/18~3/24 第一周ARTS

> 每周一个Algorithm，Review一篇英文文章，Tips 学习一个技术技巧，Share分享一篇有观点和思考的技术文章。

### Algorithm:	算法题

https://leetcode.com/problems/median-of-two-sorted-arrays/

4. > Median of Two Sorted Arrays
   >
   > There are two sorted arrays **nums1** and **nums2** of size m and n respectively.
   >
   > Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).
   >
   > You may assume **nums1** and **nums2** cannot be both empty.
   >
   > 核心算法，TopK，递归切割小的部分，移动开始位置和更新K， K值变化为K-K/2（切去的的数都是偏小的）

```
public static double findK2(int[] a, int aS, int[] b, int bS, int k) {
    if (a.length <= aS) return b[bS + k - 1];
    if (b.length <= bS) return a[aS + k - 1];

    if (k == 1) return Math.min(a[aS], b[bS]);

    int midA = Integer.MAX_VALUE;
    int midB = Integer.MAX_VALUE;

    if (a.length > aS + k / 2 - 1) {
        midA = a[aS + k / 2 - 1];
    }
    if (b.length > bS + k / 2 - 1) {
        midB = b[bS + k / 2 - 1];
    }

    return midA > midB ?
            findK(a, aS, b, bS + k / 2, k - k / 2) : findK(a, aS + k / 2, b, bS, k - k / 2);
}

```



### Review:	阅读英文文章

​	Kubernetes in Action 1.1 

 1.1 Understanding the need for a system like Kubernetes

开发与Ops独立工作，单体大模块应用经常因为一些小改动而整体部署。单体大应用被拆分成微服务架构，独立部署和开发。运维工作会越来越难。

K8s将集群资源抽象成一个大平台，开发和运维不需要知道应用部署在那台机器，K8s管理期望的部署和实际部署的调协工作。

**垂直扩展**：提升单机处理能力。垂直扩展的方式又有两种：

（1）增强单机硬件性能，例如：增加CPU核数如32核，升级更好的网卡如万兆，升级更好的硬盘如SSD，扩充硬盘容量如2T，扩充系统内存如128G；

（2）提升单机架构性能，例如：使用Cache来减少IO次数，使用异步来增加单服务吞吐量，使用无锁数据结构来减少响应时间；

**水平扩展**：只要增加服务器数量，就能线性扩充系统性能

1.1.1 Moving from monolithic apps to microservices

每个微服务依赖环境和lib独立，要支持先后兼容a backward-compatible way

> Microservices also bring other problems, such as making it hard to debug and trace
>
> execution calls, because they span multiple processes and machines. Luckily, these
>
> problems are now being addressed with distributed tracing systems such as Zipkin.

1.1.2 Providing a consistent environment to applications

方便统一开发机和线上机环境（网络，资源，文件路径等等）一致

1.1.3 Moving to continuous delivery: DevOps and NoOps 持续交付，有了K8s开发自运维，测试，部署。No need Ops

### Tips:	知识点，技术技巧

**优化sql，使用sql函数**可以很好的解决一些复杂的业务场景。**举个栗子：**

三张表 rt_topic（kafka的topic表），rt_task（抽数任务表），topic_task_relation（关联表），一个topic对应多个抽数task，

​	task表有 status状态int  3：running，4：stoped； 心跳时间date，心跳超过五分钟为失去心跳

​	topic下的所有的task都是running且心跳都在五分钟内的，topic状态才是running，3

​	topic下的所有的task都是stoped，topic状态才是stoped

​	其它情况都是exception情况

​	统计每一个topic的状态：

​	思路：

​	子查询：

​	按照topic分组，统计topic下task状态 min（status） 和 max (status)，

​	然后max（heartbeat >  SUBDATE( now( ), INTERVAL 5 MINUTE )）:返回 0,1 如果存在1就代表有失去心跳的

​	外层查询：case when 根据status 和 心跳 判断返回 topic状态

		SELECT topic_status.*,
	    CASE
	    WHEN min_status = 3  AND max_status = 3  AND max_heartbeat = 0 THEN 1
	    WHEN min_status = 4  AND max_status = 4 THEN 2 ELSE 3
	    END AS fregata_topic_status
	    FROM
	    (
					SELECT
					rtc.*,
					min( rtk.task_status ) AS min_status,
					max( rtk.task_status ) AS max_status,
					max( rtk.heartbeat > SUBDATE( now( ), INTERVAL 5 MINUTE ) ) max_heartbeat
					FROM
					rt_topic rtc
					INNER JOIN topic_task_relation rtr ON rtr.rt_topic_id = rtc.id
					INNER JOIN rt_task rtk ON rtk.id = rtr.rt_task_id
					GROUP BY
					rtc.fregata_topic
				) topic_status
### Share:	分享

1.  **工作中文档（wiki）编写，语言正式规范，步骤清晰，相关人员要准确**   今天在公司wiki的个人空间写了一篇cf，领导说像写小说，不适合分享给别人。哈哈 主要我语言太皮了
2.  **安排的需求千万不要着急下手**，不要觉得功能可用就算开发完成了，要考虑性能优化，和异常情况。比如sql优化，数据库事务处理。 这周写了一个复杂的sql，结果领导让我优化，我着急把初版的sql上了测试。后期优化又要重新改了，而且浪费了思考优化sql的时间。


> 本文参考:
>
> [分布式任务调度框架XXL-JOB解析（一）概述及搭建](https://juejin.im/post/5dd133005188252ad04c4f94 )
>
> [分布式任务调度框架XXL-JOB解析（二）注册心跳](https://juejin.im/post/5dd1f1a2f265da0bd71ff2be )
>
> [分布式任务调度框架XXL-JOB解析（三）任务调度](https://juejin.im/post/5dd235f76fb9a0201b46a8d6 )

# xxl-job使用总结

xxl-job的老版本是依赖quartz的定时任务触发，在v2.1.0版本开始，移除了quartz依赖：一方面是为了精简系统降低冗余依赖，另一方面是为了提升系统的可控性和稳定性。



## 1.quartz的不足

quartz和xxl-job对比，什么时候选用quartz，什么时候选用xxl-job？-------待思考



## 2.xxl-job的设计思想

将调度行为抽象形成调度中心公共平台，而平台自身并不承担业务逻辑，**调度中心**负责发起调度请求。

将任务抽象成分散的JobHandler，交由**执行器**统一管理，执行器负责接收调度请求并执行对应的JobHandler中业务逻辑。

因此，我们可以看出，调度和任务两部分可以相互解耦，提供系统整体稳定性和扩展性。
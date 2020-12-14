---
title: RabbitMQ 持久化
date: 2019-04-15
updated: 2019-04-15
tags: [RabbitMQ]
categories: [RabbitMQ ]
---

RabbitMQ 持久化分为三部分：交换器的持久化、队列的持久化和消息的持久化

# 交换器持久化

交换器的持久化是通过在声明队列是将 durable 参数置为 true 实现的，如果交换器不设置持久化，那么在 RabbitM 务重启之后，相关的交换器元数据会丢失，不过消息不会丢失，只是不能将消息发送到这个交换器中了。

<!-- more -->

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/loGuTiCye14rHIYp.png!thumbnail)

Features那列 大写的D说明 durable 参数置为 true，交换器进行了持久化

# 队列持久化

队列的持久化是通过在声明队列时将 durable 参数置为 true 实现的，如果队列不设置持久化，那么在 RabbitMQ 服务重启之后，相关队列的元数据会丢失，此时数据也会丢失。正所谓"皮之不存，毛将焉附"。

队列的持久化能保证其本身的元数据不会因异常情况而丢失，但是并不能保证内部所存储的消息不会丢失。

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/vnWz8Q66BKIJpcDg.png!thumbnail)

Features那列大写的D说明 durable 参数置为 true，队列进行了持久化

# 消息的持久化

通过将消息的投递模式(BasicProperties 中的 deliveryMode 属性)设置为即可实现消息的持久化。

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/N4IzbcmRW6sJNUeP.png!thumbnail)

Persistent为1说明有一条持续化消息，持久消息同时也会保存在内存中

只有同时设置了队列和消息的持久化，当 RabbitMQ 服务重启之后，消息依旧存在

单单只设置队列持久化，重启之后消息会丢失；单单只设置消息的持久化，消息并不会被持久化，重启之后队列消失，继而消息也丢失。

可以将所有的消息都设直为持久化，但是这样会严重影响 RabbitMQ 的性能(随机)。对于可靠性不是那么高的消息可以不采用持久化处理以提高整体的吞吐量。在选择是否要将消息持久化时，需要在可靠性和吐吞量之间做一个权衡。

**即使将交换器、队列、消息都设置了持久化之后也不能百分之百保证数据不丢失**

- 问题1：


首先从消费者来说，如果在订阅消费队列时将 autoAck 参数设置为 true ，那么当消费者接收到相关消息之后，还没来得及处理就宕机了，这样也算数据丢失。

解决方案：

将autoAck 参数设置为 false 并进行手动确认

- 问题2：


在持久化的消息正确存入 RabbitMQ 之后，还需要有一段时间才能存入磁盘之中。 RabbitMQ 并不会为每条消息都进行同步存盘(调用内核的 fsync方法)的处理，可能仅仅保存到操作系统缓存之中而不是物理磁盘之中。如果在这段时间内RabbitMQ 服务节点发生了岩机、重启等异常情况，消息保存还没来得及落盘，那么这些消息将会丢失。

解决方案：

可以引入 RabbitMQ 镜像队列机制，相当于配置了副本，如果主节点 master 在此特殊时间内挂掉，可以自动切换到从节点 slave ), 这样有效地保证了高可用性，除非整个集群都挂掉。虽然这样也不能完全保证 RabbitMQ 消息不丢失，但是配置了镜像队列要比没有配置镜像队列的可靠性要高很多，在实际生产环境中的关键业务队列一般都会设置镜像队列

还可以在发送端引入事务机制或者发送方确认机制来保证消息己经正确地发送并存储至

RabbitMQ 中，前提还要保证在调用 channel.basicPublish 方法的时候交换器能够将消息

正确路由到相应的队列之中。


如果exchange和queue都是持久化的，那么它们之间的binding也是持久化的。如果exchange和queue两者之间有一个持久化，一个非持久化，就不允许建立绑定











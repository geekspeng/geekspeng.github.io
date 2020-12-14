---
title: keepalived 抢占IP的问题
date: 2019-09-01
updated: 2019-09-01
tags: [keepalived,高可用]
categories: [高可用]
---

在 master->backup 模式下，一旦主库宕掉， 虚拟IP会自动漂移到从库，当主库修复后，keepalived启动后，还会把虚拟IP抢过来，即使你设置nopreempt（不抢占）的方式抢占IP的动作也会发生

<!-- more -->

* 在 master->backup 模式下，一旦主库宕掉， 虚拟IP会自动漂移到从库，当主库修复后，keepalived启动后，还会把虚拟IP抢过来，即使你设置nopreempt（不抢占）的方式抢占IP的动作也会发生
* 在 backup->backup 模式下，关闭 VIP抢占模式，当主库宕掉后虚拟IP会自动漂移到从库上，当原主恢复之后重启keepalived服务，并不会抢占新主的虚拟IP， 即使是优先级高于从库的优先级别，也不会抢占 IP



示例

节点1

```
vrrp_instance VI_1 {
    state BACKUP  # 通过下面的priority来区分MASTER和BACKUP，也只有如此，底下的nopreempt才有效
    interface eth0@if49
    virtual_router_id 51
    priority 100
    advert_int 1
    nopreempt     #防止切换到从库后，主keepalived恢复后自动切换回主库
    authentication {
        auth_type PASS
        auth_pass 1111
    }    virtual_ipaddress {
        172.17.0.4/16
    }
}
```

节点2

```
vrrp_instance VI_1 {
    state BACKUP
    interface eth0@if51
    virtual_router_id 51
    priority 90
    advert_int 1
    authentication {
    auth_type PASS
    auth_pass 1111
    }
    virtual_ipaddress {
    172.17.0.4/16
    }
}
```


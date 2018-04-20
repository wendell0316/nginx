* #### Keepalived 介绍

  Keepalived是一个基于VRRP协议来实现的服务高可用方案，可以利用其来避免IP单点故障。

  > 文本将介绍其与nginx的结合使用，来达到高可用。
  >
  > > 在两台或多台服务器中配置相同的nginx配置。

* #### Keepalived 安装
 * 查看服务器是否已安装keepalived
 ```
 which keepalived
 ```
 * 未安装可通过以下指令安装
 ```
  yum install keepalived
  ```

* #### 配置文件模板

```
  ! Configuration File for keepalived

    global_defs {
       router_id nginx_master           # 标志本节点的名称，可以将两台服务器区分开
    }

    vrrp_script chk_nginx {             # 定义检查nginx的脚本
        script "curl --HEAD localhost"  # 查询nginx代理的服务是否还在启动
        interval 2                      # 每2秒检查一次
        weight -20                      # 失败一次优先级减少20
    }

    vrrp_instance VI_1 {
        state MASTER                    # 状态，主节点为master，备份节点为BACKUP
        interface eth0                  # 绑定VIP的网络接口，通过ifconfig查看自己的网络接口
        virtual_router_id 51            # 虚拟路由的ID号,两个节点设置必须一样
        mcast_src_ip 145.4.232.211      # 本机ip地址
        priority 100                    # 节点优先级，master比backup高
        advert_int 1                    # 组播信息发送时间间隔，两个节点必须设置一样，默认为1秒
        virtual_ipaddress {             # 虚拟IP，两个节点设置必须一样。可以设置多个，一行写一个
            145.4.232.171
        }

        track_script {
            chk_nginx                   #调用上面定义的检查nginx脚本
        }
    }
```

* #### Keepalived 启动及排错

 * 启动

   ```
  systemctl start keepalived
  ```
 * 查看状态
  ```
  systemctl status keepalived -l
  ```
 * 若出错可根据提示安装（因不同服务器可能出错情况不同，不一一列举），一般是缺少依赖
 * 修改后再启动即可(若仍无法正常启动可选择卸载重装)



Title: How to make HA GlusterFs using Keepalived
Date: 2017-08-02 01:01
Category: IT
Tags: Linux, HA, Gluster, Keepalived, distributedfs

Example tested on 2 CentOS7 VM

    ! Configuration File for keepalived
    
    vrrp_script check_glusterd {
      script       "/usr/sbin/pidof glusterd"
      interval 5
      fall 1
      rise 1
    !  timeout t
    !  weight w
    }
    
    global_defs {
    }
    
    vrrp_instance gfs01 {
    !    state MASTER
        interface eth0
        virtual_router_id 51
        priority 100
        advert_int 1
        nopreempt
        authentication {
            auth_type PASS
            auth_pass 1111
        }
        virtual_ipaddress {
            192.168.122.100/24
        }
    
        track_script {
           check_glusterd
        }
    }


Some links

* RedHat [loadbalancing](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Load_Balancer_Administration/ch-lvs-overview-VSA.html)
* Keepalived [User Guide](http://www.keepalived.org/pdf/UserGuide.pdf)
* Unicsolution [blog](http://blog.unicsolution.com/2015/01/kamailio-high-availability-with.html)


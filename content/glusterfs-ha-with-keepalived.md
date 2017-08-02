Title: How to make HA GlusterFs using Keepalived
Date: 2017-08-02 01:01
Category: HA,Gluster,Keepalived,distributedfs

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


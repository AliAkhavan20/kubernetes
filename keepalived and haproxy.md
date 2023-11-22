# Install a Multi-Master kubernetes with HA and Keepalived in ubuntu

## Setup Keepalived nodes

### Update and Install keepalived
```
apt update && apt install -y keepalived
```

### Create and Edit keepalived config
#### Noted that this configuration must be installed on every ha-machine with a slightly difference.
```
nano /etc/keepalived/keepalived.conf
```
```
global_defs {
  notification_email {
  }
  router_id LVS_DEVEL
  vrrp_skip_check_adv_addr
  vrrp_garp_interval 0
  vrrp_gna_interval 0
}

vrrp_script chk_haproxy {
  script "killall -0 haproxy"
  interval 2
  weight 2
}

vrrp_instance haproxy-vip {
  state BACKUP
  priority 100
  interface ens33                          # Change Network interface based on your configuration
  virtual_router_id 60
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 1111
  }
  unicast_src_ip 192.168.74.204            # putt The IP address of this machine
  unicast_peer {
    192.168.74.205                         # putt The IP address of peer machines
  }

  virtual_ipaddress {
    192.168.74.210/24                      # The VIP address
  }

  track_script {
    chk_haproxy
  }
}
```
#### For the interface field, you must provide your own network interface type.
#### Make sure you configure Keepalived on the other machines as well.

### Enable & start keepalived service
```
systemctl enable --now keepalived.service
```
```
systemctl start keepalived.service
```
## Setup haproxy nodes

### Update and Install haproxy
```
apt update && apt install -y haproxy
```
#### Configure haproxy after keepalived has been started successfully.
#### Run and edit haproxy.cfg on every HA node and the configuration will be same on each node.
#### Dont forget to change the ip server addresses.
```
nano /etc/haproxy/haproxy.cfg
```
```
global
    log /dev/log  local0 warning
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
   
   stats socket /var/lib/haproxy/stats
   
defaults
  log global
  option  httplog
  option  dontlognull
        timeout connect 5000
        timeout client 50000
        timeout server 50000
   
frontend kube-apiserver
  bind *:6443                                          # Dont put any ip address here
  mode tcp
  option tcplog
  default_backend kube-apiserver
   
backend kube-apiserver
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server kube-apiserver-1 172.16.0.4:6443 check 	# Replace the IP address with your own.
    server kube-apiserver-2 172.16.0.5:6443 check 	# Replace the IP address with your own.
    server kube-apiserver-3 172.16.0.6:6443 check 	# Replace the IP address with your own.
```

### Enable & Restart haproxy service
```
systemctl enable haproxy
```
```
systemctl restart haproxy
```

## Verify HA

#### Before you start to create your Kubernetes cluster, make sure you have tested the high availability.
#### Now you have 1 VP for 3 or as many nodes as you want.the IP will be shift to the alive nodes in case of failure.
#### You can simply stop ha-proxy service by "systemctl stop haproxy" in one node and see if the virtual ip address has been shifted to another node.




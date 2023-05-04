# DEVWKS-2440

## 1 - Login to dcloud:
#### a - Discover the topology
#### b - Connect to the anyconnect dcloud VPN
#### c - Connect to CML
#### d - Connect to Ubuntu server using cml-ui

## 2 - connect to ubuntu-Lisbon:
       cisco@inserthostnamehere:~$ ncs --status | grep status
#### a - login to nso and make sure there are no packages there:
      cisco@inserthostnamehere:/usr/bin$ ncs_cli -C
      cisco@ncs# show packages package 
        % No entries found.
      cisco@ncs# 
#### b - discover tailf-hcc directory:
      cisco@inserthostnamehere:~/devdays2023/tailf-hcc$ cd /home/cisco/devdays2023/tailf-hcc/
      cisco@inserthostnamehere:~/devdays2023/tailf-hcc$ ls
        README.signature
        cisco_x509_verify_release.py
        cisco_x509_verify_release.py3
        ncs-5.7.6-tailf-hcc-project-5.0.3
        ncs-5.7.6-tailf-hcc-project-5.0.3.signed.bin
        ncs-5.7.6-tailf-hcc-project-5.0.3.tar.gz
        ncs-5.7.6-tailf-hcc-project-5.0.3.tar.gz.signature
        tailf.cer
#### c - go to the actual tail-f package:
      cisco@inserthostnamehere:~/devdays2023/tailf-hcc$ cd ncs-5.7.6-tailf-hcc-project-5.0.3/packages/
      cisco@inserthostnamehere:~/devdays2023/tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages$ ls
        ncs-5.7.6-tailf-hcc-5.0.3.tar.gz  tailf-hcc
#### d - move this package to nso packages directory:
      cisco@inserthcd /var/opt/ncs/packages//tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages$ cd /var/opt/ncs/packages/
      cisco@inserthostnamehere:/var/opt/ncs/packages$ cp -r /home/cisco/devdays2023/tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages/tailf-hcc/ .
      cisco@inserthostnamehere:/var/opt/ncs/packages$ ls
        tailf-hcc

#### d - enter nso and do a package reload:
      cisco@inserthostnamehere:/usr/bin$ ncs_cli -C
      cisco@ncs# packages reload 

        >>> System upgrade is starting.
        >>> Sessions in configure mode must exit to operational mode.
        >>> No configuration changes can be performed until upgrade has completed.
        >>> System upgrade has completed successfully.
        reload-result {
            package tailf-hcc
            result true
        }
        cisco@ncs# 
        System message at 2023-05-04 18:08:08...
            Subsystem started: tailf_hcc_server
        cisco@ncs# 

#### d - exit nso:
        cisco@ncs# exit

## 3 - Install gobgp: 
#### a - Go to gobgp directory:
       cisco@inserthostnamehere:~/devdays2023/gobgp$ $ cd /home/cisco/devdays2023/gobgp 
       cisco@inserthostnamehere:~/devdays2023/gobgp$ ls
        gobgp_3.8.0_linux_amd64.tar.gz  gobgpd.conf
#### b - extract gobgp_3.8.0_linux_amd64.tar.gz:
       cisco@inserthostnamehere:~/devdays2023/gobgp$ tar -xvf gobgp_3.8.0_linux_amd64.tar.gz 
        LICENSE
        README.md
        gobgp
        gobgpd
      cisco@inserthostnamehere:~/devdays2023/gobgp$ ls
        LICENSE  README.md  gobgp  gobgp_3.8.0_linux_amd64.tar.gz  gobgpd  gobgpd.conf
#### c - test gobgp deamon:
       cisco@inserthostnamehere:~/devdays2023/gobgp$ cat gobgpd.conf
       [global.config]
         as = 11
         router-id = "10.1.1.2"

       [[neighbors]]
        [neighbors.config]
        neighbor-address = "10.1.1.1"
        peer-as = 1
#### d - run gobgp deamon:
       cisco@inserthostnamehere:~/devdays2023/gobgp$ sudo ./gobgpd -f gobgpd.conf
        {"level":"info","msg":"gobgpd started","time":"2023-05-04T17:45:38Z"}
        {"Topic":"Config","level":"info","msg":"Finished reading the config file","time":"2023-05-04T17:45:38Z"}
        {"Key":"10.1.1.1","Topic":"config","level":"info","msg":"Add Peer","time":"2023-05-04T17:45:38Z"}
        {"Key":"10.1.1.1","Topic":"Peer","level":"info","msg":"Add a peer configuration","time":"2023-05-04T17:45:38Z"}
        {"Key":"10.1.1.1","State":"BGP_FSM_OPENCONFIRM","Topic":"Peer","level":"info","msg":"Peer Up","time":"2023-05-04T17:45:40Z"}
        ^X^C{"level":"info","msg":"stopping gobgpd server","time":"2023-05-04T17:46:12Z"}
        {"Key":"10.1.1.1","Topic":"Peer","level":"info","msg":"Delete a peer configuration","time":"2023-05-04T17:46:12Z"}
        {"Key":"10.1.1.1","Reason":"dying","State":"BGP_FSM_ESTABLISHED","Topic":"Peer","level":"info","msg":"Peer Down","time":"2023-05-04T17:46:12Z"}

       
#### e - stop nso deamon:
       control + c

#### f - copy gobgp deamons to /usr/bin/ :
       cisco@inserthostnamehere:~/devdays2023/gobgp$ sudo cp gobgpd /usr/bin/
       cisco@inserthostnamehere:~/devdays2023/gobgp$ sudo cp gobgp /usr/bin/
       cisco@inserthostnamehere:~/devdays2023/gobgp$ gobgp --version
        gobgp version 3.8.0

       
 
#### g - go to /usr/bin/ and fix permissions:
       cisco@inserthostnamehere:~/devdays2023/gobgp$ cd /usr/bin/
       cisco@inserthostnamehere:/usr/bin$ ls -la gobgp*
        -rwxr-xr-x 1 root root 14589952 May  4 17:54 gobgp
        -rwxr-xr-x 1 root root 15683584 May  4 17:55 gobgpd
       cisco@inserthostnamehere:/usr/bin$ sudo chmod u+s /usr/bin/gobgpd
       cisco@inserthostnamehere:/usr/bin$ sudo chmod u+s /usr/bin/gobgp



## 3 - Configure high-availability:
 
#### a - enter nso:
       cisco@inserthostnamehere:/usr/bin$ ncs_cli -C

#### b - load merge high-availability configuration:
       cisco@ncs# config 
       cisco@ncs(config)# load merge terminal
        Loading.
#### c - paste the config and commit:
        high-availability token $9$XrjhhNHOYhhNi1StjHu8ZNVcUL42D28Rfgu1aeFhcWM=
        high-availability settings enable-failover true
        high-availability settings start-up assume-nominal-role true
        high-availability settings start-up join-ha true

        high-availability ha-node Lisbon 
        address      10.1.1.2
        nominal-role master
        !
        high-availability ha-node Stockholm
        address         30.1.1.2
        nominal-role    slave
        failover-master true
        !


#### d - click on enter then control + d then commit:
       0 bytes parsed in 12.51 sec (0 bytes/sec)
       cisco@ncs(config)# commit 
       Commit complete.


## 4 - Configure hcc:

#### a - load merge hcc configuration:
       cisco@ncs# config 
       cisco@ncs(config)# load merge terminal
        Loading.
#### b - paste the config and commit:
        hcc enabled
        hcc vip-address [ 10.2.1.1 ]
        hcc bgp node Lisbon 
        enabled
        gobgp-bin-dir /usr/bin
        as            11
        router-id     10.1.1.2
        neighbor 10.1.1.1
        as      1
        enabled
        !
        !
        hcc bgp node Stockholm
        enabled
        gobgp-bin-dir /usr/bin
        as            22
        router-id     30.1.1.2
        neighbor 30.1.1.1
        as      2
        enabled
        !
        !


#### d - click on enter then control + d then commit:
       0 bytes parsed in 1.02 sec (0 bytes/sec)
       cisco@ncs(config)# commit 
       Commit complete.

## 5 - testing GoBGP integration with nso on first node:

#### a - exit config mode:
       cisco@ncs(config)# exit

#### b - check gobgp status:
        cisco@ncs# show hcc
                BGPD  BGPD                                       
        NODE ID    PID   STATUS   ADDRESS   STATE        CONNECTED  
        ------------------------------------------------------------
        Lisbon     1339  running  10.1.1.1  ESTABLISHED  true       
        Stockholm  -     -        30.1.1.1  -            -          

       
## 5 - repeat the exact same steps for the secondary node

## 6 - repeat the exact same steps for the secondary node

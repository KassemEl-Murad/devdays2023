# Staying a Step Ahead of Disasters: NSO Geo redundancy high availability

## 1 - Login to dcloud:
#### a - please use your CEC credentials
#### b - Discover the topology
#### c - Connect to the anyconnect dcloud VPN
#### d - Connect to CML using cml-ui
        Credentials will be shown when you click on the cml
        similar to: 
                IP Address:
                198.18.134.1
                Credentials:
                username: guest
                password: C1sco12345
#### e - turn on devices:
        click on the start button
#### f - devices Credentials:
        username: cisco
        password: cisco


## 2 - connect to ubuntu-Lisbon:

#### a - login to nso and make sure there are no packages there:
       cisco@inserthostnamehere:~$ ncs --status | grep status
        status: started
        cluster status:
                db=running id=28 priority=1 path=/ncs:devices/device/live-status-protocol/device-type
        cisco@inserthostnamehere:~$ 

#### b - go to the tail-f package:
      cisco@inserthostnamehere:~$  cd /home/cisco/devdays2023/tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages/
      cisco@inserthostnamehere:~/devdays2023/tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages$ ls
        ncs-5.7.6-tailf-hcc-5.0.3.tar.gz  tailf-hcc
#### f - move this package to nso packages directory:
      cisco@inserthostnamehere: /var/opt/ncs/packages//tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages$ cd /var/opt/ncs/packages/
      cisco@inserthostnamehere:/var/opt/ncs/packages$ cp -r /home/cisco/devdays2023/tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages/tailf-hcc/ .
      cisco@inserthostnamehere:/var/opt/ncs/packages$ ls
        tailf-hcc

#### g - enter nso and do a package reload:
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

#### h - exit nso:
        cisco@ncs# exit

## 3 - test gobgp: 

#### a - make sure gobgp works :
       cisco@inserthostnamehere:/var/opt/ncs/packages$ gobgp --version
        gobgp version 3.8.0

#### b - link to install GoBGP (optional):
      https://osrg.github.io/gobgp/


#### c - test gobgp deamon (optional):
       cisco@inserthostnamehere:/var/opt/ncs/packages$ cd /home/cisco/devdays2023/gobgp/
       cisco@inserthostnamehere:~/devdays2023/gobgp$ cat gobgpd.conf
       [global.config]
         as = 11
         router-id = "10.1.1.2"

       [[neighbors]]
        [neighbors.config]
        neighbor-address = "10.1.1.1"
        peer-as = 1
#### d - run gobgp deamon (optional):
       cisco@inserthostnamehere:~/devdays2023/gobgp$ sudo ./gobgpd -f gobgpd.conf
        {"level":"info","msg":"gobgpd started","time":"2023-05-04T21:39:34Z"}
        {"Topic":"Config","level":"info","msg":"Finished reading the config file","time":"2023-05-04T21:39:34Z"}
        {"Key":"10.1.1.1","Topic":"config","level":"info","msg":"Add Peer","time":"2023-05-04T21:39:34Z"}
        {"Key":"10.1.1.1","Topic":"Peer","level":"info","msg":"Add a peer configuration","time":"2023-05-04T21:39:34Z"}
        {"Key":"10.1.1.1","State":"BGP_FSM_OPENCONFIRM","Topic":"Peer","level":"info","msg":"Peer Up","time":"2023-05-04T21:39:42Z"}

       
#### e - stop nso deamon (optional):
       control + c


## 4 - Configure high-availability:
 
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
        0 bytes parsed in 7.53 sec (0 bytes/sec)
        cisco@ncs(config)# commit dry-run 
        cli {
        local-node {
                data  high-availability {
                +    token $9$XrjhhNHOYhhNi1StjHu8ZNVcUL42D28Rfgu1aeFhcWM=;
                +    ha-node Lisbon {
                +        address 10.1.1.2;
                +        nominal-role master;
                +    }
                +    ha-node Stockholm {
                +        address 30.1.1.2;
                +        nominal-role slave;
                +        failover-master true;
                +    }
                        settings {
                +        enable-failover true;
                        start-up {
                +            assume-nominal-role true;
                +            join-ha true;
                        }
                        consensus {
                        }
                        }
                }
        }
        }
        cisco@ncs(config)# commit 
        Commit complete.
        cisco@ncs(config)#


## 5 - Configure hcc:

#### a - load merge hcc configuration:
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
        0 bytes parsed in 16.83 sec (0 bytes/sec)
        cisco@ncs(config)# commit dry-run 
        cli {
        local-node {
                data  hcc {
                +    enabled;
                +    vip-address [ 10.2.1.1 ];
                        bgp {
                +        node Lisbon {
                +            enabled;
                +            gobgp-bin-dir /usr/bin;
                +            as 11;
                +            router-id 10.1.1.2;
                +            neighbor 10.1.1.1 {
                +                as 1;
                +                enabled;
                +            }
                +        }
                +        node Stockholm {
                +            enabled;
                +            gobgp-bin-dir /usr/bin;
                +            as 22;
                +            router-id 30.1.1.2;
                +            neighbor 30.1.1.1 {
                +                as 2;
                +                enabled;
                +            }
                +        }
                        }
                }
        }
        }
        cisco@ncs(config)# commit 
        Commit complete.
        cisco@ncs(config)# 

## 6 - testing GoBGP integration with nso on first node:

#### a - exit config mode:
       cisco@ncs(config)# exit

#### b - check gobgp status:
        cisco@ncs# show hcc
                BGPD  BGPD                                       
        NODE ID    PID   STATUS   ADDRESS   STATE        CONNECTED  
        ------------------------------------------------------------
        Lisbon     1339  running  10.1.1.1  ESTABLISHED  true       
        Stockholm  -     -        30.1.1.1  -            -          

       
## 7 - repeat the exact same steps for the secondary node

#### a - go to the tail-f package:
      cisco@inserthostnamehere:~$ cd /home/cisco/devdays2023/tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages/
      cisco@inserthostnamehere:~/devdays2023/tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages$ ls
        ncs-5.7.6-tailf-hcc-5.0.3.tar.gz  tailf-hcc   

#### b - move this package to nso packages directory:
        cisco@inserthostnamehere: /var/opt/ncs/packages//tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages$ cd /var/opt/ncs/packages/
        cisco@inserthostnamehere:/var/opt/ncs/packages$ cp -r /home/cisco/devdays2023/tailf-hcc/ncs-5.7.6-tailf-hcc-project-5.0.3/packages/tailf-hcc/ .
        cisco@inserthostnamehere:/var/opt/ncs/packages$ ls
        tailf-hcc
       
#### c -  enter nso and do a package reload:
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


#### d - load merge high-availability configuration:
       cisco@ncs# config 
       cisco@ncs(config)# load merge terminal
        Loading.
#### e - paste the config and commit:
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


#### f - click on enter then control + d then commit:
        0 bytes parsed in 7.53 sec (0 bytes/sec)
        cisco@ncs(config)# commit dry-run 
        cli {
        local-node {
                data  high-availability {
                +    token $9$XrjhhNHOYhhNi1StjHu8ZNVcUL42D28Rfgu1aeFhcWM=;
                +    ha-node Lisbon {
                +        address 10.1.1.2;
                +        nominal-role master;
                +    }
                +    ha-node Stockholm {
                +        address 30.1.1.2;
                +        nominal-role slave;
                +        failover-master true;
                +    }
                        settings {
                +        enable-failover true;
                        start-up {
                +            assume-nominal-role true;
                +            join-ha true;
                        }
                        consensus {
                        }
                        }
                }
        }
        }
        cisco@ncs(config)# commit 
        Commit complete.
        cisco@ncs(config)#

#### g - load merge hcc configuration:
       cisco@ncs(config)# load merge terminal
        Loading.
#### h - paste the config and commit:
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


#### i - click on enter then control + d then commit:
        0 bytes parsed in 16.83 sec (0 bytes/sec)
        cisco@ncs(config)# commit dry-run 
        cli {
        local-node {
                data  hcc {
                +    enabled;
                +    vip-address [ 10.2.1.1 ];
                        bgp {
                +        node Lisbon {
                +            enabled;
                +            gobgp-bin-dir /usr/bin;
                +            as 11;
                +            router-id 10.1.1.2;
                +            neighbor 10.1.1.1 {
                +                as 1;
                +                enabled;
                +            }
                +        }
                +        node Stockholm {
                +            enabled;
                +            gobgp-bin-dir /usr/bin;
                +            as 22;
                +            router-id 30.1.1.2;
                +            neighbor 30.1.1.1 {
                +                as 2;
                +                enabled;
                +            }
                +        }
                        }
                }
        }
        }
        cisco@ncs(config)# commit 
        Commit complete.
        cisco@ncs(config)# 
       


## 8 - enable high-availability:

#### a - on ubuntu-Lisbon:
       cisco@ncs# high-availability enable 
        result enabled
        cisco@ncs# high-availability be-master 
        result ok
        cisco@ncs# 
       
#### b - on ubuntu-Stockholm:
        cisco@ncs# high-availability enable 
        result enabled
        cisco@ncs# high-availability be-slave-to node Lisbon 
        result Attempting to be slave to node Lisbon
        cisco@ncs# 
       
#### c - on ubuntu-Stockholm:
        cisco@ncs# show high-availability status 
        high-availability status mode slave
        high-availability status current-id Stockholm
        high-availability status assigned-role slave
        high-availability status be-slave-result connected
        high-availability status master-id Lisbon
        high-availability status read-only-mode false
        cisco@ncs# 
       
#### d - on ubuntu-Lisbon:
        cisco@ncs# show high-availability status 
        high-availability status mode master
        high-availability status current-id Lisbon
        high-availability status assigned-role master
        high-availability status read-only-mode false
        ID         ADDRESS   
        ---------------------
        Stockholm  30.1.1.2  
        cisco@ncs# 

## 9 - CDB replication test:
       
#### a - create an devices authgroups on ubuntu-Lisbon:
       cisco@ncs# config  
       cisco@ncs(config)# devices authgroups group admin default-map remote-name admin remote-password admin 
        cisco@ncs(config-group-admin)# commit dry-run 
        cli {
                local-node {
                        data  devices {
                                authgroups {
                        +        group admin {
                        +            default-map {
                        +                remote-name admin;
                        +                remote-password $9$3oVkVgMlZTTvBUyB2BI+qwND6wKbSQMqPSRpGh56FFY=;
                        +            }
                        +        }
                                }
                        }
                }
        }
        cisco@ncs(config-group-admin)# commit 
        Commit complete.
        cisco@ncs(config-group-admin)# 
#### b - on ubuntu-Stockholm:
        cisco@ncs# show running-config devices authgroups 
        devices authgroups group admin
        default-map remote-name admin
        default-map remote-password $9$3oVkVgMlZTTvBUyB2BI+qwND6wKbSQMqPSRpGh56FFY=
        !
        cisco@ncs#

#### c - delete devices authgroups config on ubuntu-Lisbon:
        cisco@ncs(config-group-admin)# top
        cisco@ncs(config)# no devices authgroups group admin 
        cisco@ncs(config)# commit dry-run 
        cli {
        local-node {
                data  devices {
                        authgroups {
                -        group admin {
                -            default-map {
                -                remote-name admin;
                -                remote-password $9$hV5lq4FVK+37qpzaOBYqPhLGeH1Ypn7bv0Bnd8JadZw=;
                -            }
                -        }
                        }
                }
        }
        }
        cisco@ncs(config)# commit 
        Commit complete.
        cisco@ncs(config)# 

#### d - show running-config devices authgroups on ubuntu-Stockholm:
        cisco@ncs# show running-config devices authgroups
        % No entries found.
        cisco@ncs# 

## 10 - fail-over test:
       
#### a - shut down ubuntu-Lisbon:
        click on the red button
#### b - on ubuntu-Stockholm:
        cisco@ncs# show high-availability status
        high-availability status mode none
        high-availability status assigned-role slave
        high-availability status be-slave-result "error (25) - could not connect to master"
        high-availability status master-id Lisbon
        high-availability status read-only-mode false
        cisco@ncs# 
        cisco@ncs# 

#### c - ubuntu-Stockholm will become master after all the attempts to connect to ubuntu-Lisbon:
        cisco@ncs# show high-availability status
        high-availability status mode master
        high-availability status current-id Stockholm
        high-availability status assigned-role master
        high-availability status read-only-mode false
        cisco@ncs# 

#### d - turn on ubuntu-Lisbon and check high-availability status:
        cisco@ncs# show high-availability status
        high-availability status mode slave
        high-availability status current-id Lisbon
        high-availability status assigned-role slave
        high-availability status be-slave-result initialized
        high-availability status master-id Stockholm
        high-availability status read-only-mode false

#### e - disable high-availability on both nodes and enable it again:



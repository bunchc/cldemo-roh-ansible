Routing On the Host (RoH) With HA-Proxy Demo
===========================

This demo is based on the original Cumulus Networks [ROH Demo](https://github.com/CumulusNetworks/cldemo-roh-ansible) and corresponding topology.

This demo shows a topology using 'Routing on the Host' to add host reachability directly into a BGP routed fabric. Switches are Cumulus Linux and servers running Ubuntu. This playbook configures a CLOS topology running BGP unnumbered in the fabric with numbered links towards the hosts, and installs a webserver and haproxy on both hosts to serve as a Hello World example. The Ubuntu server acts as the host to redistribute kernel routes as /32 host routes into the routing table when VMs or containers become available.

## Topology
```
     +------------+       +------------+
     | spine01    |       | spine02    |
     |            |       |            |
     +------------+       +------------+
     swp1 |    swp2 \   / swp1    | swp2
          |           X           |
    swp51 |   swp52 /   \ swp51   | swp52
     +------------+       +------------+
     | leaf01     |       | leaf02     |
     |            |       |            |
     +------------+       +------------+
     swp1 |                       | swp2
          |                       |
     eth1 |                       | eth2
     +------------+       +------------+
     | server01   |       | server02   |
     |            |       |            |
     +------------+       +------------+
```


When the lab is operational, each server (server01 and server02) is configured to run Apache on 8081 and serve a generic index.html. Further, haproxy has been deployed to each node and configured to load balance across both servers. The servers (server01 and server02) are also configured to advertise their haproxy service addresses (seperate from their interface addresses) into BGP.

The leaf nodes (leaf01 and leaf02) are connected to both load balancers across different link networks and have two equal cost routes set up to the service IP. They know they can reach the service IP across both links and must make a routing decision about which one to use.

This demo is written for the [cldemo-vagrant](https://github.com/cumulusnetworks/cldemo-vagrant) reference topology and applies the reference BGP unnumbered configuration from [cldemo-config-routing](https://github.com/cumulusnetworks/cldemo-config-routing).

Quickstart: Run the demo
------------------------
    ### Bring up the vagrant topology
    git clone https://github.com/cumulusnetworks/cldemo-vagrant
    cd cldemo-vagrant
    vagrant up oob-mgmt-server oob-mgmt-switch leaf01 leaf02 spine01 spine02 server01 server02
    ### Run the ROH demo
    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    git clone https://github.com/bunchc/cldemo-roh-ansible
    cd cldemo-roh-ansible
    ansible-playbook run-demo.yml

    ### if needed, setup oob mgmt server
    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    sudo apt-get install software-properties-common -y
    sudo apt-add-repository ppa:ansible/ansible -y
    sudo apt-get update
    sudo apt-get install ansible -qy

### Resources

- [Getting started with Topology Converter](http://blog.codybunch.com/2016/08/22/Revisiting-BGP-on-Linux-w-Cumulus-Topology-Converter/)
- [Sysadvent Load Balancing Across Layer 3](https://sysadvent.blogspot.com/2014/12/day-11-turning-off-pacemaker-load.html)
- [High availability with ExaBGP](https://vincent.bernat.im/en/blog/2013-exabgp-highavailability.html)
- [Ansible Variables](https://docs.ansible.com/ansible/playbooks_variables.html)
- [Configure Apache with Ansible](https://www.digitalocean.com/community/tutorials/how-to-configure-apache-using-ansible-on-ubuntu-14-04)
- [Configure haproxy with Ansible](http://blog.toast38coza.me/setup-and-configure-haproxy-with-ansible/)
- [OpenSource Routing, Practical Lab](https://keepingitclassless.net/2015/06/open-source-routing-practical-lab/)
- [OpenSource Routing, Comparison](https://keepingitclassless.net/2015/05/open-source-routing-comparison/)

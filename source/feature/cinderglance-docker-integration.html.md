---
title: CinderGlance Docker Integration
category: feature
authors: sandrobonazzola, stirabos
wiki_category: Feature|CinderGlance Docker Integration
wiki_title: CinderGlance Docker Integration
wiki_revision_count: 11
wiki_last_updated: 2015-05-06
feature_name: Cinder and Glance integration via Docker
feature_modules: engine
feature_status: Developement
---

# CinderGlance Docker Integration

Cinder and Glance integration via Docker

## Your Feature Name

### Summary

The use of Docker containers make easier to locally deploy Cinder and Glance on the engine host. We are going to use docker images from kollaglue project: kollaglue is the official effort to deploy openstack components via docker.

### Owner

This should link to your home wiki page so we know who you are

*   Name: [ Simone Tiraboschi](User:stirabos)

Include you email address that you can be reached should people want to contact you about helping with your feature, status is requested, or technical issues need to be resolved

*   Email: <stirabos@redhat.com>

### Detailed Description

The aim of this feature is to let the user simply deploy and configure cinder and glance within the engine from engine-setup.

To avoid any new strong dependency a new engine-setup plugin will be developed. Installing that plugin from its rm and running engine-setup the user will be able to pull the right docker containers, configure them and configure them as external providers within the engine.

### Benefit to oVirt

The user will be able to locally run glance to import and store VM images and cinder to access external volumes. Deploying them as docker images makes it more easier to be accomplished without the need for a manual complex configuration.

### Dependencies / Related Features

glance could be already used as an external provider from the engine, cinder integraton still require more effort on backend side: <http://www.ovirt.org/Features/Cinder_Integration>

### Documentation / External references

Upstream information about kollaglue could be found there: <https://github.com/stackforge/kolla/>

glance requires two distinct containers:

      kollaglue/centos-rdo-glance-registry:latest
      kollaglue/centos-rdo-glance-api:latest

cinder one:

      kollaglue/centos-rdo-cinder:latest

both of them requires additional containers:

      kollaglue/centos-rdo-rabbitmq:latest
      kollaglue/centos-rdo-mariadb-data:latest
      kollaglue/centos-rdo-mariadb-app:latest
      kollaglue/centos-rdo-keystone:latest

### Testing

Install the plugin rpm

      yum install ovirt-engine-setup-plugin-dockerc

Run engine setup

      engine-setup

Choose to deploy cinder and glance

               Deploy Cinder container on this host (Yes, No) [No]: yes
               Deploy Glance container on this host (Yes, No) [No]: yes

It should deploy and start all the required containers

      [ INFO  ] Starting Docker
      [ INFO  ] Pulling rabbitmq
      [ INFO  ] Pulling mariadbdata
      [ INFO  ] Pulling mariadbapp
      [ INFO  ] Pulling keystone
      [ INFO  ] Pulling cinder
      [ INFO  ] Pulling glance-registry
      [ INFO  ] Pulling glance-api
      [ INFO  ] Creating rabbitmq
      [ INFO  ] Starting rabbitmq
      [ INFO  ] Creating mariadbdata
      [ INFO  ] Starting mariadbdata
      [ INFO  ] Creating mariadbapp
      [ INFO  ] Starting mariadbapp
      [ INFO  ] Creating keystone
      [ INFO  ] Starting keystone
      [ INFO  ] Creating cinder
      [ INFO  ] Starting cinder
      [ INFO  ] Creating glance-registry
      [ INFO  ] Starting glance-registry
      [ INFO  ] Creating glance-api
      [ INFO  ] Starting glance-api

When the setup completes you could check the status of the containers with:

      [root@c7t1 ~]# docker ps -a
      CONTAINER ID        IMAGE                                         COMMAND                CREATED             STATUS                               PORTS               NAMES
      ed3c0510e0d8        kollaglue/centos-rdo-glance-api:latest        "/start.sh"            4 hours ago         Up 4 hours                                               glance-api          
      0bc3674d9b63        kollaglue/centos-rdo-glance-registry:latest   "/start.sh"            4 hours ago         Up 4 hours                                               glance-registry     
      31cea8df4547        kollaglue/centos-rdo-cinder:latest            "/start.sh"            4 hours ago         Restarting (127) About an hour ago                       cinder              
      1b703e42a671        kollaglue/centos-rdo-keystone:latest          "/start.sh"            4 hours ago         Up 4 hours                                               keystone            
      cf2e262f74c1        kollaglue/centos-rdo-mariadb-app:latest       "/opt/kolla/mysql-en   4 hours ago         Up 4 hours                                               mariadbapp          
      026018f2e8e2        kollaglue/centos-rdo-mariadb-data:latest      "/bin/sh"              4 hours ago         Restarting (0) About an hour ago                         mariadbdata         
      5522df92da33        kollaglue/centos-rdo-rabbitmq:latest          "/start.sh"            4 hours ago         Up 4 hours                                               rabbitmq 

You could check the logs of a specific container with

      [root@c7t1 ~]# docker logs glance-api
`keystone is active @ `[`http://c7t1.localdomain:5000/v2.0`](http://c7t1.localdomain:5000/v2.0)
      using existing tenant admin (e7355c03ffaf453180615ccea0c33f5f)
      creating new user glance
      created user glance (528b2b926b4c4fc4b81359b0476b4fa8)
      2015-04-15 10:02:31.898 1 INFO glance.wsgi.server [-] Started child 79
`2015-04-15 10:02:31.898 79 INFO glance.wsgi.server [-] (79) wsgi starting up on `[`http://0.0.0.0:9292/`](http://0.0.0.0:9292/)
`2015-04-15 10:02:31.901 80 INFO glance.wsgi.server [-] (80) wsgi starting up on `[`http://0.0.0.0:9292/`](http://0.0.0.0:9292/)
      2015-04-15 10:02:31.901 1 INFO glance.wsgi.server [-] Started child 80

You could connect to the container with

      [root@c7t1 ~]# PID=$(docker inspect --format {{.State.Pid}} glance-api)
      [root@c7t1 ~]# nsenter --target $PID --mount --uts --ipc --net --pid
      [root@c7t1 /]# 

Than you should setup the environment

      [root@c7t1 /]# source openrc 

At this point you could download a VM image from the Internet and upload it into your local glance

`[root@c7t1 /]# IMAGE_URL=`[`http://download.cirros-cloud.net/0.3.3/`](http://download.cirros-cloud.net/0.3.3/)
      [root@c7t1 /]# IMAGE=cirros-0.3.3-x86_64-disk.img
      [root@c7t1 /]# curl -L -o ./$IMAGE $IMAGE_URL/$IMAGE
        % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100   255  100   255    0     0    451      0 --:--:-- --:--:-- --:--:--   451
      100 12.5M  100 12.5M    0     0   997k      0  0:00:12  0:00:12 --:--:-- 1088k
      [root@c7t1 /]# glance image-create --name cirros --is-public false --disk-format qcow2 --container-format bare --file ./$IMAGE
      +------------------+--------------------------------------+
      | Property         | Value                                |
      +------------------+--------------------------------------+
      | checksum         | 133eae9fb1c98f45894a4e60d8736619     |
      | container_format | bare                                 |
      | created_at       | 2015-04-15T14:54:49                  |
      | deleted          | False                                |
      | deleted_at       | None                                 |
      | disk_format      | qcow2                                |
      | id               | 2289c0f1-bc89-4f77-ac9a-f1b2d926f89d |
      | is_public        | False                                |
      | min_disk         | 0                                    |
      | min_ram          | 0                                    |
      | name             | cirros                               |
      | owner            | e7355c03ffaf453180615ccea0c33f5f     |
      | protected        | False                                |
      | size             | 13200896                             |
      | status           | active                               |
      | updated_at       | 2015-04-15T14:54:50                  |
      | virtual_size     | None                                 |
      +------------------+--------------------------------------+
      [root@c7t1 /]#

At this point, connecting to the web-admin engine UI, you should be able to find a new local glance provider with a cirros image inside.

#### Additional testing

EPEL and RHEL 7.1 includes docker 1.5 while Centos virt-SIG already includes a fresher virt 1.6 one. <http://wiki.centos.org/Cloud/Docker> It should be tested also against that to ensure future-proofness.

#### Network configuration

Docker images got their own network configuration (hotsname, /etc/hosts, dns config...). A proper network setup is required to avoid any issue.

### Release Notes

      == Docker Integration ==
      oVirt Engine setup now provides `[ `Cinder` `and` `Glance`](CinderGlance Docker Integration)` automated deployment using Docker

### Comments and Discussion

This below adds a link to the "discussion" tab associated with your page. This provides the ability to have ongoing comments or conversation without bogging down the main feature page

*   Refer to <Talk:CinderGlance_Docker_Integration>

[CinderGlance Docker Integration](Category:Feature) [CinderGlance Docker Integration](Category:oVirt 3.6 Proposed Feature) [CinderGlance Docker Integration](Category:Integration)

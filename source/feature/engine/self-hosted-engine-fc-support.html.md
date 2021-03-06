---
title: Self Hosted Engine FC Support
category: feature
authors: stirabos
wiki_category: Feature|Self Hosted Engine FC Support
wiki_title: Features/Self Hosted Engine FC Support
wiki_revision_count: 18
wiki_last_updated: 2014-12-15
feature_name: Self Hosted Engine FC Support
feature_modules: ovirt-hosted-engine-setup
feature_status: design
---

# Self Hosted Engine FC Support

### Summary

This feature enable the user to use FC storage for Hosted Engine data domain.

### Owner

*   Name: [ Simone Tiraboschi](User:Stirabos)
*   Email: <stirabos@redhat.com>

### Detailed Description

##### UX changes

Using an existing FC storage:

tbd

##### Config files changes

tbd

##### VDSM commands involved

tbd

It should be not that different from iSCSI storage.

### Benefit to oVirt

Users will be able to use FC storage as data domain for Hosted Engine.

### Dependencies / Related Features

*   A tracker bug has been created for tracking issues:

### Documentation / External references

#### Development environment

The feature can be developed and tested in a simplified environment without the need of a real SAN using FCoE (Fibre-Channel over Ethernet) in VN2VN mode (FCoE Direct End-Node to End-Node) on a nested environment.

##### Prerequisites

Two virtual machine with two VirtIO network adapter for each node. The first one (eth0) will be used for generic network traffic, the second one (eth1) will be dedicated to FCoE. The first virtual machine will be used to export a block device as a virtual SAN, the second one will connect to it and it will be used for hosted-engine.

On both the hosts, install the required FCoE utilities:

      yum install lldpad fcoe-utils

###### virtIO issue

libhbalinux currently doesn't detect correctly a VirtIO interface cause it scans the system using sysfs to look for PCI adapter. A recent patch [1] uses libudev instead of a direct scan of sysfs and so it than works also with VirtIO devices. The patch hasn't still been merged and so it should manually be applied to libhbalinux than libhbalinux , libHBAAPI and fcoe-utils should be rebuilt. [1] <http://lists.open-fcoe.org/pipermail/fcoe-devel/2014-October/012358.html> After that fcoe-utils seams to correctly work also on VirtIO interfaces.

Activate eth1 interface, don't assign any ipadress to it

      ifconfig eth1 up

###### FCoE service

Start and enable fcoe service

      systemctl start fcoe
      systemctl enable fcoe

###### FCoE interfaces

Create FCoE interface

      fcoeadm -m vn2vn -c eth1

Just being a test environment for development purposes DCB is not really needed so no really need to customize /etc/fcoe/cfg-eth1 and start lldpad.

Check the result

      [root@f20t2 ~]# fcoeadm -i
          Description:      Virtio network device
          Revision:         00
          Manufacturer:     Red Hat, Inc
          Serial Number:    Unknown
          Driver:           Unknown 1
          Number of Ports:  1
              Symbolic Name:     fcoe v0.1 over eth1
              OS Device Name:    host3
              Node Name:         0x1000001A4A4FBD29
              Port Name:         0x2000001A4A4FBD29
              FabricName:        0x0000000000000000
              Speed:             Unknown
              Supported Speed:   Unknown
              MaxFrameSize:      1452
              FC-ID (Port ID):   0x00BD29
              State:             Online

The interface should be Online

The same on the second host:

      [root@f20t3 ~]# ifconfig eth1 up
      [root@f20t3 ~]# fcoeadm -m vn2vn -c eth1
      [root@f20t3 ~]#  fcoeadm -i
          Description:      Virtio network device
          Revision:         00
          Manufacturer:     Red Hat, Inc
          Serial Number:    Unknown
          Driver:           Unknown 1
          Number of Ports:  1
              Symbolic Name:     fcoe v0.1 over eth1
              OS Device Name:    host3
              Node Name:         0x1000001A4A4FBD2B
              Port Name:         0x2000001A4A4FBD2B
              FabricName:        0x0000000000000000
              Speed:             Unknown
              Supported Speed:   Unknown
              MaxFrameSize:      1452
              FC-ID (Port ID):   0x00BD2B
              State:             Online

##### FCoE Target Setup

Now let's create the FCoE target.

On the host that will be used as the virtual SAN:

      install targetcli utility
      yum install targetcli

Use targetcli utilities

      targetcli

Create a file based block storage device

      /> backstores/fileio create disk3 /mnt/disk3.img 32G
      Created fileio disk3 with size 34359738368

Create an FCoE target instance on the previously defined VirtIo FCoE interface (pressing tab after create is enough to complete the correct device name)

      /> tcm_fc/ create naa.2000001a4a4fbd29
      Created target naa.2000001a4a4fbd29.

Map the filebased backstore to the target instance.

      /> cd tcm_fc/naa.2000001a4a4fbd29/
      /tcm_fc/naa.2000001a4a4fbd29> 
      /tcm_fc/naa.2000001a4a4fbd29> luns/ create /backstores/fileio/disk3
      Created LUN 0.

Define an ACL for the FCoE initiator (the FCoE interface on the other host, changing the interface name to match the MAC of the interface on the other host should be enough to find it)

      /tcm_fc/naa.2000001a4a4fbd29> acls/ create naa.2000001a4a4fbd2b
      Created Node ACL for naa.2000001a4a4fbd2b
      Created mapped LUN 0.

Save the configuration

      /tcm_fc/naa.2000001a4a4fbd29> cd /
      /> saveconfig 
      Last 10 configs saved in /etc/target/backup.
      Configuration saved to /etc/target/saveconfig.json

Exit target cli with exit command.

##### FCoE initiator Setup

On the second host a FCoE scan would be enough to find the new device.

      [root@f20t3 ~]# ifconfig eth1 up
      [root@f20t3 ~]# fcoeadm -S eth1

If everything is OK something like

      Dec 15 12:31:28 f20t3 kernel: [  151.162210] scsi host3: FCoE Driver
      Dec 15 12:31:28 f20t3 kernel: [  151.162712] fcoe: No FDMI support.
      Dec 15 12:31:28 f20t3 kernel: [  151.162844] host3: libfc: Link up on port (000000)
      Dec 15 12:31:28 f20t3 kernel: scsi host3: FCoE Driver
      Dec 15 12:31:28 f20t3 kernel: fcoe: No FDMI support.
      Dec 15 12:31:28 f20t3 kernel: host3: libfc: Link up on port (000000)
      Dec 15 12:31:28 f20t3 kernel: [  151.671140] host3: Assigned Port ID 00bd2b
      Dec 15 12:31:28 f20t3 kernel: host3: Assigned Port ID 00bd2b
      Dec 15 12:31:29 f20t3 kernel: [  152.073351] scsi 3:0:0:0: Direct-Access     LIO-ORG  disk4            4.0  PQ: 0 ANSI: 5
      Dec 15 12:31:29 f20t3 kernel: scsi 3:0:0:0: Direct-Access     LIO-ORG  disk4            4.0  PQ: 0 ANSI: 5
      Dec 15 12:31:29 f20t3 kernel: [  152.075125] sd 3:0:0:0: Attached scsi generic sg2 type 0
      Dec 15 12:31:29 f20t3 kernel: [  152.075169] sd 3:0:0:0: [sda] 67108864 512-byte logical blocks: (34.3 GB/32.0 GiB)
      Dec 15 12:31:29 f20t3 kernel: sd 3:0:0:0: Attached scsi generic sg2 type 0
      Dec 15 12:31:29 f20t3 kernel: sd 3:0:0:0: [sda] 67108864 512-byte logical blocks: (34.3 GB/32.0 GiB)
      Dec 15 12:31:29 f20t3 kernel: [  152.077286] sd 3:0:0:0: [sda] Write Protect is off
      Dec 15 12:31:29 f20t3 kernel: [  152.077495] sd 3:0:0:0: [sda] Write cache: enabled, read cache: enabled, supports DPO and FUA
      Dec 15 12:31:29 f20t3 kernel: sd 3:0:0:0: [sda] Write Protect is off
      Dec 15 12:31:29 f20t3 kernel: sd 3:0:0:0: [sda] Write cache: enabled, read cache: enabled, supports DPO and FUA
      Dec 15 12:31:29 f20t3 kernel: [  152.079079]  sda: unknown partition table
      Dec 15 12:31:29 f20t3 kernel: [  152.079960] sd 3:0:0:0: [sda] Attached SCSI disk
      Dec 15 12:31:29 f20t3 kernel: sda: unknown partition table
      Dec 15 12:31:29 f20t3 kernel: sd 3:0:0:0: [sda] Attached SCSI disk
      Dec 15 12:31:29 f20t3 multipathd: sda: add path (uevent)
      Dec 15 12:31:29 f20t3 multipathd: 36001405bb378722b9b34eaf92db93644: load table [0 67108864 multipath 0 0 1 1 service-time 0 1 1 8:0 1]
      Dec 15 12:31:29 f20t3 multipathd: 36001405bb378722b9b34eaf92db93644: event checker started
      Dec 15 12:31:29 f20t3 multipathd: sda [8:0]: path added to devmap 36001405bb378722b9b34eaf92db93644

should appear in the syslog Also VDSM should find the new device

      [root@f20t3 ~]# vdsClient -s 0 getDeviceList 2
      [{'GUID': '36001405bb378722b9b34eaf92db93644',
        'capacity': '34359738368',
        'devtype': 'FCP',
        'fwrev': '4.0',
        'logicalblocksize': '512',
        'pathlist': [],
        'pathstatus': [{'lun': '0',
                        'physdev': 'sda',
                        'state': 'failed',
                        'type': 'FCP'}],
        'physicalblocksize': '512',
        'productID': 'disk4',
        'pvUUID': '',
        'serial': '',
        'status': 'used',
        'vendorID': 'LIO-ORG',
        'vgUUID': ''}]

### Testing

Test plan still to be created

### Contingency Plan

Currently all the changes required for this feature are in a single patch. If it won't be ready it won't be merged.

### Release Notes

      ==Self Hosted Engine FC Support==
`Hosted Engine has now added support for `[`FC` `storage`](Features/Self_Hosted_Engine_FC_Support)

### Comments and Discussion

*   Refer to [Talk:Self Hosted Engine FC Support](Talk:Self Hosted Engine FC Support)

[Self Hosted Engine FC Support](Category:Feature) [Self Hosted Engine FC Support](Category:oVirt 3.6 Proposed Feature)

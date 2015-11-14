---
title: ImportStorageDomain
category: feature
authors: derez, mlipchuk, sandrobonazzola, vered
wiki_category: Feature|ImportStorageDomain
wiki_title: Features/ImportStorageDomain
wiki_revision_count: 178
wiki_last_updated: 2015-01-14
feature_name: Import Storage Domain
feature_modules: engine/vdsm
feature_status: Released
---

# Import Storage Domain

This feature is part of <http://www.ovirt.org/Features/ImportUnregisteredEntities>

### Summary

Today, oVirt supports importing ISO and Export Storage Domains, however, there is no support for importing an existing Data Storage Domain.
A Data Storage Domain contains disks volumes and VMs'/Templates' OVF files.
The OVF file is an XML standard representing the VM/Template configuration, including disks, memory, CPU and more.
Based on this information stored in the Storage Domain, we can revive entities such as disks, VMs and Templates in the setup of any Data Center the Storage Domain will be attached to.
The usability of the feature might be useful for various use cases, here are some of them:

*   Recover after the loss of the oVirt Engine's database.
*   Transfer VMs between setups without the need to copy the data into and out of the export domain.
*   Support migrating Storage Domains between different oVirt installations.

Storage Domains that can be restored for VMs/Templates must contain OVF_STORE disks.
Since OVF_STORE disk is only supported from a 3.5v Data Center, the Storage Domains that can be restored have to be managed in a 3.5v Data Center before the disaster.

### Owner

*   Maor Lipchuk
*   Email <mlipchuk@redhat.com>

### Current status

*   Implemented

### General Functionality

*   The feature should be fully supported from oVirt 3.5.
*   Storage Domains that can be restored for VMs/Templates must contain OVF_STORE disks. Since OVF_STORE disk is only supported from a 3.5v Data Center, the Storage Domains that can be restored have to be managed in a 3.5v Data Center before the disaster.
*   Attach of a Storage domain from a disaster environment, which its meta data still indicates it is attached to another Data Center, is only supported for 3.5 Data Center.
*   The feature is dependent on both features:

1.  Detach/Attach Storage Domain - <http://www.ovirt.org/Features/ImportUnregisteredEntities>. The following is the general functionality of the Detach/Attach Storage Domain:
    1.  On detach of Storage Domain the VMs/Templates related to the Storage Domain should be deleted from the engine, but their data will be converted to an XML data which will be preserved in a DB table called unregistered_ovf_of_entities, and will still be part of the OVF disk contained in the Storage Domain.
    2.  On attach the user will be able to choose the VMs/Templates/Disks he/she desires to register in the Data Center, and will choose which Cluster and quota for each Vm/Template it will be assigned with.
    3.  After a successful registration of a VM/Template, the entity should be removed from the entities candidates to be registered.
    4.  The VM's snapshots and VM's disks (active/deactivate) should be preserved on attach, the same as they were when those entities were on the detached Storage Domain.
    5.  Regarding quota enforcement Data Centers, the user will choose for each disk the quota he/she will want to consume from, when it will choose a VM/Template to register in the setup.

2.  OVF on any Storage Domain - <http://www.ovirt.org/Feature/OvfOnWantedDomains>

*   The user can import a Storage Domains and attach it directly to a Data Center, or it can be imported as 'unattached' Storage Domain, and later the user can attach it to a Data Center he desires.
*   When attaching a Storage Domain to a Data Center, all the entities(VMs,Templates) from the OVF_STORE disk should be retrieved from the tar file and into the Data Base table unregistered_ovf_of_entities, later the user can decide how to register them into the Data Center (see <http://www.ovirt.org/Features/ImportUnregisteredEntities#General_Functionality>)
*   Once those VM/Template will be in the Data Base, the user should be able to register those entities using the import unregistered entities feature [see <http://www.ovirt.org/Features/ImportUnregisteredEntities#Work_flow_for_detach_and_attach_Storage_Domain_with_entities_-_UI_flow>]

#### Restrictions

*   Detach/Attach Storage Domain, containing entities, should not be restricted by any Data Center version.
     VMs and Templates can be moved from old/new Data Center to another with no limitation, except the cluster which the user choose for each VM/Template.
*   An import of a Storage Domain will not reflect the status of a VM (Up, Powring Up, Shutting Down...) all the VMs will be registered with down status.
*   An import of a Storage Domain should be supported for block Storage Domain, and file Storage Domain.
*   In a disaster recovery scenario, if the Host, which the user about to use, was in the environment which was destroyed, it is recommended to reboot this Host before adding it to the new setup. The reason for that is first, to kill any qemu processes which are still running and might be automatically be added as VMs into the new setup, and also to avoid any sanlock issues.
*   Detach will not be permitted if there are VMs/Templates which are delete protected. In case the Storage Domain contains disks which are attached to VMs which are configured as delete protected, the operation should be blocked and an appropriate message should be presented to the user.
*   Detach will not be permitted if there are VMs which are in preview mode. In case the Storage Domain contains disks which are attached to VMs which are in preview mode, the operation should be blocked and an appropriate message should be presented to the user.
*   Detach will not be permitted if there are VMs which are part of pools, In case the Storage Domain contains disks which are attached to VMs which are part of pool, the operation should be blocked and an appropriate message should be presented to the user.
*   a Storage Domain can not be detached if it contains disks which are related to a running VM, unless this disks are inactive.
*   Shareable and Direct lun disks are not supported in the OVF file, therefore will not be part of the recovered VM.
*   The OVF_STORE disk will contain all the entities configuration which are candidates to be registered.
     The candidates are VMs and Templates which has at least one disk exists in the Storage Domain OVF contained in the unregistered_ovf_of_entities table.
*   Currently all the Storage Domains which are related to the VMs/Templates disks must exist and be active in the Data Center once the entity get registred. (see <https://bugzilla.redhat.com/1133300>)
*   Registering a thin provisioned VM which is based on a Template is dependent on the Template existence in the setup.
*   Currently floating disks will be registered using the existing REST command of import unregistered disk.(see REST part for how to register a floating disk)
*   Permissions on VMs and Templates will not be preserved on detach, since they are not part of the OVF. (https://bugzilla.redhat.com/1138177)
*   detach/attach operations with Local Storage Domain will not support migrating unregistered entities, the reason for that is that on the detach the Local Storage Domain is being deleted from the Host.
*   Attaching an imported Storage Domain can only be applied with an initialized Data Center. (see [6])
*   If a Storage Domain will not contain the OVF_STORE disk, the engine should attach the Storage Domain without any unregistered entities, and an audit log should be presented.
*   The engine should retrieve the unregistered entities from the most updated OVF_STORE disk from all the OVF_STORE disks contained in the Storage Domain.
*   If the chosen OVF_STORE disk will contain an entity which already exists in the unregistered_ovf_of_entities table (see <http://www.ovirt.org/Features/ImportUnregisteredEntities#General_Functionality>), the engine will replace the data in the unregistered_ovf_of_entities table with the VM fetched from the OVF_STORE disk.

#### Implementation gaps

[2] The attach operation should notify the user, a warning, whether the Storage Domain is already attached to another Data Center.
 The user can then choose whether to run over the meta data or neglect its operation. (https://bugzilla.redhat.com/1138115)
[4] Open Issue: We should have an indication of External LUN disk on the Lun (https://bugzilla.redhat.com/1138121)
[5] When the user moved the Storage Domain to maintenance, all the entities related to the Storage Domain should be updated in the OVF_STORE disk. (https://bugzilla.redhat.com/1138124 )
[6] Currently, VDSM take a lock on the storage pool when performing a detach operation, this obstacle should be removed in a later version, once the storage pool will be removed completely in VDSM. (https://bugzilla.redhat.com/1138126)
[7] Currently alias names of disks are not persisted in the Storage Domain, so registering disks, will not have alias names. The alias name should be persisted in the Description of the disk in the Storage Domain. (https://bugzilla.redhat.com/1138129)
[8] Add support for importing iSCSI Storage Domain through REST api.
[9] The login button, when picking the targets for importing iSCSI Storage domain should be more noticeable in the GUI (https://bugzilla.redhat.com/1138131)

### Disaster Recovery flows

This is an example of how to recover a setup if it encountered a disaster.
1. Create a new engine setup with new Data Base (see <http://www.ovirt.org/Quick_Start_Guide#Install_oVirt>)
2. Create a new Data Center version 3.5 with cluster and add a Host to this cluster. (Recommended to reboot the Host)
3. Once the Host is UP and running, add and activate a new empty Storage Domain to initialize the Data Center.
4. If there were VMs/Templates which ran in the old setup on different compatible versions, or different CPU types, then those type of clusters should be created on the new Data Center.
5. Follow the instructions of importing Storage Domain, depended on the type of Storage Domain which the user wants to recover:
\* For Import block Storage Domain - <http://www.ovirt.org/Features/ImportStorageDomain#Work_flow_for_Import_block_Storage_Domain_-_UI_flow>

*   For Import file Storage Domain - <http://www.ovirt.org/Features/ImportStorageDomain#Work_flow_for_Import_File_Storage_Domain_-_UI_flow>

### GUI Perspective

### Work flow for detach and attach Storage Domain with entities - UI flow

Video Example: <iframe width="300" src="//youtube.com/embed/DLcxDB0MY38" frameborder="0" align="right" allowfullscreen="true"> </iframe> 1. Choose an active Storage Domain from an active Data Center, make sure this Storage Domain contains VMs/Templates with disks hosted in the specific Storage Domain
2. Move the Storage Domain to maintenance, and detach it from the Data Center - At this point all the entities related to the Storage Domain should be deleted from the setup
3. Attach the Storage Domain to another Data Center and activate it.
4. After the Storage Domain is activated, go to the Storage main tab and pick the Storage Domain which was activated a minute ago
5. In the same Storage main tab, the user should see two sub tabs, "Import VMs" and "Import Tempaltes", in the "Import VMs" sub tab, the user should see all the VMs which are candidates to be imported, and in the "Import Tempaltes" sub tab, there should be the same only for templates.
6. The user can pick several VMs (or Templates), and press on the "import" button.
7. When the "Import" button is pressed, a dialog should be opened, showing the list of all the entities the user chose to register.
 The user should choose a cluster for each entity which should be compatible for it.
 The user can also watch the entity properties (such as disks, networks) in the sub tab inside the dialog.

#### Work flow for Import block Storage Domain - UI flow

On import a Block Device Storage Domain The user should do the following steps:
1. The user should press the "import Storage Domain" button.
2. The user should choose iSCSI or FCP type of Storage Domain.
3. The user should provide a Storage Server name or IP, to be imported from.
4. The engine should present the user a list of targets related to the Storage Server provided in step 3.
5. The user should pick the targets which he knows are related to the Storage Domains he/she wants to import and press the connect button at the top.
6. After the engine will connect to those targets, the user should see in the bottom of the dialog a list of Storage Domains which are candidates to be imported.
7. The user should then choose the Storage Domains which he/she wants to import and press the ok button.
8. Once the Storage Domain has been imported, the user should attach the Storage Domain to an initialized Data Center and activate the Storage Domain.
9. After the Storage Domain is activated, go to the Storage main tab and pick the Storage Domain which was activated a moment ago.
10. In the same Storage main tab, the user should see two sub tabs, "Import VMs" and "Import Tempaltes", in the "Import VMs" sub tab, the user should see all the VMs which are candidates to be imported, and in the "Import Tempaltes" sub tab, there should be the same only for templates.
11. The user can pick several VMs (or Templates), and press on the "import" button.
12. When the "Import" button is pressed, a dialog should be opened, showing the list of all the entities the user chose to register.
The user should choose a cluster for each entity which should be compatible for it.
The user can also watch the entity properties (such as disks, networks) in the sub tab inside the dialog.

#### Work flow for Import File Storage Domain - UI flow

<iframe width="300" src="//youtube.com/embed/YbU-DIwN-Wc" frameborder="0" align="right" allowfullscreen="true"> </iframe> On import a File Device Storage Domain The user should do the following steps:
1. The user should press the "import Storage Domain" button.
2. The user should choose a file type domain (NFS, POSIX, etc.).
3. The user should provide the path where this Storage exists and press on the import button.
4. Once the Storage Domain has been imported, the user should attach the Storage Domain to an initialized Data Center and activate the Storage Domain.
5. After the Storage Domain is activated, go to the Storage main tab and pick the Storage Domain which was activated a moment ago.
6. In the same Storage main tab, the user should see two sub tabs, "Import VMs" and "Import Templates", in the "Import VMs" sub tab, the user should see all the VMs which are candidates to be imported, and in the "Import Tempaltes" sub tab, there should be the same only for templates.
7. The user can pick several VMs (or Templates), and press on the "import" button.
8. When the "Import" button is pressed, a dialog should be opened, showing the list of all the entities the user chose to register.
The user should choose a cluster for each entity which should be compatible for it.
The user can also watch the entity properties (such as disks, networks) in the sub tab inside the dialog.

#### Work flow for recovery of local Data Center - UI flow

<iframe width="300" src="//youtube.com/embed/T03ai6FrMI4" frameborder="0" align="right" allowfullscreen="true"> </iframe> On import a Local Storage Domain The user should do the following steps:
1. The user should first must initialize a Local Storage Domain.
2. The user should press the "import Storage Domain" button.
3. The user should choose a file type domain Data/ Local on Host.
4. Once the Storage Domain has been imported, the user should attach the Storage Domain to the Data Center he has created and initialized.
5. After the Storage Domain is activated, go to the Storage main tab and pick the Storage Domain which was activated a moment ago.
6. In the same Storage main tab, the user should see two sub tabs, "Import VMs" and "Import Templates", in the "Import VMs" sub tab, the user should see all the VMs which are candidates to be imported, and in the "Import Tempaltes" sub tab, there should be the same only for templates.
7. The user can pick several VMs (or Templates), and press on the "import" button.
8. When the "Import" button is pressed, a dialog should be opened, showing the list of all the entities the user chose to register.
The user should choose a cluster for each entity which should be compatible for it.
The user can also watch the entity properties (such as disks, networks) in the sub tab inside the dialog.

#### Work flow for importing GlusterFS Storage Domain - UI flow

<iframe width="300" src="//youtube.com/embed/4YKXHp8wxvI" frameborder="0" align="right" allowfullscreen="true"> </iframe> On import a GlusterFS Storage Domain The user should do the following steps:
1. The user should press the "import Storage Domain" button.
2. The user should choose a file type domain Data/GlusterFS on Host.
3. Once the Storage Domain has been imported, the user should attach the Storage Domain to an initialized Data Center .
4. After the Storage Domain is activated, go to the Storage main tab and pick the Storage Domain which was activated a moment ago.
5. In the same Storage main tab, the user should see two sub tabs, "Import VMs" and "Import Templates", in the "Import VMs" sub tab, the user should see all the VMs which are candidates to be imported, and in the "Import Tempaltes" sub tab, there should be the same only for templates.
6. The user can pick several VMs (or Templates), and press on the "import" button.
7. When the "Import" button is pressed, a dialog should be opened, showing the list of all the entities the user chose to register.
The user should choose a cluster for each entity which should be compatible for it.
The user can also watch the entity properties (such as disks, networks) in the sub tab inside the dialog.

#### Mockups

The following UI mockups contain guidelines for the different screens and wizards related for file Storage Domains:
The user flow for importing NFS Storage Domain, will be similar to importing Export/ISO domain.
The user will enter the path of the storage domain and will start the import process.
 An import screen for NFS Storage Domain :
![](ImportNFS.jpeg "fig:ImportNFS.jpeg")
An import screen for POSIX Storage Domain :
![](ImportPosix.jpeg "fig:ImportPosix.jpeg")
An import screen for Gluster Storage Domain :
![](ImportGluster.jpeg "fig:ImportGluster.jpeg")

The following UI mockups contain guidelines for the different screens and wizards related to the block domain:
An import screen for Fibre Channel Storage Domain :
![](FibreChannel.png "fig:FibreChannel.png")
An import screen for iSCSI Storage Domain :
![](Iscsi.jpeg "fig:Iscsi.jpeg")
Import VM/Template sub-tabs
![](import_vm_template_subtab.png "fig:import_vm_template_subtab.png")
Import VM/Template Dialog
![](import_vm_template_dialog.png "fig:import_vm_template_dialog.png")

### REST

#### Import block Storage Domain

##### Discover the targets in your iSCSI Storage Server

      POST /api/hosts/052a880a-53e0-4fe3-9ed5-01f939d1df66/iscsidiscover
      Accept: application/xml
      Content-Type: application/xml

<action>
`  `<iscsi>
           

<address>
iscsi.server

</address>
`  `</iscsi>
          `<iscsi_target>`iqn.iscsi.120.01`</iscsi_target>`  
`    `<iscsi_target>`iqn.iscsi.120.02`</iscsi_target>
`   `<iscsi_target>`iqn.iscsi.120.03`</iscsi_target>
</action>

##### Get a candidates Storage Domains list to be imported

After the iscsilogin operation, the host is already connected to the targets in the iSCSI and we can fetch the Storage Domains which are candidates to be imported.

      POST /api/hosts/052a880a-53e0-4fe3-9ed5-01f939d1df66/unregisteredstoragedomainsdiscover HTTP/1.1
      Accept: application/xml
      Content-type: application/xml

<action>
`   `<iscsi>
             

<address>
iscsiHost

</address>
`   `</iscsi>
`   `<iscsi_target>`iqn.name1.120.01`</iscsi_target>
`   `<iscsi_target>`iqn.name2.120.02`</iscsi_target>
`   `<iscsi_target>`iqn.name3.120.03`</iscsi_target>
</action>

The response which should returned as a list of Storage Domains, as follow:

<action>
`   `<iscsi>
             

<address>
iscsiHost

</address>
`   `</iscsi>
`   `<storage_domains>
`       `<storage_domain id="6ab65b16-0f03-4b93-85a7-5bc3b8d52be0">
`           `<name>`scsi4`</name>
`           `<type>`data`</type>
`           `<master>`false`</master>
`           `<storage>
`               `<type>`iscsi`</type>
`               `<volume_group id="OLkKwa-VmEM-abW7-hPiv-BGrw-sQ2E-vTdAy1"/>
`           `</storage>
`           `<available>`0`</available>
`           `<used>`0`</used>
`           `<committed>`0`</committed>
`           `<storage_format>`v3`</storage_format>
`       `</storage_domain>
`   `<status>
`       `<state>`complete`</state>
`   `</status>
`   `<iscsi_target>`iqn.name1.120.01`</iscsi_target>
`   `<iscsi_target>`iqn.name2.120.02`</iscsi_target>
`   `<iscsi_target>`iqn.name3.120.03`</iscsi_target>
</action>

##### Import the block Storage Domains to the setup

      POST /api/storagedomains/ HTTP/1.1
      Accept: application/xml
      Content-type: application/xml

<storage_domain id="39baf524-380e-407c-8625-50709fcaa9c2">
`  `<import>`true`</import>
`  `<host id="052a880a-53e0-4fe3-9ed5-01f939d1df66" />
`  `<type>`data`</type>
`  `<storage>
`     `<type>`iscsi`</type>
`  `</storage>
</storage_domain>

#### Import NFS Storage Domain

Importing a Storage Domain requires a POST request, with the storage domain representation included, sent to the URL of the storage domain collection.
 POST /api/datacenters/01a45ff0-915a-11e0-8b87-5254004ac988/storagedomains HTTP/1.1

      Accept: application/xml
      Content-type: application/xml

` `<storage_domain>
`   `<name>`data1`</name>
`   `<type>`data`</type>
`   `<host id="052a880a-53e0-4fe3-9ed5-01f939d1df66"/>
`   `<storage>
`     `<type>`nfs`</type>
           

<address>
10.35.16.2

</address>
`     `<path>`/export/images/rnd/maor/data9`</path>
`   `</storage>
` `</storage_domain>

The API creates an NFS data storage domain called data1 with an export path of 10.35.16.2:/export/images/rnd/maor/data9 and sets access to the storage domain through the hypervisor host.
The API also returns the following representation of the newly created storage domain resource:
![](Screenshot_from_2014-11-13_14-34-19.png "fig:Screenshot_from_2014-11-13_14-34-19.png")

#### Attach a Storage Domain

      POST /api/datacenters/01a45ff0-915a-11e0-8b87-5254004ac988/storagedomains HTTP/1.1
      Accept: application/xml
      Content-type: application/xml

`  `<storage_domain>
`    `<name>`data1`</name>
`  `</storage_domain>

#### Get list of unregistered VM/Template

The user can get a list of all the unregistered VMs or unregistered Templates by adding the prefix ";unregistered" after the vms/Templates, in the Storage Domain.
For example to get all the unregistered VMs in the Storage Domain fa38172b-baae-4ca3-b949-95619c01ca31 the URL will be :

` `[`http://localhost:8080/ovirt-engine/api/storagedomains/fa38172b-baae-4ca3-b949-95619c01ca31/vms;unregistered`](http://localhost:8080/ovirt-engine/api/storagedomains/fa38172b-baae-4ca3-b949-95619c01ca31/vms;unregistered)

![](UnregisteredVms.png "fig:UnregisteredVms.png")
=== Register VM to a new cluster === If the user want to register a VM to the setup, then the URL should indicate register after the VM id, as follow:

      POST /api/storagedomains/xxxxxxx-xxxx-xxxx-xxxxxx/vms/xxxxxxx-xxxx-xxxx-xxxxxx/register HTTP/1.1
      Accept: application/xml
      Content-type: application/xml

<action>
`  `<cluster id='xxxxxxx-xxxx-xxxx-xxxxxx'></cluster>
</action>

![](UnregisterVM1.png "fig:UnregisterVM1.png")

#### Get all the unregistered disks in the Storage Domain

If the user want to get a list of all the floating disks in the storage domain he should use the following URL:
 <http://localhost:8080/ovirt-engine/api/storagedomains/60cec75d-f01d-44a0-9c75-8b415547bc3d/disks;unregistered> ![](ListUnregisteredDisk.png "fig:ListUnregisteredDisk.png")
=== Register an unregistered disk === If the user want to register a specific floating disks in the system he should use the following:

      POST /api/storagedomains/60cec75d-f01d-44a0-9c75-8b415547bc3d/disks;unregistered HTTP/1.1
      Accept: application/xml
      Content-type: application/xml

<disk id='8ddb988f-6ab8-4c19-9ea0-b03ab3035347'></disk>

![](RegisterDisk.png "RegisterDisk.png")

### Permissions

*   No additional permissions will be added.

### Future Work

*   Import Storage Domain : The user will be able to import a list of Storage domains all at once.
*   Adding validation for checking image corruption after importing the Storage Domain. - Mainly for sync issues with the OVF.
*   Import an Export Domain as a regular Storage Domain

### Related Bugs

*   <https://bugzilla.redhat.com/1069780>
*   <https://bugzilla.redhat.com/1069173>

### Related Features

*   OVF on any domain
*   Import Unregistered entities
*   Local Storage Domain
*   Gluster
*   PosixFs
*   Quota - The user might import disks which will extend a defined Quota in DC.

This scenario is similar to when a user enforce a quota though it already been extended. The default behaviour will treat that by letting the user still use the resources though he/she will not be able to create any more disks.

### Comments and Discussion

*   Refer to [Talk: ImportStorageDomain](Talk: ImportStorageDomain)

[ImportStorageDomain](Category:Feature) [ImportStorageDomain](Category:oVirt 3.5 Feature)
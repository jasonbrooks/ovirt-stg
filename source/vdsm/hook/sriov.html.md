---
title: sriov
authors: dyasny, knesenko
wiki_title: VDSM-Hooks/sriov
wiki_revision_count: 3
wiki_last_updated: 2014-01-14
---

The sriov vdsm hook enables SRIOV support in oVirt.

The hook works as follows:

*   receives a VF via its os nic names, i.e. `sriov=eth5`
*   gets its pci address
*   detaches it from the os
*   creates an xml representation of the device for libvirt domain
*   adds it to the guest xml

syntax:

      sr-iov: sriov=eth10,eth11

Will attach 2 sr-iov VFs to vm

## RHEL6 sr-iov notes:

===================

*   Enable IOMME:
    -   Intel CPU, pass intel_iommu=on to the kernel command line

      (dmesg | grep "Intel-IOMMU: enabled" # make sure that iommu is enabled)

*   -   AMD CPU

<!-- -->

*   Load sr-iov PF and VF
    -   Intel Corporation 82576 Gigabit Network Connection:

      modprobe igb max_vfs=7 

7 is max for 82576 card

Please install **vdsm-hook-sriov** rpm from the following repository: <http://ovirt.org/releases/nightly/rpm/EL/6/noarch>

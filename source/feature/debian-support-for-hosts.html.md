---
title: Debian support for hosts
category: feature
authors: sandrobonazzola
wiki_category: Feature|Debian support for hosts
wiki_title: Features/Debian support for hosts
wiki_revision_count: 1
wiki_last_updated: 2015-03-11
feature_name: Debian support for hosts
feature_modules: vdsm,ioprocess
feature_status: NEW
---

# Debian support for hosts

### Summary

Add support for Debian (or similar) hosts

### Owner

*   Name: [Simone Tiraboschi](User:Stirabos)
*   Email: <stirabos@redhat.com>

### Detailed Description

*   Support building of host related rpms on Debian
*   Create Jenkins jobs for automated build and testing on Debian
*   Create Debian Jenkins slaves
*   Verify that all the components have no regressions only due to Debian

### Benefit to oVirt

It will be possible to add Debian hosts to an oVirt datacenter

### Dependencies / Related Features

*   All host related subprojects must support Debian
*   A tracker bug has been created for tracking issues:

### Documentation / External references

*   [oVirt build on debian/ubuntu](Ovirt_build_on_debian/ubuntu)
*   [Guest Agent on Debian](Debian/GuestAgent)
*   [How to install the guest agent in Debian](How to install the guest agent in Debian)
*   [VDSM on Debian](VDSM on Debian)

### Testing

The whole [Test Case](http://www.ovirt.org/Category:TestCase) collection must work when hosts are running Debian.

### Contingency Plan

The feature is self contained: if support for Debian won't be ready for 3.6.0 we won't deliver Debian packages.

### Release Notes

      == Debian Support for Hosts ==
      Support for running oVirt Hosts on Debian (or similar) has been added providing custom packaging of needed dependencies.

### Comments and Discussion

*   Refer to [Talk:Debian support for hosts](Talk:Debian support for hosts)

[Debian support for hosts](Category:Feature) [Debian support for hosts](Category:oVirt 3.6 Proposed Feature) [Debian support for hosts](Category:Integration)

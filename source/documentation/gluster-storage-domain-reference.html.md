---
title: Gluster Storage Domain Reference
category: documentation
authors: fsimonce
wiki_category: Documentation
wiki_title: Gluster Storage Domain Reference
wiki_revision_count: 1
wiki_last_updated: 2014-05-21
---

# Gluster Storage Domain Reference

## Suggested Gluster Volume Configuration for Storage Domain

In order to use a gluster volume as an oVirt Storage Domain we suggest to:

*   use the gluster replica 3 (three copies)
*   set the network.ping-timeout to 10 seconds
*   set the cluster.quorum-type to auto

Volume Info:

      Volume Name: (...)
      Type: Replicate
      Volume ID: (...)
      Status: Started
      Number of Bricks: 1 x 3 = 3
      Transport-type: tcp
      Bricks:
      (...three bricks...)
      Options Reconfigured:
      network.ping-timeout: 10
      cluster.quorum-type: auto

This configuration will prevent a split-brain on the files that are accessed in read-write mode from multiple hosts (e.g. sanlock ids file).

The commands used to configure the options are:

      # gluster volume set `<volume_name>` network.ping-timeout 10
      # gluster volume set `<volume_name>` cluster.quorum-type auto

Bug [BZ#1066996](https://bugzilla.redhat.com/show_bug.cgi?id=1066996) still generated in some cases a split-brain on sanlock files even with the proper configuration. It is suggested to use a gluster build where this bug is solved (when it will be available).

<Category:Documentation>

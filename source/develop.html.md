---
title: Develop
category: developer
authors: dneary, jbrooks, ykaplan
wiki_category: Developer
wiki_title: Develop
wiki_revision_count: 8
wiki_last_updated: 2012-11-30
---

# Develop
{:.hidden}

<section class="row">

<section class="col-md-4">

## Projects

oVirt is a [collection of projects](Architecture) which work together to provide a complete data center virtualization solution.

ovirt-engine
: oVirt Engine allows you to configure your network, storage, nodes and images. oVirt Engine also provides a command line interface tool (ovirt-engine-cli) and a RESTful API (ovirt-engine-api), including a Python wrapper (ovirt-engine-python-sdk) which allow developers to integrate management functionality into shell scripts of third party applications.

VDSM
: The Virtual Desktop and Server Management daemon runs on oVirt managed nodes, and allows oVirt to remotely deploy, start, stop and monitor virtual machines running on the node.

ovirt-node
: oVirt Node is just enough operating system to run virtual machines. It is also possible to convert a standard Linux distribution into a node which can be managed by ovirt-engine by installing VDSM and other dependencies.

dwh and reports
: The reports and data warehouse components for ovirt-engine are optional, and are packaged and developed separately.

_More information on [oVirt subprojects](Subprojects)_

</section>


<section class="col-md-4">

## Developer documentation

- [Install nightly snapshot](Install nightly snapshot)
- [Building oVirt engine](Building oVirt engine)
- [Building oVirt Node](Node Building)
- [Building VDSM](Vdsm Developers)
- [Contributing to the Node project](Contributing to the Node project)
- [Submitting a patch with Gerrit](Working with oVirt Gerrit)
- [The development process](DevProcess)
- [Release management](Release process)
- [Getting in contact with the oVirt community](Communication)
- [Becoming a maintainer](Becoming a maintainer)
- [oVirt architecture](Architecture)
- [Feature Roadmap oVirt 3.6](OVirt 3.6 Release Management)
  (see also old roadmaps for
  [oVirt 3.5](OVirt 3.5 release-management),
  [oVirt 3.4](OVirt 3.4 release-management#Features), and
  [oVirt 3.3](OVirt 3.3 release-management#Features))
- [Building a custom user portal](Sample user portals)
- [Building oVirt engine DWH](OVirt DWH development environment)
- [Building oVirt engine Reports](OVirt Reports development environment)

</section>


<section class="col-md-4">

## oVirt Architecture

[![oVirt architecture](Overall-arch.png)](images/wiki/Overall-arch.png)

</section>
</section>

---
title: Optaplanner
category: feature
authors: adahms, msivak
wiki_category: Feature
wiki_title: Features/Optaplanner
wiki_revision_count: 55
wiki_last_updated: 2015-01-21
---

# Optaplanner integration with scheduling

### Owner

*   Feature Owner: Martin Sivák: [msivak](User:msivak)
*   Email: <msivak@redhat.com>

### Current status

*   Tech preview release - 0.3 - will be part of oVirt 3.5 RC2
*   Last updated: -- by [ WIKI}}](User:{{urlencode:{{REVISIONUSER}})
*   Tracking BZs: [rhbz#1093051](https://bugzilla.redhat.com/show_bug.cgi?id=1093051)

### Use Cases

*   A virtual machine fails to start due to insufficient resources, and the user wants to know if there is a way to fit the virtual machine
*   The user wants to rebalance the cluster to achieve an "optimal" state

### Benefit to oVirt

Our users will get hints about how to utilize their hardware better.

# **Goals**

*   We want to write a service that takes a snapshot of a cluster (list of hosts and Vms) and computes an optimized Vm to Host assignment solution.
*   Optimization will be done on per Cluster basis.
*   We intend to implement the service using rules on top of [Optaplanner](http://www.optaplanner.com) engine.
*   The administrator should then be able to use that as a hint to tweak the situation in the cluster to better utilize resources.
*   The idea is to have the service free-running in an infinite loop and improve the solution over time while adapting to changes in the cluster.
*   The service will be separated from the ovirt-engine to not endanger the datacenter by using too much memory or cpu. It will talk to the engine using an API.

# Deployment manual

Two hosts (or virtual machines) are needed - one will host the ovirt-engine and the other will contain the ovirt-optimizer service. Your ovirt-engine must already be installed and configured before you perform the following steps.

Four packages are currently available (the latest build is available from <http://jenkins.ovirt.org/job/ovirt-optimizer_master_create-rpms_merged/>):

*   ovirt-optimizer-0.3-3.fc19.noarch.rpm
*   ovirt-optimizer-ui-0.3-3.fc19.noarch.rpm
*   ovirt-optimizer-jboss7-0.3-3.fc19.noarch.rpm
*   ovirt-optimizer-jetty-0.3-3.fc19.noarch.rpm

There are also packages for CentOS 6 and Fedora 20. Fedora 20 provides Jetty deployment only, because Fedora 20 ships with WildFly, which is not supported at the moment.

### Installing the ovirt-optimizer machine

*   Install the ovirt-optimizer-jetty or ovirt-optimizer-jboss7 package depending on which application server you want to use.
*   Edit the /etc/ovirt-optimizer/ovirt-optimizer.properties file and set the address of your ovirt-engine instance and the credentials for the REST API.
*   Check if the firewall allows external access to the port where your application server runs (8080 on TCP if you are using Jetty).
*   If you performed a fresh installation of Jetty on Fedora 19, you must remove the demonstration configuration file for Jetty to start - /usr/share/jetty/start.d/900-demo.ini
*   Start the application server - both Jetty and Jboss should detect and deploy the ovirt-optimizer service automatically.
*   Check the logs (depends on the application server configuration) and you should see that ovirt-optimizer detected some cluster and tries to compute a solution.

### Installing the UI

*   Switch to your ovirt-engine machine.
*   Install the ovirt-optimizer-ui package.
*   Configure the user interface plug-in by updating /etc/ovirt-engine/ui-plugins/ovirt-optimizer-config.json - you must put the address and port of the ovirt-optimizer service there.
*   Restart the ovirt-engine service and reload the Administration Portal.
*   Log in to the Administration Portal, navigate to the main Cluster tab and select a cluster. You will now see the oVirt Optimizer subtab in the lower half of the page.
*   When you switch to the subtab, it should load some data (might take couple of seconds).

### Present Features

*   Each cluster has a new subtab - Optimizer results - that shows the proposed optimized solution (both the final state and the steps to get there) and makes it possible to start a migration by clicking the relevant buttons.
*   Each virtual machine has two new elements in their context menu - Optimize start and Cancel start optimization. These elements are designed to be used with stopped virtual machines (status Down) and tell the optimizer to identify a solution when the selected virtual machine are started. The cancel menu item cancels this request. You can select multiple virtual machines this way. Sadly, there is no indication in the user interface of the currently optimized virtual machines. However, the result subtab provides a list of virtual machines that are supposed to be started together with the solution details.
*   The solution should obey the cluster policy to some extent - OptimalForEvenDistribution, OptimalForEvenGuestDistribution and OptimalForPowerSaving will be computed using the memory assignments though (the engine uses CPU load)

### Missing Features

*   Some hard constraint rules are missing so the solution might not be applicable because of the current scheduling policy.
*   Balancing check is missing so the engine might decide to touch the cluster in the middle of your optimization steps - you can disable automatic balancing in the scheduling policy to prevent this.
*   No CPU load based rules, the optimizer tries to use the hosts' memory in an even way (engine uses CPU load in the balanced rule).

# Known issues

### Data refresh failed: undefined

Check your Firefox (or other browser) version. There is a chance that your browser is new enough and enforces mixed content security rules. That blocks the request to get results from the optimizer. See <https://developer.mozilla.org/en-US/docs/Security/MixedContent> for details.

You can work around this in Firefox by going to <about:config> page and setting security.mixed_content.block_active_content to false.

### java.lang.OutOfMemoryError

Please check the amount of memory available to the application server.

Jboss is configured in /usr/share/jbossas/bin/standalone.conf (or the respective file in /usr/share/ovirt-engine-jboss-as) and look for the -Xmx option in JAVA_OPTS.

         example: JAVA_OPTS="-Xms2048m -Xmx8192m -XX:MaxPermSize=256m -Djava.net.preferIPv4Stack=true"

# Detailed Description - Internals

This feature will allow the user to get a solution to his scheduling needs. Computing the solution might take a long time so an [Optaplanner](http://www.optaplanner.org) based service will run outside of the engine and will apply a set of rules to the current cluster's situation to get an optimized VM to Host assignments.

### Getting the cluster situation to Optaplanner

This will be based on REST API after the missing entities for Cluster policy are implemented. In the worst case we might consider to utilize direct database access to get missing data.

The following information is needed:

*   List of hosts in the cluster with information about provided resources
*   List of all VMs running in the cluster with information about required resources (probably without statistics data for now)
*   The currently selected cluster policy is needed together with the custom parameters
*   The configuration of the cluster policy - list of filters, weights and coefficients

The ideal situation would be if it was possible to get the described data in an atomic snapshot way. Which means that all the data will be valid at one single exactly defined moment in time.

The Optaplanner service will use Java SDK to get the data and the idea is to get the data once and then (cache and) reuse them during the whole optimization run.

### Representing the solution in Optaplanner

Optaplanner requires at least two java classes to be implemented:

*   Solution description -- a set of all VMs with assigned Hosts.
*   Mutable entity for the optimization algorithm to update -- the VM itself, the mutable field is the Host assignment

The VM and Host classes can be represented in couple of ways:

*   SDK's Host and VM classes
*   Engine internal Vds and Vm classes

The selected representation will depend on the way our Optaplanner service will define the rules for the optimization algorithm (see the next sections).

### Reporting the result of optimization

The result will be presented using an UI plugin in the engine's webadmin. That way the user will have comfortable access to the results from a UI he is used to. Also the authentication and access management to REST will be provided by the webadmin. The disadvantage is that the UI plugin will have to use some kind of new protocol (REST, plain HTTP, …) to talk to the Optaplanner service. Also the UI plugin will have to authenticate to the Optaplanner service, but we are probably not implementing that in the tech preview phase.

There is also a question of how to represent the solution:

*   The first prototype will just show a table (graph, image) of the "optimal" VM to Host assignments in a dialog window
*   in the future we might be even able to tell the user the order of steps he should perform (migrate A to B…) to reach the state we are showing him

### Rules to select the optimal solution (high level overview)

All optimization tasks need to know how does a possible solution look like and how to select the best one. The main task we are trying to accomplish is:

1.  consolidate the free resources -- It should do a defragmentation of free memory or spare cpu cycles so more or big VMs can be started. The extreme case is our Power Saving policy as its side-effect is that a lot of hosts end up totally free of VMs. But we do not want to load the hosts that much. The actual rules that will describe this are yet to be determined, but we are currently looking into using only the hard constraints (filters) of the currently selected cluster policy.

There are two situations that should be avoided in the computed solution:

1.  Unstable cluster -- what I mean here is that once the user performs the changes to get the cluster to the "optimal" state, the engine's internal balancing kicks in and rearranges the cluster differently. It would mean that the output of the optimization algorithm was not as smart as we wanted it to be. This might be mitigated a bit by using the result of balancer as part of the scoring mechanism.
2.  Impossible solution -- if the user gets a solution from the optimization algorithm and then finds out that the cluster policy prevents him from reaching it, we will have an issue that the theoretical solution can be useless to the user. We will probably start by telling the user to disable load balancing while he is trying to reach "a better" state.

In the case where no solution can be found (for example to the start VM case) we should inform the user that there is no solution with the current cluster policy rules, but that solution to the optimization can still be found if the rules are relaxed a bit.

It is my opinion that the cluster policy rules reflect the actual user's requirements and we should obey them and make sure all solutions are valid in that context.

### Future enhancements

There are couple of other optimization tasks for us to consider in the future. We might then even allow the user to select the desired task from a list, if we later decide that more than one is useful:

1.  score according to the currently selected cluster policy -- The rationale here is that when VMs are started one by one then the assignment might be suboptimal, because the scheduling algorithm has no knowledge about the VMs that are yet to start. ([Example](#Example_of_suboptimal_balancing_as_a_result_of_starting_VMs_one_by_one)) If we base our rules on the current cluster policy we might be able to compute a solution that takes all running VMs into account at once. This approach will then use:
    -   filters as source for hard constraint score
    -   weights for the soft constraint score
    -   balancers can be possibly added to the scoring system to detect if the solution is stable (no migration will be triggered in that state) or not (engine will want to migrate something)
    -   Another metric we should use here is the necessary number of actions to change the current situation to the computed "optimal" solution.

2.  find a place for new VM -- This should try to rebalance a cluster in such a way that a VM that is not running can be started. It is closely related to the first option except it needs to know what resources should be reserved or ideally what VM is supposed to be started (we may offer a list of stopped VMs for the user to select from).

### Implementation details

We implemented the engine using Drools rule language. The other two options are here only to document the design process.

#### Reusing the existing engine's PolicyUnits

This approach will use the existing ovirt-engine rpm on the OptaPlanner machine (just the files, no daemon or engine-setup necessary). The OptaPlanner service will then be implemented as Jboss module that requires some of the engine modules (rest mappers, common, bll, scheduling). The scheduling classes will be decoupled from the DbFacade using interface (DaoProviderInterface, see <http://gerrit.ovirt.org/#/c/26199/> and <http://gerrit.ovirt.org/#/c/26200/>) and so will be reusable in the scoring mechanism of OptaPlanner.

We already have a template to base this on. Our engine-config and engine-manage-domains tools use this approach.

Advantages are:

*   scoring uses exactly the same code as the scheduling in engine and that guarantees that the solution is 100% valid in the engine as well
*   no code duplication, the PolicyUnits are already implemented and in use
*   PolicyUnits can be easily enabled/disabled in the scoring function depending on the cluster policy

Disadvantages are:

*   Users might be used to Drools rule language and might not be willing to use Java for extending the functionality
*   A REST to common mapping will have to be prepared (already part of the engine though) to map Java SDK classes to Vds, Vm and other classes that are used in PolicyUnits
*   Java modules have lower performance than drools' rule files
*   Installing the ovirt-engine RPM file can pull unnecessary dependencies (yum install jboss-ass on CentOS 6 pulls 246MB and ovirt-engine additional 211MB of packages)
*   If somebody renames the classes in the chain (Vds, VM, REST mappers, ...) the scoring app will have to be updated (but only if the ovirt-engine RPM changes on the Optaplanner machine)

#### Writing the rules in the Drools rule language

This approach will require that we copy the logic from our Java code to drools rules as exactly as possible. Any difference might cause the found solution to not be applicable to the actual cluster. The rules will have to follow strict naming scheme so we can enable/disable them according to the currently selected cluster policy.

Advantages:

*   Users might already be using Drools for other business logic purposes
*   Better performance
*   Can probably use Java SDK classes directly
*   Smaller footprint (does not need the ovirt-engine)

Disadvantages:

*   Code duplication -- the rules will have to be kept in sync with engine's PolicyUnits or we might compromise the fitness of computed solutions
*   Enabling/disabling rules according to cluster policy might not be easy

#### Using the external scheduler infrastructure for scoring

This idea is based on our ovirt-scheduler-proxy infrastructure. It would require us to implement our PolicyUnits in python and decouple them from the REST API to be able to pass the cached data there. OptaPlanner would then have to be able to execute the Python filters and weights to perform scoring.

Advantages:

*   We would get fully working external scheduling for future use
*   Python is easy to write and read
*   It would also support scheduling plugins provided by the customer

Disadvantages:

*   Code duplication again
*   Decoupling the plugins from REST is not trivial, there is no API to pass the required information to the proxy together with the scheduling task
*   Lower performance
*   If the user is not using Python modules only, then the optimization won't find the proper solution. This is an issue, because we need some information (pending memory) that is currently not exposed by the SDK

# Examples and demostrations

### Example of suboptimal balancing as a result of starting VMs one by one

This example uses just a single resource and evenly balanced policy. At the end all the Hosts should be providing the same amount of that resource to the VMs.

There are 4 VMs and 2 hosts:

1.  Vm_A: 330MB of memory
2.  Vm_B: 330MB of memory
3.  Vm_C: 330MB of memory
4.  Vm_D: 1000MB of memory

Now lets investigate what happens when the starting order is A,B,C,D:

![](Abcd_1.png "Abcd_1.png")

![](Abcd_2.png "Abcd_2.png")

![](Abcd_3.png "Abcd_3.png")

![](Abcd_4.png "Abcd_4.png")

And as a second case D, C, B, A:

![](Dcba_1.png "Dcba_1.png")

![](Dcba_2.png "Dcba_2.png")

![](Dcba_3.png "Dcba_3.png")

![](Dcba_4.png "Dcba_4.png")

Notice that the second case is much better with regards to equal balancing, but has less free space for a new VM. It is necessary to determine the priorities without guessing to select the proper solution according to user's needs.

### Screenshots of the UI plugin in version 0.3

When there is nothing that needs to be done in the cluster, you will see something similar to this:

![](No-actions.png "No-actions.png")

After a VM start is requested, the display will reflect that a VM is being scheduled and give you the chance to start the VM on the computed dectination host:

![](Start-vm.png "Start-vm.png")

Starting the VM changes the status (the icon is missing here, but will be present in the version):

![](Starting-vm-1.png "Starting-vm-1.png")

VM started successfully. It is still visible here, but will disappear from the list after the next result refresh (the optimizer needs some time to update the solution with the new state):

![](Vm-up.png "Vm-up.png")

### Compute a "complicated" start VM solution

This demonstration shows a situation where the starting VM does not directly fit to any host. The first picture shows that all hosts are partially occupied and there is no host with 1.5 GB of free RAM that is needed for the VM we are about to start.

![](Before-solution.png "Before-solution.png")

When optimizer kicks in the following solution is found. One of the small VMs is migrated and the freed space is used to start the big VM.

![](Start-solution.png "Start-solution.png")

# Comments and Discussion

*   Refer to <Talk:Optaplanner>

<Category:Feature> <Category:SLA>

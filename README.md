# pansi

### What problems does pansi target, and in what ways?

Problem 1: Ansible's documentation defines patterns for idempotently maintaining set numbers of hosts on IAAS providers such as AWS and Rackspace using attributes or tags to maintain desired counts for each type of host (specifically, the pattern of running a cloud module as a 'local_action:' and then iterating with 'add_host:' to get the cloud hosts into live inventory). However, the natural progression of deploying with this model is to statically write these host tags and counts into the playbook for each type of service that needs to be deployed. Further, if some instances of a service are on different providers or need to run different numbers of each type of host, the playbook for that specific service instance will then itself have multiple versions. Playbook upkeep and task duplication under this approach can become an issue even with a relatively small number of services deployed.

  - The main goal of pansi is to document a pattern in which the variables that define the host-layout of a service can be stored in a variables file and "passed" (by the playbook for the service being deployed) to a set of generalized IAAS roles that will do the actual work with Ansible's cloud modules. Or in other words it is an abstraction designed to separate the playbooks that create and deploy the service from the variables that distinguish it from other services of the same type. Under this approach, each service only needs one playbook to deploy and maintain all instances of itself.

Problem 2: Statically writing the 'local_action:' cloud module tasks into the playbooks that deploy your services (or, slightly better, storing those tasks separately and importing them) makes it cumbersome to move services or parts of services to other providers.

  - Storing the parameters that define a service's layout separately allows them to be edited more easily and much less (if any) editing be done to actual playbooks.

Problem 3: Adding separate tooling (Terraform, perhaps) to your systems to solve these problems can take time and may often get blocked by more immediate tasks.

  - pansi can be used as a stepping stone here because it has conceptual similarities to these "declarative infrastructure" tools, and because it consists purely of Ansible concepts and processes it is more quickly understood and implemented.

### What is the downside to this approach?

A small portion of the overhead that would have gone into maintaining a playbook for each instance of a service is 
 redirected into maintaining the variable files that hold the service parameters. However, this is still much less 
 overhead than the alternative because many tasks like moving portions of a service (or entire services) to another 
 provider become a matter of changing a few parameters instead of rewriting portions of the playbook.
 #FIXME: reinforce; find a more compelling way to demonstrate this point ^^

One important point is that this overhead is **NOT** the same as the overhead one would incur by trying to maintain a 
standard static inventory file, a custom static inventory file that generates a standard static inventory file, 
trying to keep any static inventory file in sync with dynamic inventory scripts, or trying various other solutions. 
The distinction lies in that pansi stores what types of nodes a service requires, how many there are of each type, 
and what service-specific variables there might be, but knows next to nothing about any individual host (thereby 
leaving that job to dynamic inventory scripts). This is a critical distinction for the purposes of staying on 
the 'cattle' side of the traditional pets v. cattle analogy.

### What does the name mean?

Deploying named instances of services onto providers is kind of PAAS-like; pansi is an assortment of 
letters from the phrase "PAAS in Ansible" run together.

### Definitions

**NOTE**: Many of these terms are intentionally misspelled. The purpose of this is to help
 maintain manageability. For example, the first term below 'servis' is merely a misspelling
 of 'service'. The misspelled option is more easily located and distinguished in codebases,
 which often use these common terms in many different contexts. Should a term ever need to
 be renamed or modified procedurally, the find/replace or other operation will be much
 easier on the human running it and there may be 100 instances of the misspelled term to
 sort through instead of (in some cases) thousands of unrelated instances in the case of
 using a generic term spelled correctly.

**servis**: A 'service', or 'deployment'. A servis is a server (or servers working
 together) to do an assigned job such as run builds, host a package cache, store backups,
 host a database or web frontend, or etc.

**servisName**, and **servisType**: Each servis has a "servisName" and is an instance of its
 "servisType", just as a traditional programming object is an instance of its declaration.
 Servis names and types are semi-arbitrary so long as they are consistent in the couple
 of places they are defined. A servis named 'debby' might be of the servisType 'apt-cache',
 while a servis named 'who' might be of the servisType 'bind9-deployment'.

**servisParams**: A data structure that stores these other parameters that define a servis,
 which takes the form of an Ansible group_vars file named for the servisName.

**node**: A servis consists of node(s); each node is a host/instance/VM. (This term is
 practically interchangeable with Ansible's term 'host'; the only reason 'node' has not
 already been replaced with 'host' is to see if it is helpful to distinguish pansi
 resources from other Ansible resources).

 Nodes are placed into groups in Ansible's live inventory based on a list of groups
 defined in the servisParams, and Ansible plays and roles are run on the nodes
 depending on their group membership. (This is normal Ansible functionality, the
 difference is the method used to add the nodes to their groups dynamically from the
 groups list)

**provider**: Nodes are provided by providers. These are Infrastructure as a Service (IAAS)
 providers such as EC2 and Rackspace. Providers' names are semi-arbitrary, as long as they
 match in the couple of places they are defined in pansi. Providers are actually a reference
 to an individual tennant/project/VPC-subnet within an IAAS provider (as opposed to the
 entire IAAS company, or everything a you host there... which would be too broad). Example
 provider names might be 'awapne2' if you had a single VPC in the Asia-Pacific
 Northeast-2 region of AWS or 'raxord2' for the second of two Rackspace accounts that
 deploy into the ORD datacenter in Chicago.

**typeLabel**: A node's "typeLabel" is its primary group. The fictitious servis this file
 describes ('testar') is primarily tasked with running Jenkins, but it also hosts LDAP.
 So the typeLabel is 'jenkins', and the one "additional_group" defined is 'ldap'. Aside
 from providing a portion of the node's name in IAAS providers, distinguishing one group
 as the typeLabel is merely intended as a convenience for us humans. (testar is an instance
 of the fictitious servisType 'skylab', a name which could reflect the logistical/oversight
 aspects of the work it does.)

 The deploy order is determined by the playbook for the servisType and does not
 depend on which group is the typeLabel. The playbook deploying a skylab servis needs
 LDAP running before Jenkins can use it for auth, so the group and plays for LDAP in
 the skylab playbook are actually processed first in spite of LDAP not being those
 node's typeLabel.

 However, changing the typeLabel to a different group (whether to an added group or
 swapping it out with one of its existing additional_groups) requires redeploying the
 servis, or you'll end up with twice the intended nodes (one set with the new typeLabel,
 and one orphaned set set with the old one *which may still be serving requests*).

### Is there any other information about pansi that maybe hasn't been completely formatted yet that you could just sort of paste-vomit into this space for me to read?

These values are important to pansi:

- Creating new concepts and terms should be avoided in most cases in favor of recycling existing Ansible (or general 
"cloud-ops") concepts and terminology. For example pansi defines node's Ansible group memberships directly in the
 group_vars files that define the servisParams, rather than creating a "servisRole" or "servisComponentType"
 parameter for what each node's purpose is and then having to map the appropriate set of Ansible groups in a
 servisRole-to-ansibleGroup data structure. Calling them what they are right in the servisParams removes the need
 for this extra layer of indirection.
- As pansi itself will be seen by some as an anti-pattern anyway, it's reasonable to utilize some anti-patterns or
 generally odd methods of accomplishing its goals. **HOWEVER**, finding innovative ways to make pansi's operations
 work more like current established patterns is a high-priority goal.
- The roadmap should always include discussion of options for further removing concepts, terms, and anti-patterns 
that do not have their own application outside of pansi.

This is the current model in brief (and probably with technical omissions or misstatements):

- group_vars files that are understood to be used solely for pansi are created with a structure that defines an 
instance of a service, and any variables specific to that service.
- The name of the service to be deployed is passed on the ansible-playbook command line (-e).
- A localhost play imports the group_vars file using the service name passed on the command line.
- 'add_host:' is used to create an alias of localhost for each combination of: 'IAAS_provider:typeLabel'.
  - host vars that will be used by provider roles are passed along in the add_host task.
- The play calls the provider role(s) once for each alias of localhost.

pansi is currently part of a production deployment and includes some interesting features like replacing the 
default administrative user in the IAAS provider's disk image and switching to the new user both within a single run
of ansible-playbook, as well as deploying services divided arbitrarily across multiple IAAS providers (because of 
how Ansible's cloud modules are used, the provider roles are written specifically for pansi, but they are all 
functionally very similar to one another. EC2, Rackspace, and private OpenStack [the older nova compute module] 
are implemented thus far).

Most if not all of this functionality will be added to this repo as portions of the existing implementation 
are cleaned up and made less specific to their current use case.

### A couple of screen shots

![Untitled1.png](Untitled1.png)

![Untitled2.png](Untitled2.png)

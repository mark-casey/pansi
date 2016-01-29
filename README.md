# pansi

### What problem does pansi attempt to solve?

Ansible's documentation defines patterns for idempotently creating and maintaining specified numbers of hosts on 
IAAS providers such as AWS and Rackspace using attributes or tags to maintain desired counts for each type
 of host (specifically - the pattern of running a cloud module as a 'local_action:' and then iterating with 
 'add_host:' to get the cloud hosts into live inventory). However, it was "our" (the author[s]) finding that the next 
 place the admin will find themselves is with a playbook for each type of service they deploy that has these host tags
 and counts statically written in. Further, if some instances of a service are on different providers or need to 
 run different numbers of each type of host, the playbook for that specific service will then also have multiple
 versions. Playbook upkeep and task duplication under this approach can become an issue even with a relatively small 
 number of services deployed.

### How does pansi help with this problem?

With the above in mind, the goal of pansi is to document a pattern by which a service's parameters can be stored in an 
Ansible variables file and passed (by the playbook for the service being deployed) to a set of IAAS-specific roles 
that will do the actual work with Ansible's cloud modules. Or in other words it is an abstraction designed to 
 separate the playbooks that create and deploy the service from the variables that distinguish it from other services 
 of the same type. Under this approach, all instances of a service can be deployed and maintained by a single 
 playbook.

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

Deploying named instances of services onto providers is kind of PAAS-like; pansi is a run-together assortment of 
letters from the phrase "PAAS in Ansible".

### Is there any other important information about pansi that maybe hasn't been well formatted yet that you could just 
sort of paste-vomit into this space for me to read?

These are kind of the values of the project:

- New concepts and terms should be avoided wherever possible in favor of recycling existing Ansible (or general 
"cloud-ops") concepts and terminology.
- As pansi itself will be seen as an anti-pattern by some, it is allowed to utilize some anti-patterns or generally 
odd methods of accomplishing its goals. **HOWEVER**, finding innovative ways to make pansi's operations look more 
like current established patterns is of utmost importance.
- The roadmap should always include discussion of options for further removing concepts, terms, and anti-patterns 
that are pansi-specific.

This is the current model in brief (and probably with technical omissions or misstatements):

- group_vars files that are understood to be used solely for pansi are created with a structure that defines an 
instance of a service, and any variables specific to that service.
- The name of the service to be deployed is passed on the ansible-playbook command line (-e).
- A localhost play imports the group_vars file using the service name passed on the command line.
- 'add_host:' is used to create an alias of localhost for each: 'IAAS_provider-type_of_service-service_component'.
  - host vars specific to the component are passed along by add_host.
- The play that calls the cloud module as a 'local_action:' repeats once for each alias of localhost using the 
host vars to define the options to be used by the cloud module.

pansi is currently part of a production deployment and includes some interesting features like replacing the 
default administrative user in the IAAS provider's disk image and switching to the new user both within a single run
of ansible-playbook, as well as deploying services divided arbitrarily across multiple IAAS providers (because of 
how the Ansible's cloud modules are run, a role is written for each IAAS provider as a shim but they are all 
functionally very similar to one another. EC2, Rackspace, and private OpenStack [the older nova compute module] 
are implemented thus far).

Most if not all of this functionality will be added to this repo as portions of the existing implementation 
are cleaned up and made less specific to the current use case, with a timeline that is likely to be denominated in 
number of months (but the timeframe to a coherent, easily adoptable set of resources may well stretch longer ).


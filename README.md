# pansi

Ansible's documentation defines patterns for idempotently creating and maintaining specified numbers of hosts on 
IAAS providers such as AWS and Rackspace using attributes or tags to maintain desired counts for each type
 of host (specifically - the pattern of running a cloud module as a 'local_action:' and then iterating with 
 'add_host:' to get the cloud hosts into live inventory). However, it was the author(s) finding that the next place 
 the admin will find themselves is with a playbook for each type of service they deploy that has these host tags
 and counts statically written in. Further, if some instances of a service are on different providers or need to 
 run different numbers of each type of host, the playbook for that specific service will then also have multiple
 versions.

With this in mind the goal of pansi is to document a pattern by which a set of roles and plays can receive parameters 
of what to create on an IAAS provider **without** requiring that static inventory files exist and be kept up once 
the created hosts have been bootstrapped. It is an abstraction designed to store what a service would look like if 
you were to redeploy it from scratch, and to then hand over maintenance of inventory to Ansible's dynamic inventory 
scripts. This is important for the purpose of staying on the 'cattle' side of the traditional pets v. cattle analogy.

In this way the author(s) see the project as a PAAS-like thing for Ansible, or pansi.

These ideals are important to the project:

- New concepts and terms should be avoided wherever possible in favor of recycling existing Ansible (or general 
"cloud-ops") concepts and terminology.
- As pansi itself will be seen as an anti-pattern by some, it is allowed to utilize some anti-patterns or generally 
odd methods of accomplishing its goals. **HOWEVER**, finding innovative ways to make pansi's operations look more 
like current established patterns is of utmost importance.
- The roadmap should always include discussion of options for further removing concepts, terms, and anti-patterns 
that are pansi-specific.

The current model in brief:

- group_vars files that are understood to be used solely for pansi are created with a structure that defines an 
instance of a service, and any variables specific to that service.
- The name of the service to be deployed is passed on the ansible-playbook command line (-e).
- A localhost play imports the group_vars file using the service name passed on the command line.
- 'add_host:' is used to create an alias of localhost for each: 'IAAS_provider-type_of_service-service_component'.
  - host vars specific to the component are passed along by add_host.
- The play that calls the cloud module as a 'local_action:' repeats once for each alias of localhost using the 
host vars to define the options to be used by the cloud module.
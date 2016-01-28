# pansi

A PAAS-like thing for Ansible.

pansi is (or is becoming) a minimal PAAS-like abstraction to store codified descriptions of cloud infrastructure 
into source control for use by Ansible **without** having to track actual inventory statically/manually or 
like "pets" (when applying the usual pets v. cattle analogy).

There are established patterns for using Ansible to idempotently create and maintain specified numbers of hosts 
on IAAS providers such as AWS, Rackspace, or etc. and use attributes or tags to maintain desired counts for each type
 of host. However, it was the finding of the author(s) that there was not a great way to commit into source control 
what those host counts and attributes **actually were** and **deploy services** using that information while **still** 
utilizing Ansible's dynamic inventory scripts.

These things are important to the project:

- New concepts and terms are avoided wherever possible in favor of recycling existing terminology.

- While the goals of this project are fairly well defined, it is young enough that none of the current methods 
for achieving those goals are yet so revered that they cannot be changed. Changes will be made to methodology to 
move away from using anti-patterns and to further reduce the need for new or duplicated concepts or terms. 

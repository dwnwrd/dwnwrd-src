---
title: Using Ansible Filters to Customize the Order Of Hosts in a List
banner: /img/banners/banner-11.jpg
date: 2015-02-01
layout: post
tags: [ ansible, zimbra ]
---

Zimbra is a email / collaboration suite that is typically deployed in a cluster or clusters of dedicated servers which fill roles like LDAP master, LDAP replica, Proxy, MTA, Mailstore, etc.

The LDAP servers are used by all the other servers to store configuration and provisioning data. Servers in the cluster understand where to find the LDAP master (read/write) and LDAP replicas (read only) though values defined in `/opt/zimbra/conf/localconfig.xml`.

There are 2 values relevant to LDAP server lists and they have values like this:

```bash
ldap_master_url = "ldap://zimbra-ldap-master-01"
ldap_url        = "ldap://zimbra-ldap-01 ldap://zimbra-ldap-02 ldap://zimbra-ldap-master-01"
```

That should be easy enough to construct based on group memberships, right? Unfortunately there is a bit of complexity lurking here. LDAP replica servers should always list themselves first in the `ldap_url`, and the `ldap_url` should end with an LDAP master. LDAP master servers should always list themselves first in `ldap_master_url`.

This is what I came up with.

The result is 2 fact variables for each host: `zimbra_ldap_url` and `zimbra_ldap_master_url`. Those facts can later be applied with the `zmlocalconfig` command. (I wrote a Ansible module to do that as well. Maybe I will be able to post that at some point.)

```yaml
---
# file: roles/zimbra/tasks/zimbra-define-ldap-urls.yml
# Just set facts: zimbra_ldap_master_url, zimbra_ldap_url

# ldap_master_url and ldap_url are used by all zimbra servers,
# but zimbra LDAP servers need to always be first in their own list

#------------------------
# LDAP Master URL
# If I am an LDAP master, I should be first value. Everything else shuffled.

- name: Shuffled list of LDAP masters other than me
  set_fact: 
    ldap_master_sans_me: "{{ groups['zimbra-ldap-master'] | difference(ansible_fqdn) | shuffle }}"

- name: Define LDAP Master URL for Masters
  set_fact:
    zimbra_ldap_master_url: 'ldap://{{ [ ansible_fqdn ] | union(ldap_master_sans_me) | join(" ldap://") }}'
  when: '"zimbra-ldap-master" in group_names'

- name: Define LDAP Master URL for Non-masters
  set_fact:
    zimbra_ldap_master_url: 'ldap://{{ ldap_master_sans_me | join(" ldap://") }}'
  when: '"zimbra-ldap-master" not in group_names'


#------------------------
# LDAP URL
# If I am an LDAP server, I should be first value. Everything else shuffled, and masters should come last.

- name: Shuffled list of LDAP replicas other than me
  set_fact: 
    ldap_replica_sans_me: "{{ groups['zimbra-ldap-replica'] | difference(ansible_fqdn) | shuffle }}"

- name: Define LDAP URL for LDAP replicas
  set_fact:
    zimbra_ldap_url: 'ldap://{{ [ ansible_fqdn ] | union(ldap_replica_sans_me) | join(" ldap://") }} {{ zimbra_ldap_master_url }}'
  when: '"zimbra-ldap-replica" in group_names'

- name: Define LDAP URL for LDAP masters
  set_fact:
    zimbra_ldap_url: 'ldap://{{ [ ansible_fqdn ] | union(ldap_replica_sans_me) | union(ldap_master_sans_me) | join(" ldap://") }}'
  when: '"zimbra-ldap-master" in group_names'

- name: Define LDAP URL for non-LDAP servers
  set_fact:
    zimbra_ldap_url: 'ldap://{{ ldap_replica_sans_me | join(" ldap://") }} {{ zimbra_ldap_master_url }}'
  when: '"zimbra-ldap" not in group_names'
```

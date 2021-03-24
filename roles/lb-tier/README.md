LB Tier
=========

A role to install haproxy to load balance traffic to backend application servers.

Requirements
------------

None

Role Variables
--------------


| variable name                 | purpose                                        | default value |
|-------------------------------|------------------------------------------------|---------------|
| lb_tier_application_groupname | Name of a group of application backend servers | *apps*        |

Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - name: setup load-balancer tier
      hosts: frontends
      gather_facts: false
      become: yes
      roles:
        - {name: lb-tier, tags: [lbs, haproxy]}

License
-------

BSD

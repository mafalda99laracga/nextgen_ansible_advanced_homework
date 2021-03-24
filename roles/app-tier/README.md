App Tier
=========

Ansible Role to:

- install server prerequisites for Tomcat application
- deploy Tomcat application

Requirements
------------

None

Role Variables
--------------

None

Dependencies
------------

None

Example Playbook
----------------

Example usage:


    - name: setup app tier
      hosts: apps
      become: yes
      gather_facts: false
      roles:
        - {name: app-tier, tags: [apps, tomcat]}

License
-------

BSD

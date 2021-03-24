DB Tier
=========

Role to install postgres server

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

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:


    - name: setup database tier
      become: yes
      gather_facts: false
      hosts: dbs
      roles:
        - {name: db-tier, tags: [dbs, postgres]}

License
-------

BSD

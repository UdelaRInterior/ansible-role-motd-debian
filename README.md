Role MotD for debian
=========

Debian motd management, as [described in official wiki](https://wiki.debian.org/motd), seems broken, and so the ansible roles existing in the Galaxy. But the motd can still be managed: 
* a static first part in the static `/etc/motd` file,
* a dynamic script generated part, with .sh scripts in `/etc/profile.s/`. 

This role manages this motd in a debian, building: 
* a static part build by text fragments, that can be included as such or processed by `figlet` for an ASCII art
* a dynamic part installing scripts based on templates. Some templates are proposed and new ones can be built (PRs welcome)

Requirements
------------

A debian host installed

Role Variables
--------------

`motd_deb_motd_file_structure` structure describes the /etc/motd to build. Items with `build: figlet` are processed by figlet (that the role can install in local controller), those with `build: figlet` are included as such. 

`motd_deb_scripts` structure describe the scripts to be installed in `/etc/pofile.d/`. For the moment ther is only a very simple `fortune | cowsay`, but complex scripts, from templates with ansible parameters, can bi imagined. 

Defalut values of these variables are: 

```
motd_deb_motd_file_structure:
- build: figlet
  fragment: '{{ inventory_hostname.split(".",1) | first | capitalize }}.'
- build: figlet
  fragment: '{{ inventory_hostname.split(".",1) | last }}'
- build: text
  fragment: '{{ lookup( "file", "{{ role_path }}/files/motd" ) }}'

motd_deb_scripts:
- file: 20-cow_fortune
  template: cow_fortune.j2 
  dependencies:
  - cowsay
  - fortunes
  - fortunes-{{ ansible_facts.env.LANGUAGE[:2] }}
```

Dependencies
------------

no dependencies

Example Playbook
----------------

Just call motd_debian on a debian host: 

    - hosts: all
      roles:
         - role: udelarinterior.motd_debian

License
-------

GPLv3

Author Information
------------------

Daniel Viñar Ulriksen @ulvida, [Red de Unidades Informáticas de la UdelaR en el Interior](https://github.com/udelarinterior)
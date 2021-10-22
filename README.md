Role MotD for debian
=========

Debian motd management, as [described in official wiki](https://wiki.debian.org/motd), seems broken, and so the ansible roles existing in the Galaxy. But the motd can still be managed by: 
* a static first part in the static `/etc/motd` file,
* a dynamic script generated part, with .sh scripts in `/etc/profile.d/`. 

This role manages this motd in a debian, building: 
* a static part build by text fragments, that can be included as such or processed by `figlet` for an ASCII art
* a dynamic part installing scripts based on templates. Some templates are proposed and new ones can be built (PRs welcome)

Requirements
------------

A debian host installed

Role Variables
--------------

`motd_deb_motd_file_structure` describes the structure of the /etc/motd to build. It's a list of hashes, which keys can be:
* `fragment` is the text of the static part,
* `build:` which presently accepts `text` or `figlet`:
  * hashes with `build: figlet` are processed by figlet (that the role can automatically install in local controller),
  * hashes with `build: text` are included as such.
* `lolcat:` a boolean that indicates if the result is processed with lolcat o have a raimbow display (the role can also automatically install lolcat in the local controller). However, this option is still not working for static motd, only for scripts.

`motd_deb_scripts` structure describe the scripts to be installed in `/etc/pofile.d/`. For the moment ther is only a very simple `fortune | cowsay | lolcat`, but complex scripts, from templates with ansible parameters, can bi imagined. 

`motd_deb_scripts` is also a list of hashes, which keys can be: 
* `file:` a filename (without `.sh`) for the script to be uploaded at `/etc/profile.d/`. If omited, the role tries to extract the namefile of the template and it sets `default` as the script file name. 
* `template:` the template file to build the script depending on playbook parameters
* `dependencies:` a list of packages that need to be installed in the host to run the script
* `lolcat:` a boolean that indicates if the result is processed with lolcat o have a raimbow display
* any other hash key: we can define any kind of parameters to be used in the thmplate 

Defalut values of these variables are: 

```
motd_deb_motd_file_structure:
- build: figlet
  fragment: |
      {{ inventory_hostname.split(".",1) | first | capitalize }}.
      {{ inventory_hostname.split(".",1) | last }}
  lolcat: true
- build: text
  fragment: '{{ lookup( "file", "{{ role_path }}/files/motd" ) }}'

motd_deb_scripts:
- file: 20-cow_fortune
  template: cow_fortune.j2 
  dependencies:
  - cowsay
  - fortunes
  - fortunes-{{ ansible_facts.env.LANGUAGE[:2] }}
  fortunes_dir: /usr/share/games/fortunes/
  fortunes:
  - file: '{{ ansible_facts.env.LANGUAGE[:2] }}'
    percent: 60%
  - file: .
    percent: 40%
  lolcat: true
```
The only script template we presently have is cow_fortune.j2, which contains: 
```
#!/bin/sh
# A welcome saying
 
/usr/games/fortune \
  {% for motd_fortune in motd_deb_curr_script.fortunes -%}
  {{ motd_fortune.percent }} {{ motd_deb_curr_script.fortunes_dir }}{{ motd_fortune.file }} {% endfor %}\
  | /usr/games/cowsay
  {%- if motd_deb_curr_script.lolcat %}| /usr/games/lolcat {% endif %}
```
This generates the following shell message of the day, where the cow is displayed with raimbow colors and the say she says is randomly generated, 60% in local language, 40% in english: 
```
ulvida@paname:~/tech/interior/config$ ssh root@pavada
Linux pavada 5.4.78-2-pve #1 SMP PVE 5.4.78-2 (Thu, 03 Dec 2020 14:26:17 +0100) x86_64
 ____                      _         
|  _ \ __ ___   ____ _  __| | __ _   
| |_) / _` \ \ / / _` |/ _` |/ _` |  
|  __/ (_| |\ V / (_| | (_| | (_| |_ 
|_|   \__,_| \_/ \__,_|\__,_|\__,_(_)
                                     
 _       _            _                    _                    
(_)_ __ | |_ ___ _ __(_) ___  _ __ ___  __| |_   _  _   _ _   _ 
| | '_ \| __/ _ \ '__| |/ _ \| '__/ _ \/ _` | | | || | | | | | |
| | | | | ||  __/ |  | | (_) | | |  __/ (_| | |_| || |_| | |_| |
|_|_| |_|\__\___|_|  |_|\___/|_|(_)___|\__,_|\__,_(_)__,_|\__, |
                                                          |___/ 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Oct 22 09:50:45 2021 from 2001:1328:6a:5::1008
 _________________________________
/ Los pueblos que tienen memoria, \
| progresan.                      |
|                                 |
\ -- Anónimo.                     /
 ---------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
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
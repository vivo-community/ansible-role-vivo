[![CI](https://github.com/vivo-community/ansible-role-vivo/workflows/molecule-ci/badge.svg?event=push)](https://github.com/vivo-community/ansible-role-vivo/actions?query=workflow%3Amolecule-ci)


Ansible role to install VIVO
=========

Ansible role to install and configure [VIVO](https://github.com/vivo-project/VIVO) - 
the extensible semantic web application for research discovery and showcasing scholarly work.
It follows the directions given in the [Installing VIVO](https://wiki.lyrasis.org/display/VIVODOC112x/Installing+VIVO) guide.


Example Playbook
----------------

This example is taken from `molecule/default/converge.yml`.
```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: true

  roles:
    - role: vivo_community.vivo
```

Requirements
------------
VIVO has some [system requirements](https://wiki.lyrasis.org/display/VIVODOC112x/System+Requirements) 
that need to be installed beforehand. In CI this is done using `molecule/default/prepare.yml`:
```yaml
- name: prepare
  hosts: all
  become: yes
  gather_facts: no

  roles:
    # prepare instance for ansible
    - role: robertdebock.bootstrap
    - role: robertdebock.core_dependencies

    # install java jdk
    - role: robertdebock.java
      java_vendor: openjdk
      java_type: jdk
      java_default_version: "8"

    # install solr with no initial cores
    # but don't start it (solr is not started until vivocore is in place for discovery)
    - role: geerlingguy.solr
      solr_version: "8.11.1"
      solr_cores:
      solr_service_state: stopped
      solr_restart_handler_enabled: false
      solr_service_manage: false

    # install tomcat
    # and set CATALINA_OPTS="-Xms512m -Xmx512m -XX:MaxPermSize=128m"
    # see https://wiki.lyrasis.org/display/VIVODOC112x/Installing+VIVO#InstallingVIVO-ConfigureandStartTomcat
    - role: robertdebock.tomcat
      tomcat_version: 9
      java_opts:
        - name: CATALINA_OPTS
          value: "-Xms512m -Xmx512m -XX:MaxPermSize=128m"

    # install maven
    - role: gantsign.maven
```


Role Variables
--------------

The default values for the variables are set in `defaults/main.yml`:
```yaml
# solr settings - defaults from geerlingguy.solr
vivo_solr_home: /var/solr/data             # solr_home
vivo_solr_user: solr                       # solr user
vivo_solr_group: solr                      # solr group
vivo_solr_service_name: solr               # service name to restart solr if handler is enabled
vivo_solr_restart_handler_enabled: true    # enable or disable solr restart
```
The vivo-solr core will be placed below vivo_solr_home and vivo_solr_user and vivo_solr_group will be set as file owners,
so core discovery will find it after a solr restart. To restart solr the service name is needed.
If you need to make additional adjustments to solr and don't want it to restart yet, you can disable the restart handler.

```yaml
# tomcat settings - defaults from robertdebock.tomcat
vivo_tomcat_user: tomcat                   # tomcat user
vivo_tomcat_group: tomcat                  # tomcat group
vivo_tomcat_service_name: tomcat           # service name to restart tomcat if handler is enabled
vivo_tomcat_restart_handler_enabled: true  # enable or disable tomcat restart
```
VIVO will be installed into the tomcat directory (see below vivo_settings_tomcatdir) and tomcat will be restarted.
To restart tomcat the service name is needed.
If you need to make additional adjustments to tomcat and don't want it to restart yet, you can disable the restart handler.
Additionally after installation the directory permissions of tomcat webapps/logs/work/temps will be hardened 
using the tomcat user and group.

```yaml
################# VIVO SETTINGS ##################
vivo_version: 1.12.2                        # version of VIVO (currently supported: 1.12.1 and 1.12.2)
# overwrite example-settings.xml
vivo_settings_appname: vivo
vivo_settings_vivodir: /usr/local/vivo/home # dir where vivo will be installed
vivo_settings_tomcatdir: /opt/tomcat        # dir where tomcat is installed into
vivo_settings_defaulttheme: wilma
```
Currently the VIVO installation covers only version 1.12.1 and 1.12.2 and will include VIVO-languages and Vitro-languages. 
Additionally the required parameters for the settings.xml file are needed, because they will be used during installation.

Compatibility
-------------
The role was developed and tested with Ansible v2.10.

License
-------

Apache-2.0

Author Information
------------------

Sandra Mierz for [TIB](https://www.tib.eu/en/).

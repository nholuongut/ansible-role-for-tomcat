# Install and configure tomcat on your system.

![](https://i.imgur.com/waxVImv.png)
### [View all Roadmaps](https://github.com/nholuongut/all-roadmaps) &nbsp;&middot;&nbsp; [Best Practices](https://github.com/nholuongut/all-roadmaps/blob/main/public/best-practices/) &nbsp;&middot;&nbsp; [Questions](https://www.linkedin.com/in/nholuong/)
<br/>

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/nholuongut/ansible-role-for-tomcat/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: yes

  vars:
    # tomcat_address: "127.0.0.1"
    tomcat_instances:
      - name: "tomcat"
      # - name: "tomcat-version-7"
      #   version: 7
      #   shutdown_port: 8007
      #   non_ssl_connector_port: 8082
      #   ssl_connector_port: 8445
      #   ajp_port: 8011
      # - name: "tomcat-version-8"
      #   version: 8
      #   shutdown_port: 8008
      #   non_ssl_connector_port: 8083
      #   ssl_connector_port: 8446
      #   ajp_port: 8012
      # - name: "tomcat-version-9"
      #   version: 9
      #   shutdown_port: 8019
      #   non_ssl_connector_port: 8084
      #   ssl_connector_port: 8447
      #   ajp_port: 8013
      # - name: "tomcat-specific"
      #   user: "specificuser"
      #   group: "specificgroup"
      #   shutdown_port: 8020
      #   shutdown_pass: shutme
      #   non_ssl_connector_port: 8085
      #   ssl_connector_port: 8448
      #   ajp_port: 8014
      #   xms: 256M
      #   xmx: 512M
      # - name: "tomcat-with-wars"
      #   shutdown_port: 8021
      #   non_ssl_connector_port: 8086
      #   ssl_connector_port: 8449
      #   ajp_port: 8015
      #   wars:
      #     - url: https://tomcat.apache.org/tomcat-7.0-doc/appdev/sample/sample.war
      #     - url: "https://github.com/aeimer/java-example-helloworld-war/raw/master/dist/helloworld.war"
      # - name: "tomcat-java_opts"
      #   shutdown_port: 8022
      #   non_ssl_connector_port: 8087
      #   ssl_connector_port: 8449
      #   ajp_port: 8016
      #   java_opts:
      #     - name: UMASK
      #       value: "0007"
      # - name: "tomcat-with_lib"
      #   shutdown_port: 8023
      #   non_ssl_connector_port: 8088
      #   ssl_connector_port: 8450
      #   ajp_port: 8017
      #   libs:
      #     - url: "https://search.maven.org/remotecontent?filepath=io/prometheus/simpleclient/0.6.0/simpleclient-0.6.0.jar"
      # - name: "tomcat-access-logs"
      #   shutdown_port: 8024
      #   non_ssl_connector_port: 8089
      #   ssl_connector_port: 8451
      #   ajp_port: 8018
      #   access_log_enabled: yes
      #   access_log_directory: "my-logs"
      #   access_log_prefix: my-access-logs
      #   access_log_suffix: ".log"
      #   access_log_pattern: "%h %l %u %t &quot;%r&quot; %s %b"
      # - name: "tomcat-config-files"
      #   shutdown_port: 8025
      #   non_ssl_connector_port: 8090
      #   ssl_connector_port: 8452
      #   ajp_port: 8019
      #   ajp_secret: "SoMe-SeCrEt"
      #   config_files:
      #     - src: "{{ role_path }}/files/dummy.properties"
      #       dest: "./"
      #       mode: "0644"

  roles:
    - role: nholuongut.tomcat
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/nholuongut/ansible-role-for-tomcat/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: yes
  gather_facts: no

  roles:
    - role: nholuongut.bootstrap
    - role: nholuongut.core_dependencies
    - role: nholuongut.java
      # __java_version:
      #   default: 8
      #   Debian: 11
      #   Debian-bookworm: 17
      # java_version: "{{ _desired_java_version[ansible_distribution ~ '-' ~ ansible_distribution_release] | default(_desired_java_version[ansible_distribution] | default(_desired_java_version['default'])) }}"
```

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/nholuongut/ansible-role-for-tomcat/blob/master/defaults/main.yml):

```yaml
---
# defaults file for tomcat

# Some "sane" defaults.
tomcat_name: tomcat
tomcat_directory: /opt
tomcat_version: 9
tomcat_user: tomcat
tomcat_group: tomcat
tomcat_xms: 512M
tomcat_xmx: 1024M
tomcat_non_ssl_connector_port: 8080
tomcat_ssl_connector_port: 8443
tomcat_shutdown_port: 8005
tomcat_shutdown_pass: SHUTDOWN
tomcat_ajp_enabled: yes
tomcat_ajp_port: 8009
tomcat_ajp_secret: "SoMe-SeCrEt"
tomcat_jre_home: /usr
tomcat_service_state: started
tomcat_service_enabled: yes
# You can bind Tomcat to a specified address globally using this variable, or
# in the `tomcat_instances`. The `tomcat_instances.address` is more specific
# so it takes priority over `tomcat_address`.
tomcat_address: "0.0.0.0"

# Configure tomcat access logs
tomcat_access_log_enabled: yes
tomcat_access_log_directory: logs
tomcat_access_log_prefix: localhost_access_log
tomcat_access_log_suffix: ".txt"
tomcat_access_log_pattern: "%h %l %u %t &quot;%r&quot; %s %b"

# This role allows multiple installations of Apache Tomcat, each in their own
# location, potentially of different version.
# This is done by defining a "tomcat_instances" where "name:" is a unique
# identifier of an instance.
# The default tomcat_instances is one instance using the defaults described
# in defaults/main.yml.
tomcat_instances:
  - name: "{{ tomcat_name }}"
    version: "{{ tomcat_version }}"
    user: "{{ tomcat_user }}"
    group: "{{ tomcat_group }}"
    xms: "{{ tomcat_xms }}"
    xmx: "{{ tomcat_xmx }}"
    non_ssl_connector_port: "{{ tomcat_non_ssl_connector_port }}"
    ssl_connector_port: "{{ tomcat_ssl_connector_port }}"
    shutdown_port: "{{ tomcat_shutdown_port }}"
    ajp_enabled: "{{ tomcat_ajp_enabled }}"
    ajp_port: "{{ tomcat_ajp_port }}"
    ajp_secret: "{{ tomcat_ajp_secret }}"
    # You can pick an address per instance:
    # address: "127.0.0.1"
    packet_size: 8192
    java_opts:
      - name: JRE_HOME
        value: "{{ tomcat_jre_home }}"
    access_log_enabled: "{{ tomcat_access_log_enabled }}"
    access_log_directory: "{{ tomcat_access_log_directory }}"
    access_log_prefix: "{{ tomcat_access_log_prefix }}"
    access_log_suffix: "{{ tomcat_access_log_suffix }}"
    access_log_pattern: "{{ tomcat_access_log_pattern }}"
    service_state: "{{ tomcat_service_state }}"
    service_enabled: "{{ tomcat_service_enabled }}"

# The explicit version to use when referring to the short name.
tomcat_version7: "7.0.109"
tomcat_version8: "8.5.73"
tomcat_version9: "9.0.55"
tomcat_version10: "10.1.12"

# The location where to download Apache Tomcat from.
tomcat_mirror: "https://archive.apache.org"
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/nholuongut/ansible-role-for-tomcat/blob/master/requirements.txt).

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://nholuongut.nl/) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/nholuongut/ansible-role-for-tomcat/png/requirements.png "Dependencies")

The minimum version of Ansible required is 2.12, tests have been done to:

- The previous version.
- The current version.
- The development version.

# 🚀 I'm are always open to your feedback.  Please contact as bellow information:
### [Contact ]
* [Name: nho Luong]
* [Skype](luongutnho_skype)
* [Github](https://github.com/nholuongut/)
* [Linkedin](https://www.linkedin.com/in/nholuong/)
* [Email Address](luongutnho@hotmail.com)

![](https://i.imgur.com/waxVImv.png)
![](Donate.png)
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/nholuong)

# License
* Nho Luong (c). All Rights Reserved.🌟

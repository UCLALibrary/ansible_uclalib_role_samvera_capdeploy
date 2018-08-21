uclalib_role_samvera_capdeploy
=========

Ansible role to perform an initial code deploy to a UCLA Library Samvera/Hyrax system using Capistrano

Requirements
------------

This role handles the __first deployment__ of hyrax project code from a git repository to the rails application server.

Please take note of the following assumptions:
* the rails application server uses Red Hat Enterprise Linux 7 for the OS
* a Solr 7.X server is available with the index core created and project-specific configuration files installed in the core
* a Fedora 4.7.X repository server is available
* a MySQL database server is available with the project-specific database created and user account/privileges established
* a git repository is available containing the project's code
* all project-specific variables for this role should be defined in a __vars__ file with a name following the format of `projectname_envname.yml`
    * this vars file will contain sensitive information and should be encrypted with ansible-vault

Role Variables
--------------
Variables the need to be defined in the **play file** or the **host inventory file** - please note these should match the naming used for the vars file:
* `project_name` - defines the name of the rails application project - there is no default value
* `env_name` - defines the name of the deploy environment (e.g. test, stage, prod) - there is no default value

Variables with default values that **do not** need to be defined in the project vars file:
* `capistrano_user` - defines the user account that will perform the code deploy - default is `deploy`
* `capistrano_base` - defines the top-level directory where the project will be deployed - default is `/opt`

Variables that **do** need to be defined in the project vars file:
* `rails_host_fqdn` - defines the fully qualified domain name of the rails app server
* `rails_db_adapter` - defines the type of database back-end (`mysql2` or `postgresql`) - please note that currently this role only supports a db backend of `mysql2`
* `rails_db_pool` - defines the number of connections to maintain to the database
* `rails_db_host` - defines the hostname of the database server
* `rails_db_name` - defines the name of the project's database
* `rails_db_user` - defines the user account with access to the database
* `rails_db_pass` - defines the password for the database user
* `project_default_admin_password` - defines the rails app admin password that will be created at first deploy
* `contact_email_addr` - defines the email address for a project contact
* `fedora_repo_url` - defines the HTTP url to the Fedora REST interface
* `fedora_repo_user` - defines the username for Fedora access
* `fedora_repo_password` - defines the password for the Fedora user
* `solr_index_url` - defines the HTTP url to the project's Solr core
* `redis_host` - defines the hostname of the server running Redis
* `redis_port` - defines the port number where Redis is reachable
* `sidekiq_num_threads` - defines the number of Sidekiq threads to maintain
* `git_repo_url` - defines the HTTP url to the project's git repository
* `git_repo_branch` - defines the name of the project's git branch to deploy
* `ssh_pub_keys` - defines the list of ssh public keys to install in the deploy user's authorized_keys file; this allows for other users to perform deployments

An example vars file is available as a part of this role, named `exampleproj_test.yml`

Variables with default values that define if this deployment should use SSL
For a local dev deployment, default values enable SSL and install self-signed certificates
For a production install, defaults values are overriden by variables defined in host_vars
* `enable_ssl` - defines if this deloyment should use SSL (`yes` or `no` - default is `yes`)
* `ssl_cert_base_path` - defines the base path to the SSL certs and key
* `ssl_cert_file_path` - defines the path to the SSL certs
* `ssl_key_file_path` - defines the path to the SSL private key
* `ssl_files`
    * `self_signed` - defines if the certificates are self-signed (`yes` or `no` - default is `yes`)
    * `crt` - contains the contents of the SSL public certificate
    * `interm` - contains the contents of the SSL intermediate chain certificate (only needed if using a trusted cert)
    * `key` - contains the contents of the SSL private key

Dependencies
------------

The following roles must be run on the rails application server prior to executing this deploy role:

```
uclalib_role_rhel7repos
uclalib_role_epel
uclalib_role_uclalibrepo
uclalib_role_samvera_basepackages
uclalib_role_java
uclalib_role_clamav
uclalib_role_pip
uclalib_role_imagemagick
uclalib_role_libreoffice
uclalib_role_ffmpeg
uclalib_role_fits
uclalib_role_ruby
uclalib_role_apache
uclalib_role_passenger
uclalib_role_nodejs
uclalib_role_yarn
uclalib_role_redis
```

Example Playbook
----------------
```
---

- name: uclalib_capdeploy.yml
  become: yes
  become_method: sudo
  hosts: test
  user: ansible
  vars:
    project_name: "californica"
    env_name: "test"

  roles:
    - { role: uclalib_role_rhel7repos }
    - { role: uclalib_role_epel }
    - { role: uclalib_role_uclalibrepo }
    - { role: uclalib_role_samvera_basepackages }
    - { role: uclalib_role_java, oracle_java_version: '1.8.0_181' }
    - { role: uclalib_role_clamav }
    - { role: uclalib_role_pip }
    - { role: uclalib_role_imagemagick }
    - { role: uclalib_role_libreoffice, libreoffice_version: '5.4.7' }
    - { role: uclalib_role_ffmpeg, ffmpeg_version: '4.0.2' }
    - { role: uclalib_role_fits, fits_version: '1.3.0' }
    - { role: uclalib_role_ruby, ruby_version: '2.5.1' }
    - { role: uclalib_role_apache }
    - { role: uclalib_role_passenger, passenger_version: '5.3.3' }
    - { role: uclalib_role_nodejs, nodejs_version: '8' }
    - { role: uclalib_role_yarn }
    - { role: uclalib_role_redis }
    - { role: uclalib_role_samvera_capdeploy }
```

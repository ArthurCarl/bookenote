# Ansible

## Ansible server environment

### Vagrant and VB

#### Vagrant Command

```shell
# add vagrant box
vagrant box add geerlingguy/centos7 

# create virtual server configuration using the box 
vagrant init geerlingguy/centos 

# boot up server
vagrant up 

# stop server
vagrant halt

# rm vagrant box from VitualBox
vagrant destroy

# connet to vagrant server
vagrant ssh 
vagrant ssh-config
```

`Vagrantfile` vagrant config

```txt
  # provion cammond exec
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end
```

### Ansible command

- `-f` fork thread num. `ansible -i inventory/host multi -a "hostname" -f 1`
- `--become, -b` become `sudo`
- `--limit` : regrex pattern `--limit ~"*.4"` 
- `-B <seconds>` ansible job run time
- `-P <seconds>` ansible job poll interval.

## Ansible playbook

### Ansible playbook Conmmand

- `register` create new variable
- `change_when` ansible if change server

#### `handlers` : 

- run only once.
- one host faile then handler not run
- `- meta: flush_handlers` handler the handlers beheave
- `-- force-handlers` force run handlers

#### Environment variables

- edit `.bash_profile`, `shell` only
- `register` option, for further use
- var file, var in playbook, var in commandline

#### Inventory variables

- define var in host
- `group_vars` or `host_vars`

#### Register Var

- `register` store the output of a particular command in a variable at runtime
- `when: var_name` run task if `var_name` is true
- `changed_when` and `failed_when` influence Ansible's reporting when a task results in cahgnes or failures
- `ignore_errors` 

#### Delegation, Local Actions, and Pause

`delegate_to` is used for delegating a task to a particular host

```yml
- name: Add server to Munin monitoring configuration
  command: monitor-server webservers {{inventory_hostname}}
  delegate_to: "{{monitoring_master}}"
```

`local_action` delegate to localhost.

#### Pausing playbook execution with `wait_for`

`wait_for` module

#### Running an entire playbook locally

`--coonnection=local` to speed up playbook execution by avoiding the SSH connection overhead.

#### Tag

Tag can run subsets of a playbook's tasks.

```yml
---
- hosts: webservers
  tags: deploy
  roles:
    - {{role: tomcat, tags: ['tomcat','app']}}
  tasks:
    - name: Notify on completion
      local_action:
        module: osx_say
        msg: "{{inventory_hostname}} is finished!"
        voice: Zarvox
      tags: 
        - notifications
        - say
    - include: foo.yml
```

#### Blocks

group related tasks together and apply paritcular task parameters on the block level.

```yml
---
- hosts: web
  tasks:
    # Install and configure Apache on RedHat/CentOS hosts.
    - block:
        - yum: name=httpd state=present
        - template: src=httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
        - service: name=httpd state=started enabled=yes
      when: ansible_os_family == 'RedHat'
      sudo: yes
    # Install and configure Apache on Debian/Ubuntu hosts.
    - block:
        - yum: name=apache2 state=present
        - template: src=httpd.conf.j2 dest=/etc/apache2/apache2.conf
        - service: name=apache2 state=started enabled=yes
      when: ansible_os_family == 'Debian'
      sudo: yes
      rescue: 
        - name: This will only run in case of an error in the block
          debug: msg="There was an error in the block."
      always: 
        - name: This will only always run, no matter what.
          debug: msg="This always excutes."
```
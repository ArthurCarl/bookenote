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
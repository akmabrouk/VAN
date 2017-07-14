# Vagrant_Ansible_Nginx

This is a repository for experimenting the combination of Vagrant, Ansible and Nginx.

## Design
![Design](https://lh3.googleusercontent.com/XDzGYwN1FIZzW5pAtJJauwuQL9bUGDi5oax2URKPHw7CKUe0KwXonqNU-rrCRFNGXzNMr3db1QE0HW0jvti5H9Y1158AI90Z378R1BqWe_T2XpycwphWVq71PMSUgRKJw3NmNOXx0dfEmhm_GKUSj0Ly3UA97UrpMNS9_p3_dGFo7tuewyOm4Oui8wMj1tyTzunXm5KtIskK4KI-FsQ3-wUuL5Ps8mDIgfEsO_-m8DsIsJNzVLNWgdvfdrXgn89eoTSg8a_MX3aTwC4S8DUkPIeKGpWbaLH-3ptGb6dnY0EyCd6PpsMvtnFUOSN-XzufP6-TCOViCSZbujWZvxvB7tvB0pC2LI8IzMlytZ0vHlBCJP6MVwoa5F8N4PYO4hX1xr4OQ0nhd4O0Jb76mlKZMfmxaKqTwyiI130sw7MNw9iB05Q5xhxnIdzhifdWN9LhBeyjx-57jC-uCSquCuvxsiykcnQT1xqNvfZbPSCLHZ1rNtZgJIM8j8JLjsi0k6svaJUn6yaM0fiEHWujJNeHLCQ2iB_wkkG-G0UFWH9B78aYo_pGl1gh_k9sc4NJMfIiEzB0hBYB9WY4zfyW5zMx_RlqXp8CJAKPBeV4KRe2zrf8LGtgmWE8Glg-2FUTUEHEN0mVqQ1cTfKWoNzGgmiAmR1VoMlzQKHEmWIOI1EJOQPqUg=w1366-h768-no)

## Prerequisite

For this project, it is required to install Vagrant locally either on a mac/windows OS. As for Ansible, it is not required to install the tool because it will be all executed inside the VM images.

## How to use

First clone or download the repository and go the the directory where the following command can be executed to get this setup running. 

```
Vagrant up
This command creates and configures guest machines according to your Vagrantfile.

Vagrant provision
Runs any configured provisioners against the running Vagrant managed machine.

Vagrant destroy
This command stops the running machine Vagrant is managing and destroys all resources that were created during the machine creation process. After running this command, your computer should be left at a clean state, as if you never created the guest machine in the first place.
```

Once the machines are up and playbook is fully played, go checkout the URL: http://192.168.88.10. If you refresh it few times, it balance between application server 192.168.88.11 and 192.168.88.12.

## Vagrantfile
The syntax of the vagrantfile is based on Ruby. It is simple setup for two application servers thats serves the loadbalancer. The application servers can be scaled from two to three simply by changing the values: (1..2).each do |i| into (1..3).each do |i|. 

To understand the Ruby each Iterator go to [Tutorialspoint](https://www.tutorialspoint.com/ruby/ruby_iterators.htm).

```
Vagrant.configure("2") do |config|
        config.vm.define "loadbalancer" do |loadbalancer|
          loadbalancer.vm.box = "puppetlabs/ubuntu-14.04-64-nocm"
          loadbalancer.vm.network :private_network, ip: "192.168.88.10"

          loadbalancer.vm.provider :virtualbox do |v|
            v.name = "loadbalancer"
            v.customize [
                "modifyvm", :id,
                "--name", "loadbalancer",
                "--memory", 512,
                "--natdnshostresolver1", "on",
                "--cpus", 1,
            ]
            end

        config.vm.provision :ansible_local do |ansible|
          ansible.playbook = "./ansible/playbook.yml"
        end
    end

(1..2).each do |i|
        config.vm.define "application_#{i}" do |application|
          application.vm.box = "puppetlabs/ubuntu-14.04-64-nocm"
          application.vm.network :private_network, ip: "192.168.88.1#{i}"

          application.vm.provider :virtualbox do |v|
            v.name = "application_#{i}"
            v.customize [
                "modifyvm", :id,
                "--name", "application_#{i}",
                "--memory", 512,
                "--natdnshostresolver1", "on",
                "--cpus", 1,
            ]
            end

        config.vm.provision :ansible_local do |ansible|
          ansible.playbook = "./ansible/playbook.yml"
        end
    end
  end
end
```
As a reminder, ansible is used within the VM images, which means the Vagrant Ansible Local provisioner allows it to provision the guest using Ansible playbooks by executing ansible-playbook directly on the guest machine.

## Playbook
Remember to scale up the application server in the playbook if the vagrantfile is also scaled up by a numbers. 

```
---
- hosts: loadbalancer
  become: true
  vars_files:
    - vars/all.yml
  roles:
    - server
    - nginx_proxy

- hosts: application_1
  become: true
  vars_files:
    - vars/all.yml
  roles:
    - server
    - nginx
    - php

- hosts: application_2
  become: true
  vars_files:
    - vars/all.yml
  roles:
    - server
    - nginx
    - php
    - test
```

## Roles
These roles are a further level of abstraction that can be useful for organizing playbooks. As you add more and more functionality and flexibility to your playbooks, they can become difficult to maintain as a single file. Roles allow you to create very minimal playbooks that then look to a directory structure to determine the actual configuration steps they need to perform.

For this project the roles were created (with tasks, handlers and templates) to serve the loadbalancer and the application servers. 

- Server
Includes update_cache to ensures the indexes are synchronized with the sources list and installation curl, wget and vim for test/troubleshooting purposes.

- Nginx_proxy
Includes installation of Nginx (latest version) and round-robin template. This also include restaring handler for nginx. 
See more options at [Loadbalancing types with Nginx](http://nginx.org/en/docs/http/load_balancing.html). 

- Nginx
Includes the installation of the latest Nginx and its template for setting up the config with restarting handler for Nginx. 

- Php
Includes the installaton of php php5-fpm, php5-mcrypt, php5-gd and php5-curl and a restarting handler for php5-fpm. 

- Test
Testing connectivity with ports 80 to the loadbalancer and the application servers.  

## Improvements

The goal is to improve the current design on the following points:

- [X] Explanatories on every line of code with a hash-mark for understandability
- [X] Add inventories for replicating deployment environments
- [X] Add a debug handler with block module 
- [X] Add a report module
- [X] Add more test module
- [X] Add tags for organizing and testing purposes

## License

MIT License. See the [LICENSE file](LICENSE) file for details.

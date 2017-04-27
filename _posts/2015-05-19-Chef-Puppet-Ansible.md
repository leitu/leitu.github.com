## Chef VS Puppet VS Ansible VS osquery

update on Mar 8, update facebook/osquery

###Overview

This is test trail on Mac, in order to run ansbile you need to install [sshpass](http://www.hashbangcode.com/blog/installing-sshpass-osx-mavericks)

###Compare

Here is the result with same method to create a file and test for the running speed with standalone.

####chef

```
file '/tmp/test-chef' do
  owner 'atu'
  content 'Hellow chef'
  action :create
end

chef-apply hello.rb  2.72s user 1.87s system 94% cpu 4.863 total
```
####puppet

```
class test {
  file { '/tmp/test-puppet' :
    ensure  => present,
    content => 'hello puppet',
  }
}

puppet apply site.pp --modulepath=`pwd`  1.75s user 0.78s system 44% cpu 5.750 total
```

####ansible
```
more hosts
[all]
127.0.0.1 ansible_ssh_user=atu ansible_ssh_pass=xxxxx

more hello.yml
---
- hosts: all
  tasks:
  - name: create test file
    file: path=/tmp/test-ansible state=touch
  - name: add content
    lineinfile: dest=/tmp/test-ansible line="Hello Ansible"
    
ansible-playbook -i hosts hello.yml  0.27s user 0.23s system 9% cpu 5.021 total
```


####osquery

Now we are having brand new from facebook, osquery.

```bash
osqueryi "select version from kernel_info" --json
[
  {"version":"15.3.0"}
]
```

##Ohai vs Facter vs Ansible_os_Facts
###Ohai
```
ohai ipaddress
[
  "10.75.106.119"
]
```
###Facter
```
time facter ipaddress
10.75.147.40
facter ipaddress  0.08s user 0.05s system 34% cpu 0.379 total
```
###Cfacter
After puppet 3.7.x, it introduced cfater.
```
time /opt/puppetlabs/bin/cfacter ipaddress
10.0.2.15

real	0m0.030s
user	0m0.018s
sys	0m0.004s
```
###Ansible
```
time ansible all -i hosts -m setup -a 'filter=ansible_all_ipv4_addresses'

127.0.0.1 | success >> {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.75.106.119"
        ]
    },
    "changed": false
}

ansible all -i hosts -m setup -a 'filter=ansible_all_ipv4_addresses'  0.17s user 0.10s system 2% cpu 10.182 total
```

as you can see, cfacter is much faster than any other tools.


```
ansible --version
ansible 2.0.0

time ansible local -m setup -a 'filter=ansible_all_ipv4_address'
127.0.0.1 | SUCCESS => {
    "invocation": {
        "module_name": "setup",
        "module_args": {
            "filter": "ansible_all_ipv4_address"
        }
    },
    "verbose_override": true,
    "changed": false,
    "ansible_facts": {}
}

real	0m1.983s
user	0m0.160s
sys	0m0.044s

```

much faster than v1, after Redhat acquire Ansible, hopefully it will be more than great.

###OSquery


```bash
time osqueryi "select * from interface_addresses" --json
[
  {"address":"::1","broadcast":"","interface":"lo0","mask":"ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff","point_to_point":"::1"},
  {"address":"127.0.0.1","broadcast":"","interface":"lo0","mask":"255.0.0.0","point_to_point":"127.0.0.1"},
  {"address":"fe80::1","broadcast":"","interface":"lo0","mask":"ffff:ffff:ffff:ffff::","point_to_point":""},
  {"address":"fe80::9a5a:ebff:fecb:de93","broadcast":"","interface":"en3","mask":"ffff:ffff:ffff:ffff::","point_to_point":""},
  {"address":"10.75.106.145","broadcast":"10.75.106.255","interface":"en3","mask":"255.255.255.0","point_to_point":""}
]
0.03s user 0.01s system 87% cpu 0.048 total
```
It's extremly fast than the other facter tool.




##Extention
###Facter

if I need use the data with facter

require 'facter'
puts "kernel: #{Facter.value(:operatingsystem)}"

irb(main):009:0> op = Facter.value(:operatingsystem)
=> "Darwin"

it's easy to get the value

###Ohai


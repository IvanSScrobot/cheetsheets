## Inventory
General: https://www.ansible.com/ansible-inventory-for-fun-and-profit (slides: https://www.ansible.com/hubfs//AnsibleFest%20ATL%20Slide%20Decks/Fest%202019%20-%20Ansible%20Inventory%20for%20Fun%20and%20Profit.pdf)

Inventory Plugins: https://www.ansible.com/managing-meaningful-inventories (slides: https://www.ansible.com/hubfs//AnsibleFest%20ATL%20Slide%20Decks/AnsibleFest%202019%20-%20Managing%20Meaningful%20Inventories.pdf)

[Ansible Custom Inventory Plugin - a hands-on, quick start guide](https://termlen0.github.io/2019/11/16/observations/)

[Creating custom dynamic inventories for Ansible](https://www.jeffgeerling.com/blog/creating-custom-dynamic-inventories-ansible) (Jeff Geerling)

[Developing dynamic inventory](http://people.cs.uchicago.edu/~kauffman/brocade/icx/ansible_modules/dev_guide/developing_inventory.html)

[How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#how-variables-are-merged)

## How to pass a password
```
read -p "Password: " -s pw; export pw
ansible  ... -e ansible_password='{{ lookup("env", "pw") }}'
```

## How to pass a dictionary as an extra variable
```
ansible-playbook play.yml -e '{"test":["1.23.45", "12.12.12"]}'
```

## Error: "The specified collections path is not part of the configured Ansible collections paths" 
in ansible.cfg, add 
```
[defaults]
collections_paths = ./collections/ansible_collections
```

## Developing modules

[Developing plugins](https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html#inventory-plugins)

```
#!/usr/bin/python
 
from ansible.module_utils.basic import *
from subprocess import Popen, PIPE
import urllib2,socket
 
def main():
 
 
    fields = {
        "url": {"required": True, "type": "str" },
 
    }
    response={}
    module = AnsibleModule(argument_spec=fields)
    try:
        urllib2.urlopen(module.params['url'],timeout=10.0)
        socket.setdefaulttimeout(10.0)
        response={"status":"Able to reach successfully"}
        module.exit_json(changed=True, meta={"status": response},msg="VPN connectivity is healthy")
    except Exception as e:
        response={ "Reason":"Unable to reach endpoint under VPN , Please check the connectivity" , "url": module.params['url']}
        module.fail_json(msg="failed to run",meta= response ) 
 
if __name__ == '__main__':
    main()

```

Then call it from ansible (e.g. from main.yml):
```
- name: Checking VPN connectivity
  urlcheck:
    url: "https://{{ url }}"
  register: _responce_
```

## to get all mount points:

https://stackoverflow.com/questions/74866028/get-list-of-filesystems-and-mountpoints-using-ansible 

```
- hosts: localhost
  gather_facts: false
  tasks:
    - setup:
        gather_subset:
          - distribution
          - mounts
    - debug:
        var: ansible_distribution
    - debug:
        var: ansible_mounts
```

## To disable host key checking:
export ANSIBLE_HOST_KEY_CHECKING=False

## The right way to limit hosts

--limit='~sparkworker(1[1-9]|2[0-9]|3[0-6])-${fqdn},localhost'

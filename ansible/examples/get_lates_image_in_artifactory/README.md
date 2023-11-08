## Ansible module:
```
#!/usr/bin/python
 
DOCUMENTATION = '''
---
module: artifactory_get_latest
short_description: Searches on provided Artifactory web page for latest version of an artifact.
requirements: []
description:
   Searches through provided Artifactory Repository web page for latest version of an artifact.
   Artifact is identified by regex pattern.
   Example of path: https://docker-release.artifactory.com/ui/native/repo/microservice_name/
   Example of pattern: '1.1.2-SNAPSHOT-[\d]*\.[\d\w-]*'
options:
    path:
        required: true
        description:
            - path in Artifactory repository to search
    pattern:
        required: true
        description:
            - Python regex expression to extract an artifact name from path in Artifactory
author:
    - "Ivan Skrobot"
'''
from ansible.module_utils.basic import *
import re
## Adding condition for python version 2 and 3
if int(sys.version[0]) == 2:
        from urllib import *
elif int(sys.version[0]) == 3:
    from  urllib.request  import *
 
def find(url, pattern):
    http_response = urlopen(url)
    filtered_list = re.findall(pattern, str(http_response.read()))
     
    if not filtered_list:
        return ""
    else:
        return(filtered_list[-1])
 
def main():
    fields = {
        "path": {"required": True, "type": "str" },
        "pattern": {"required": True, "type": "str" }
    }
    module = AnsibleModule(argument_spec=fields)
     
    result = dict(
        changed=False,
        name=''
    )
     
    result['name'] = find(module.params['path'], module.params['pattern'])
    module.exit_json(**result)
 
 
if __name__ == '__main__':
    main()
```


## Ansible code

```

- name: "Pull {{ docker_image_name }} docker image from Artifactory"
  block:
    - name: Try to pull a specific version of {{ docker_image_name }} docker image from Artifactory
      docker_image:
        name: "{{docker_registry_url}}/{{ microservice_name }}"
        tag: "{{ image_tag }}"
        source: pull
        debug: yes
        force_source: yes
      become: yes
  rescue:
 
    - name: find the latest image in Artifactory
      artifactory_get_latest:
        path: "{{ gr_artifactory }}/{{ repo  }}//{{ microservice_name }}/"
        pattern: '{{ version }}-[\d]*\.[\d\w-]*'
      register: latest_tag
 
    - debug:
        var: latest_tag.name
 
    - set_fact:   
        image_tag: "{{ latest_tag.name }}"
 
    - debug:
        var: image_tag
 
    - name: Try to pull the latest version of {{ microservice_name }} docker image from Artifactory
      docker_image:
        name: "{{docker_registry_url}}/{{ microservice_name }}"
        tag: "{{ latest_tag.name }}"
        source: pull
        debug: yes
        force_source: yes
      become: yes
```

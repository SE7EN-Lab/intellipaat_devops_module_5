#### Ansible hands-on

| Type | Host Name | OS    |
| :--: | :---:     | :---: |
| Ansible Master | ec2-65-2-73-231.ap-south-1.compute.amazonaws.com | Amazon Linux 2 |
| Ansible slave-1 | ec2-65-0-71-50.ap-south-1.compute.amazonaws.com | Amazon Linux 2 |
| Ansible slave-2 | ec2-3-108-64-248.ap-south-1.compute.amazonaws.com | Amazon Linux 2 |

1. Ansible version on master
```
[ec2-user@ip-172-31-15-156 ansible]$ ansible --version
ansible 2.9.25
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ec2-user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.18 (default, Jun 10 2021, 00:11:02) [GCC 7.3.1 20180712 (Red Hat 7.3.1-13)]
```
2. Create a custom inventory file
```
[ec2-user@ip-172-31-15-156 ansible]$ ls -ltr inventory 
-rw-rw-r-- 1 ec2-user ec2-user 179 Nov  5 13:47 inventory

[ec2-user@ip-172-31-15-156 ansible]$ ansible-inventory -i inventory --list

{
    "_meta": {
        "hostvars": {
            "ec2-3-108-64-248.ap-south-1.compute.amazonaws.com": {
                "ansible_user": "ec2-user"
            }, 
            "ec2-65-0-71-50.ap-south-1.compute.amazonaws.com": {
                "ansible_user": "ec2-user"
            }
        }
    }, 
    "all": {
        "children": [
            "apache_servers", 
            "nginx_servers", 
            "ungrouped"
        ]
    }, 
    "apache_servers": {
        "hosts": [
            "ec2-65-0-71-50.ap-south-1.compute.amazonaws.com"
        ]
    }, 
    "nginx_servers": {
        "hosts": [
            "ec2-3-108-64-248.ap-south-1.compute.amazonaws.com"
        ]
    }
}

```
3. Ansible Role initialization
```
[ec2-user@ip-172-31-15-156 ansible]$ ansible-galaxy init web --offline
- Role web was created successfully
[ec2-user@ip-172-31-15-156 ansible]$ tree web
web
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

8 directories, 8 files
```
4. Role definition for Apache HTTPD (web)
5. Playbook execution on apache_servers group
```
ec2-user@ip-172-31-15-156 ansible]$ ansible-playbook -i inventory site.yml

PLAY [apache_servers] ******************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
[WARNING]: Platform linux on host ec2-65-0-71-50.ap-south-1.compute.amazonaws.com is using the discovered Python interpreter at /usr/bin/python, but future  
installation of another Python interpreter could change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more 
information.
ok: [ec2-65-0-71-50.ap-south-1.compute.amazonaws.com]

TASK [web : install httpd] *************************************************************************************************************************************
ok: [ec2-65-0-71-50.ap-south-1.compute.amazonaws.com]

TASK [web : install java] **************************************************************************************************************************************
ok: [ec2-65-0-71-50.ap-south-1.compute.amazonaws.com]

TASK [web : show java version] *********************************************************************************************************************************
changed: [ec2-65-0-71-50.ap-south-1.compute.amazonaws.com]

TASK [configure webpage] ***************************************************************************************************************************************
changed: [ec2-65-0-71-50.ap-south-1.compute.amazonaws.com]

TASK [web : start httpd service] *******************************************************************************************************************************
ok: [ec2-65-0-71-50.ap-south-1.compute.amazonaws.com]

RUNNING HANDLER [web : restart server] *************************************************************************************************************************
changed: [ec2-65-0-71-50.ap-south-1.compute.amazonaws.com]

PLAY RECAP *****************************************************************************************************************************************************
ec2-65-0-71-50.ap-south-1.compute.amazonaws.com : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
#### Validation
 - Hit http://ec2-65-0-71-50.ap-south-1.compute.amazonaws.com/home.html
 - SSH to ec2-65-0-71-50.ap-south-1.compute.amazonaws.com
 - Launch the below commands
```
[ec2-user@ip-172-31-15-174 html]$ java -version
openjdk version "1.8.0_312"
OpenJDK Runtime Environment (build 1.8.0_312-b07)
OpenJDK 64-Bit Server VM (build 25.312-b07, mixed mode)

[ec2-user@ip-172-31-15-174 html]$ httpd -version
Server version: Apache/2.4.51 ()
Server built:   Oct  8 2021 22:03:47
``` 

6. Role definition for NGINX (web-ng)
```
[ec2-user@ip-172-31-15-156 ansible]$ ansible-galaxy init web-ng --offline
[ec2-user@ip-172-31-15-156 roles]$ tree web-ng
web-ng
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

8 directories, 8 files
```
7. Playbook execution on nginx_servers group
```
[ec2-user@ip-172-31-15-156 ansible]$ ansible-playbook -i inventory site.yml 
PLAY [apache_servers] ****************************************************************************************************************************************
TASK [Gathering Facts] ***************************************************************************************************************************************
[WARNING]: Platform linux on host ec2-65-0-71-50.ap-south-1.compute.amazonaws.com is using the discovered Python interpreter at /usr/bin/python, but futureTASK [web-ng : install java] 
*********************************************************************************************************************************
changed: [ec2-3-108-64-248.ap-south-1.compute.amazonaws.com]TASK [web-ng : show java version] ****************************************************************************************************************************
changed: [ec2-3-108-64-248.ap-south-1.compute.amazonaws.com]TASK [web-ng : configure webpage] ****************************************************************************************************************************
changed: [ec2-3-108-64-248.ap-south-1.compute.amazonaws.com]TASK [web-ng : start nginx service] **************************************************************************************************************************
changed: [ec2-3-108-64-248.ap-south-1.compute.amazonaws.com]RUNNING HANDLER [web-ng : restart server] ********************************************************************************************************************
changed: [ec2-3-108-64-248.ap-south-1.compute.amazonaws.com]PLAY RECAP ***************************************************************************************************************************************************
ec2-3-108-64-248.ap-south-1.compute.amazonaws.com : ok=8    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0ec2-65-0-71-50.ap-south-1.compute.amazonaws.com : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0[ec2-user@ip-172-31-15-156 ansible]$
```
#### Validation:
- Hit url http://ec2-3-108-64-248.ap-south-1.compute.amazonaws.com/index.html
- SSH to ec2-3-108-64-248.ap-south-1.compute.amazonaws.com and Execute the following command
```
[ec2-user@ip-172-31-8-70 ~]$ nginx -version
nginx version: nginx/1.20.2
[ec2-user@ip-172-31-8-70 ~]$ java -version
openjdk version "1.8.0_312"
OpenJDK Runtime Environment (build 1.8.0_312-b07)
OpenJDK 64-Bit Server VM (build 25.312-b07, mixed mode)
```
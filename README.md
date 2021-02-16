this repository is forked from jdelvecchio/ansible-role-zabbix-webscenario

the purpose of this repository is...

 - Follow new version of Zabbix
 - Change the interface to the same as community.zabbix https://github.com/ansible-collections/community.zabbix
 - Improve to be idempotent, flexible and easy to use


# ansible-role-zabbix-webscenario

Ansible role which creates or deletes (NO UPDATE) zabbix web scenarios
For each entry in the 'z_websites' variable with state: present, this role will :

* Add a web scenario
* Add a basic step
* Add a trigger associated to that web scenario

For each entry in the 'z_websites' variable with state: absent, this role will :
* Delete the web scenario associated to its name
* Delete the trigger associated to it

For any update of an existing web scenario, it must first be deleted (manually or by changing its state to absent, executing the playbook, then changing it back to 'present')

### Zabbix versions
This role was tested and validated with :
* Zabbix 5.0

### Variables

Here is the list of all variables and their default values :
```yaml

# Authentification to API
server_url: http://localhost:8080'                      #zabbix URL
login_user: Admin                                       #zabbix user to authenticate with API, must have access to the host
login_password: zabbix                                  #zabbix user password

# Common variables to all webscenarios
interval: 30                                            #default interval of execution of the web scenarios
regries: 5                                              #default maximum number of retries
agent: Zabbix                                           #useragent used by zabbix

# Definition of each website to monitor
step:
    ## Creates a webscenario with two steps
    ## The associated trigger will have two tags, display and any_other_tag.
  - name: 'Our Company Website'                         #name of the web scenario
    url: 'mycompany.com'                                #url of the web scenario (DO NOT PUT http(s)://)
    method: 'GET'                                       #http method used, GET or HEAD, HEAD by default
    status_codes: '200,301,302'                         #expected http code returned, 200 by default
    required: 'Welcome to mycompany website'            #required string on the page, method MUST be GET for this to work
    auth:                                               #for websites that require basic HTTP authentication
      http_user: 'jim'                                  
      http_password: 'jimspassword'
    trigger_tags:                                       #tags to add with the associated trigger
      display: 'yes'
      any_other_tag: 'any_value'

    ## Creates a webscenario with one step, checking http://anotherwebsite.com/ using HEAD HTTP method, expects 200 as return code
  - name: 'Another website'
    url: 'anotherwebsite.com'

    ## Other possible variables to put in z_websites
    ssl_verify_peer: 'no'                               #for https websites, disables the checking of the matching between the cn and the domain name of the website (default yes)
    ssl_verify_host: 'no'                               #for https websites, disables the checking of the root certificate against trusted store (default yes)

```

### Usage

Clone the repo.
```bash
$ git clone https://github.com/miettal/ansible-role-zabbix-webscenario
```
Then set vars in your playbook file.

Example playbook file :

```yaml
---
- hosts: 127.0.0.1
  gather_facts: no
  become: no
  roles:
    - role: ansible-role-zabbix-webscenario
      server_url : http://127.0.0.1:8080
      login_user: Admin
      login_password: zabbix

      name: testwebscenario
      host: testhost
      interval: 30
      regries: 5
      agent: Zabbix

      steps:
        - name: 'Our Company Website'
          url: 'example.com'
          state: present
          https: 'yes'
          method: 'GET'
          status_codes: '200,301,302'
          required: 'Welcome to mycompany website'
          trigger_tags:
            display: 'yes'
            any_other_tag: 'any_value'
```

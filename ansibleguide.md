---

Title: "ANSIBLE GUIDE"

---

---

Author: "Faiez Ben Habel"

---

  ![Ansible](./Ansible.png)

- ansible is a CMS (configuration management system).with ansible you will work with one central computer which called ansible control node where
 you will install ansible software and python and for the managed computer you just need an ssh connection and python because module
 generate python script to manage hosts.


- requirements for control node:
	- ssh key need to be populated to all managed host
	- create a user for ansible in control node/managed hosts
	- make the user a sudeors in order to do its job a tip to do this just scp the sudoers file to all your managed hosts in the /etc/sudoers.d directory
	- recomended to set up fixed ip address or set up dns for your managed hosts .you can 	use /etc/hosts file to do this job for you .
	- configure ssh to be authenticated with ssh key public key instead of password


- ansible installation:

	- you can install it from the repositories or from pip in python.
   python method:
	- yum instal python3 python3-pip
	- alternatives --set python /usr/bin/python3 (alias for python3 to python)
	- pip3 install ansible --user
	
- preparing managed nodes:

      - ansible needs a dedicated non-root user account with sudo privileges that can ssh into the managed hosts without entering password

- command to set up ansible user :

    - useradd ansible
    - echo "password"|passwd --stdin ansible
    - echo "ansible ALL=(ALL) NOPASSWD:ALL">/etc/sudoers.d/ansible

- next in control node do :
	- ssh-keygen
	- ssh-copy-id host1.example.com
	- install python3 software using yum
	   - yum install python3
       - alternatives --set python /usr/bin/python3

- verify ansible installation:
	- use the command:  ansible --version


## Set up static Inventory:

- in order to manage your hosts you need as we said earlier a dns resolution for your hosts or you can use /etc/hosts for small env

- ansible get the list of machines that will be managed from the inventory file. Invetory file can point to single vm or list of machines
      or even vm in the cloud.

- if you are working with individual server you can use static invetory file containing the ip addr or names of servers

- if you want to manage servers in the cloud enviroment  its recomended to use dynamic inventory which is a script that going to
     discover the servers that are living in the cloud and that makes it easy to discover new servers that are being added to the cloud env.

- in inventory you can define groups that regroup your managed hosts such webserver or database servers or by region in the cloud
- in inventory you can use variable (not recomended)

==> its better to use the inventory file instead of /etc/hosts or name resolution because you can regroup your servers and you can use the
    feature of dynamic inventory.

- naming resolution could be an option to type names of your servers instead of ip .

###  Managing static inventory:
- a static inventory is a list of host names and ip addresses  that can be managed by ansible
- hosts can be grouped in inventory to make it easy to address multiple hosts at once
- a host can be member of multiple groups
- nested groups are also available
- it is common to work with project based inventory files
- variables can be set from the inventory file but this is deprecated practice
- ranges can be used:
	-server[1:20] matches server1 up to server20
	-192.168.[4:5].[0:255] matches to full class C subnets
      - inventory location:
	-/etc/ansible/hosts is the default inventory
	- alternative escalation solution
        - become_user: the target user used for privilege escalation
    ==>those parameters are the same parameters in the ansible.cfg file but if they are specified in the play they take precedence
     invetory location can be specified through the ansible.cfg configuration file.
	- you can use the -i inventoryfile option to specify the location of the inventory file to use.
	- it is common practice to put the inventory file in the current project directory.
  - Static inventory file example:
```
 	  [webservers] ==>group of webservers
		web1.example.com
		web2.example.com
		[fileservers] ==> group of fileservers
		file1.example.com
		file2.example.com
		[servers:children]=>Nested groups servers:children define a new nested group named "servers"
		webservers
		fileservers
```
- HOST GROUP USAGE:
    - functional host grouping: database,web etc..
	- regional host groups:europe,africa,west-us..
	- staging host groups:test,development,production

- to list hosts in the inventory in with ansible command use:
     - ansible -i inventoryfile all --list-hosts
     - ansible -i inventoryfile ungrouped --list-hosts: list ungrouped hosts in the inventory file.
     - ansible-inventory --list -y: list inventory hosts in yaml format (-y option ,default json format)

###  Understanding  dynamic inventory:
  static inventory is good but not suitable for big enviroment.so the solution is to use dynamic inventory.

- Many community-provided dynamic inventory scripts are available.

- these scripts are referred to in the same way as static inventory but must have the execute permission set.

- alternatively it's relatively easy to write your own dynamic Invetory.

###  Understanding ansible configuration file:

- ansible.cfg example:
```
        [defaults]
        inventory = inventory ==> look for file named inventory in the current directory ;)
        remote_user = ansible ==> user to use in the remote hosts
        host_key_checking = false
        [privilege_escalation]
        become = true
        become_method = sudo ==> how to become  the other user
        become_user = root ==> the user to become in the remote hosts
        become_ask_pass = false ==> you dont need to prompt for  password to become root with sudo 
```
- Settings in ansible.cfg are organized in two sections
 - [defaults]: sets default Settings
 - [privilege_escalation]: specifies how ansible runs commands on managed hosts
 - other connection methods are available to manage windows for instance ,use ansible_connection: winrm and set ansible_port:5986 (not part of RHCE)

    - sudo is the default mechanism for privilege escalation
- set up passwordless configuration in sudoers file
- ansible has an implicit localhost entry to run ansible commands on the localhost
- when connecting to localhost the default become Settings are not used,but the account that ran the ansible command is used

    ==>ensure this account has been configured with the appropriate sudo privileges.

    ===>if the inventory specifed is a directory all inventory files in that directory are considered(static+ dynamic)

###  Managing ansible configuration files:
 - the ansible.cfg file ins considered in order of precedence:
     - /etc/ansible/ansible.cfg is the default
 
     - ~/.ansible.cfg if exists will overwrite the default
  
     - ./ansible.cfg is the configuration file in the current ansible project directory and it will always have precedence if it exists
    - alternatively if a variable ANSIBLE_CONFIG exists to refer to a specific config file this will always have precedence.
    - ansible --version to find which configuration file currently is used.
    
###  Using ansible Ad Hoc commands:

- in order to run ansible modules you can use ansible ad hoc commands.
- ad hoc are convinient for quick  tasks or setup like in command line
- for more complex tasks use ansible playbook written in yaml
- syntax of ad hoc command: ansible hosts -m modulneName [-a 'module arguments'] [-i inventory]
- example: ansible all -m user -a "name=lisa" : add user lisa to all managed hosts

###  Understanding ansible modules:

 - modules are key elements in working with ansible

 - you will always use modules to tell ansible what you want it to do in ad hoc commands as well as in playbook

 - use ansible-doc -l for a list of modules currently available

 - all modules work with arguments, ansible-doc will show which arguments are available and which are required.

 - ansible modules are idempotent: which means that when running them again they'll give the same result and if a task has already been configured the wont do it again

###  Using ansible doc to get module documentation:
- the ansible-doc command provides authoritative documentation about modules

- use ansible-doc -l for a list of all modules

- ansible-doc modulneName : to get documentation for a specific module

- you can find example with ansible-doc command

- modules are very actively developed by the community and the status field in the module documentation indicates the current status:

    - stableinterface: the module is stable and safe to use

    - preview: the module is in tech preview and its keywords my change

    - deprecated: the module should not be used anymore and will be removed in a future release.

    - removed: the module has been removed  and the documentation only still exists to help users migrating to its replacement

- the supported_by field in ansible-doc indicates who is responsible for supporting a module:

    - core: the module is supported by the core ansible team

    - curated: the primary support responsibility is with partners and companies in the community.Proposed changes are reviewed bu the core developers

    - community: support is completely within the community.

### introducing essential ansible modules:
- ping : verifies the ability to log in and python has been installed

    - ansible all -m ping

- service:  Controls services on remote hosts

    - ansible all -m service -a "name=httpd state=started" : check if the service httpd is running

- command: runs any command but not through a shell

    - ansible all -m command -a "/sbin/reboot -t now"

    - ansible host1.example.com -m command -a "/sbin/reboot"

- shell: runs arbitrary command through a shell

    - ansible all -m shell -a set (show information about connection to managed hosts about shell  variable enviroment)

    - -vvvv adds information about plug-ins,users used to run scripts and names of scripts that are executed

- use -C option to perform a dry run with ansible-playbook

- examples:
     - ansible-playbook --syntax-check vsftpd.yml

     - ansible-parameters -C vsftpd.yaml 
     
     - ansible all -m shell -a 'cat /etc/motd'

- raw : runs a command on a remote host without a need for python

- copy : copies a file from the local or remote machine to a location on the remote machine.

    - ansible all -m copy -a 'content="hello world " dest=/etc/motd'

==> content:When used instead of `src', sets the contents of a file directly to the specified value.

### Using yaml to write playbook:
- a playbook starts with ---

- then a  - name for the playbook

- then a  - tasks which is an object array that contains:

```
 - name:
      modulneName:
            arg1:
            arg2:
 - name:
      modulneName:
 
```
- example of a playbook :

```
---
- name : deploy vsftpd
  hosts: ansible2.example.com
  tasks:
    - name: install vsftpd
      yum:
        name: vsftpd
    - name: enable vsftpd
      service:
        name: vsftpd
        enabled: true
    - name: create readme file
      copy:
        content: "free downloads for everybody"
        dest: /var/ftp/pub/readme
        foce: false  #Influence whether the remote file must always be replaced.
        mode:0444
...
  
```

- use ansible-playbook vsftpd.yml to run the playbook

- seccessful run requires the inventory and become parameters to be set correctly and also requires access to an inventory 
file

- the output of the playbook of the ansible-playbook command will show what exactly has happened

- playbook are  idempotent which means it will give the same results if it runs multiple time.

- there is no easy way to undo changes made by a playbook

###  Verifying playbook syntax:

- ansible-playbook --syntax-check vsftpd.yaml

- use -v[vvv]:

    - -v will show task results

    - -vv will show task results and task configuration

    - -vvv also shows information about connection to managed hosts

    - -vvvv adds information about plug-ins,users used to run scripts and names of scripts that are executed
- use -C option to perform a dry run with ansible-playbook

- examples:

    - ansible-playbook --syntax-check vsftpd.yml
  
    - ansible-playbook -C vsftpd.yaml
  
    - ansible-playbook -vvvv vsftpd.yaml: so much logs will be shown (trace Debug)

- hint : in ~/.vimrc include the following setting:

  - autocmd Filetype yaml setlocal ai ts=2 sw=2 et

###  Writing multiple-play playbooks

- a play is a series of tasks that are executed against selected hosts from the inventory using specific credentials

- using multiple plays allows running tasks on different hosts using different credentials from the same playbook

- within a play definition escalation parameters can be defined:

    - remote_user: the name of the remote user

    - become: to enable or disable privilege escalation

    - become_method: to allow using an alternative escalation solution

    - become_user: the target user used for privilege escalation

==> those parameters are the same parameters in the ansible.cfg file but if they are specified in the play they take precedence

- example of multiple plays playbook :

```
---
- name: enable webserver
  hosts: ansible1.example.com
  tasks:
    - name: install httpd and firewalld
      yum:
        name:
            -httpd
            -firewalld
        state: latest
    - name: install welcome page
      copy:
        content: "hello world"
        dest: /var/www/html/index.html
    - name: start web services
      service:
        name: httpd
        enabled: true
        state: started
    - name: start firewalld services
      service:
        name: firewalld
        enabled: true
        state: started
    - name: open firewalld http port
      firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: yes
    - name: test web server
      hosts: localhost # we will test the webserver from localhost machine
      become: no
      tasks:
        - name: connect to the web server
          uri:
            url: http://ansible1.example.com
            return_content: true
            status_code: 200
...
```

- ansible-playbook --syntax-check myplay.yml

- ansible-playbook myplay.yml

==> dont be confused between multiple play and multiple tasks playbook!!!!

###  Understanding variables:
    
- in order to make your playbook modulare you need to use variables for example let's say you want to install httpd package
  in debian like system it is apache2 and in redhat system like is httpd so you can use variables in this case and assign value
  to it for each system

- you can set your variables definition  in the playbook file but it is a good practice because your playbook will be static use
  another file instead and set the variable in your playbook

- variables in the playbook are called with {{variableName}}

- a fact is a special type of variable that refers to a current state of an ansible managed system.
### Using variables:

 - variables can be set at different levels:

     - in a playbook

     - in inventory (deprecated)

     - in inclusion file

 - variables names have some requirements:

     - the name must start wih a letter

     - variables name can only contain letters,numbers and underscores

 - variables can be defined in a vars block in the beginning of a playbook:
```

     - hosts: all
       vars:
         web_package: httpd
```

 - alternatively variables can be defined in a variable file which will be included from the playbook:

```
     - hosts: all
       vars_files:
         - vars/users.yml
```

==> it is a convention to put the variable file in the vars directory (not mandatory)

- refer to a variable as {{ web_package }}

- if the variable is the first element using quotes is mandatory example:

```
---

- name: create a user using variable
  hosts: all
  vars:
    user: lisa
  tasks:
    - name : create a user {{ user }} #not first element so no quotes here
      user:
        name: "{{ user }}" # first element so quotes are mandatory
...
```
###  Understanding variables precedence:

- variables can be set with different types of scope:
    - GLobal scope: this is when a variable is set from inventory or the command line
    - play scope: this is applied when it is set from a play
    - Host scope : this is applied when set in inventory or using a host variables inclusion file
- when the same variable is set at a different levels the most specific level gets precedence
- when a variable is set from the command line it will overwrite anything else
    - ansible-playbook site.yml -e "web_package=apache"
- some variable are built in and cannot be used for anything else:
    - hostvars
    - inventory_hostname
    - inventory_hostname_short
    - groups
    - group_names
    - ansible_check_mode
    - ansible_play_batch
    - ansible_play_hosts
    - ansible_version

###  Managing Host variables:
- variable can be assigned to hosts and to groups of hosts

- a variable can be assigned directly to a host:
```   
     [servers]
     srv1.example.com web_package= httpd
```
- alternatively variables can be set for host groups
```   
    [servers:vars]
     web_package= httpd
```
- this is the inventory method which is not recomended

- create a folder named s or host_vars in the project directory (names are not random here predefined)

- example of s dir file named lamp  which refer to the group of host named lamp in the inventory :

```   
    web_package: httpd
    web_service: httpd
```
- you can include your group vars or host vars directory from another location but name of folder should be s and host_vars not random!

- host vars have the precedence of group vars test by me

- vars and vars_files has the precedence from host and group vars tested by me

###  using multi valued variables:
- arrays can be used as variables

- when doing so with_item can be used to refer to all values of the variable

- specific items in the array can be referred using dotted notation

- this is common in ansible fact as well

- example of a playbook that call an array

```
---
#beginning of ansible playbook 
vars_files:
    -vars/users
tasks:
    - name: print array values
      debug:
        msg: " User {{ users.linda.username }} has homedirectoy {{ users.linda.homedir }} and shell {{ users.linda.shell }}
...

    - example of the vars/users file:
        users:
            linda:
                username: linda
                homedir: /home/linda
                shell: /bin/bash
            lisa:
                username: lisa
                homedir: /home/lisa
                shell: /bin/bash
            anna:
                username: anna
                homedir: /home/anna
                shell: /bin/bash
```

###  Using ansible vault:

- some modules require sensitive data to be processed

- this may include webkeys, passwords and more

- to process sensitive data in a secure way ansible vault can be used

- ansible vault is used to encrypt and decrypt files

- to manage this process ansible-vault command is used

- to create an encrypted file use ansible-valut create playbook.yml

- this command will prompt for a new vault password and opens the file in vi for further editing

- as an alternative for entering passwords on the prompt a vault password file may be used but you'll have to make sure this is protected in another way

    - ansible-vault create --vault-password-file=vault-pass playbook.yml

- the view a vault encrypted file use:

    - ansible-vault view playbook.yml

- to edit the file use:

    - ansible-vault edit playbook.yml

- to encrypt an existing file use:

    - ansible-vault encrypt playbook.yml

- to decrypt an existing file use:

    - ansible-vault decrypt playbook.yml

- to change a password on an existing file use:

    - ansible-vault rekey playbook.yml

- to run a playbook that access vault encrypted files you need to use the --ask-vault-pass option to be prompted for password :

    - ansible-playbook firstplaybook.yml --ask-vault-pass

- alternatively you can store the password as a single-line string in a password file and access that using the --vault-password-file=file option:

    - ansible-playbook firstplaybook.yml --vault-password-file=passfile

- when setting up project with vault encrypted files it makes sense to use separate files to store encrypted and non-encrypted variables

- by convention the folder for the encryped variables is vault :
```
    #beginning of ansible playbook 
        vars_files:
            -vars/users
            -vault/secret
```

###  Working with facts

- ansible facts are variable that are automatically set and discovered by ansible on managed hosts

- facts contain information about hosts that can be used in conditionals

- for example before installing specific software you can check that a managed host runs a specific kernel version

- by default all playbook perform fact gathering before running the actual play

- you can run fact gathering in an ad hoc command using the setup module:

   - ansible -m setup all or ansible all -m setup

- to shwo facts, use the debug module to print the value of the ansible_facts variabl

```
        - name: debug fact
          debug:
            msg: "your system is : {{ ansible_facts.distribution }} and its hostname is {{ ansible_facts.hostname }} "
```

- in previous example we used dotted format to refer to a specific fact

- if you have a playbook that manages alot of hosts fact gathering may slow your playbook to disable it use:

    - use gather_facts: no in the playbook header to disable:

```
           - name: "install httpd and enable it"
             hosts: hosts
             gather_facts: no # you can use false too
             vars_files:
              - vault/secret
              - vars/var
```

- you can filter for specific information with the ad hoc command like :

   - ansible -m setup -a 'filter=ansible_default_ipv4' all

   - the dotted notation doesnt work well as the same way in conditionals in a playbook

###  Creating custom facts:

- custom facts allow administrator to dynamically generate variables which are stored as facts

- custom are stored in an ini file or json file in the /etc/ansible/facts.d directory on the managed host:

    - the name of these files must end in .fact

- to display local facts use :

    - ansible hostname -m setup -a "filter=ansible_local"

- example of a custom local file :
```
    [localfacts] ==> lable needed in an ini file
     package = vsftpd
     service = vsftpd
     state = started
```
- you can use a playbook to copy the fact file to managed host with copy module

- to get the value of the previous facts use:

    - ansible hostname -m setup -a "filter=ansible_local"

- you can refer to those fact in playbook like {{ansible_local.package}}

###  Understanding loops:

- the loop keyword allows you to iterate through a simple list of items

- example of using loop in playbook:

```
            - name: start some services
              service:
                name: "{{ item }}"
                state:started
              loop:
                - vsftpd
                - httpd
```
- the list used in loop can be defined by a variable:

```
        vars:
            my_services:
                - httpd
                - vsftpd
        tasks:
            - name: start some services
              service:
                name: "{{ item }}"
                state: started
              loop: "{{ my_services }}"
```

- you can use variable file instead

- example of loop to create users :

```
        tasks
         - name: create users
           user:
             name: "{{ item.name }}"
             state: present
             groups: "{{ item.groups }}"
          loop:
            - name: anna
              groups: wheel
            - name: linda
              groups: users
            - name: bob
              groups: users
```

###  Understanding register variable:

- a register is used to store the output of a command and address it as a variable it is similaire to command substitution in shell scrip

- you can use the result of the command in a condition or in a loo

- example of playbook :

```
        name: demo register
        hosts: all
        tasks:
            name: show loop register
            shell: " echo {{ item }} "
            loop:
                - "one"
                - "two"
            register: echo
            name: show register result
            debug:
                var: echo # debug var means  A variable name to debug and has an implicit `{{ }}' wrapping, similaire to msg
```
- more useful example:

```
        name: test register
        hosts: all
        tasks:
            - name: find user in passwd
              shell: cat /etc/passwd
              register: passwd_content # can be any name result if you want
            - name: debug the output
              debug:
                msg: echo "passwd contains user lisa" # you get the name of the user from a variable
              when: passwd_content.stdout.find('lisa') != -1 # condition , show message only when lisa user is found
```
###  Using when to run tasks conditionaly:

- when statments are used to run a task conditional

- a condition can be used to run a task only if a specific condition are tru

- playbook variables, registered variables and facts can be used in condition and make sure that tasks only run if specific condition are tru

- for example you can check if a task run successfully , a certain amount of memory is available , a file exists etc..

- examples of conditions

    - ansible_machine == "x86_64"
    
    - ansible_distribution_major_version == "8"
    
    - ansible_memfree_mb == 1024
    
    - ansible_memfree_mb < 256
    
    - ansible_memfree_mb > 256
    
    - ansible_memfree_mb <= 256
    
    - ansible_memfree_mb >= 256
    
    - ansible_memfree_mb != 256
    
    - my_variable is defined # boolean check if my_variable is defined and exist
    
    - my_variable is not defined
    
    - my_variable # boolean check if my_variable is defined and exist
    
    - ansible_distribution in supported_distros # supported distro a variable we set it

- example of playbook:

```
        name: when condition demo
        hosts: all
        vars:
            supported_distros:
                - RedHat
                - CentOs
                - Fedora
        tasks:
          - name: install RH family specific package
            yum:
              name: httpd
              state: present
            when: ansible_distribution in supported_distros
```
- when can be used to test multiple conditions as wel

- use "and" or "or" and group the conditions together:

    - when: ansible_distribution == "CentOs" or ansible_distribution == "RedHat"

    - when: ansible_machine == "x86_64" and ansible_distribution == "CentOs"

- the when keyword also supports a list and when using a list all of the condition must be true this is the same of using "and" statmen

- complex conditional statments can group conditions using parenthese

- example of multiple condition playbook:

```
        name: using multiple conditions
        hosts: all
        tasks
         - name: install package
           yum:
            name: httpd
            state: present
           when:
             - ansible_distribution == "RedHat"
             - ansible_memfree_mb > 512 #type int no need for quotes otherwise an error will be shown .
```
- example of multiple complex conditions playbook:
```
        name: using multiple conditions
        hosts: all
        tasks:
          - name: install package
            yum:
             name: httpd
             state: present
            when: > ( ansible_distribution == " RedHat" and
             ansible_memfree_mb > 512 )
             or
             (ansible_distribution == "CentOs" and ansible_memfree_mb > 1024 )
             # the > sign in yaml means it is a single line string so all those lines will be processed as a single line.
```
- loops and conditions can be combine

- for example you can iterate through a list of dictionaries and apply the conditional statment only if a dictionary is found that matches the conditio

- example of a playbook:

```
        name: restart sshd only crond  is running
        hosts: all
        tasks:
            - name: get te crond server status
              command: /usr/bin/systemctl is-active crond
              ignore_errors: yes # if the command fails playbook will continue
              register: result
            - name: restart sshd based on crond status
              service:
                name: sshd
                state: restartd
              when: result.rc == 0
```        

###  Using Handlers

- handlers allow you to configure playbook in a way that one task will only run if another task has been running successfully

- in oreder to run the handler a "notify" statment is used from the main task to trigger the handler

- handlers typically are used to restart services or reboot hosts

- handlers are executed after running all tasks in a play

- handlers will only run if a task has changed something, so if an ok result instead of changed result is reported te handler will not run

- if one of the tasks fails the handler will not run but this may overwritten using "force_handlers: true"

- one task may trigger more than one handler

- handler example in a playbook:

```
        name: handler demo
        hosts: all
        tasks:
            - name: install httpd
              yum:
              name: httpd
              state: latest
            - name: copy index.html
              copy:
                src: /tmp/index.html # copy the file from the control node !!
                dest: /var/www/html/index.html
              notify:
                - restart_web
            - name: copy nothing - we want it to fail
              copy:
                src: /tmp/nothing
                dest: /var/www/html/nothing.html
        handlers:
            - name: restart_web  # same as in the notify
              service:
                name: httpd
                state: restarted
```

###  Using blocks:

- a block is logical group of tasks

- it can be used to control how tasks are executed

- one block can  be enabled using a single "when"

- blocks can also be used in error condition handling
    - use block to define the main tasks to run
    
    - use rescue to define tasks that run if tasks defined in the block fail
    
    - use always to define tasks that will run regardless the success or the failure of the block and rescue tasks

- example of a playbook

```
        name: blocks demo
        hosts: all
        tasks:
            - name: setting up  httpd
              block:
               - name: install httpd
                 yum:
                    name: httpd
                    state: latest
              - name: restart httpd
                service:
                  name: httpd
                  state: started
              when: ansible_distribution == " CentOs"
```       
- another example of block:

```
        name: blocks demo2
        hosts: all
        tasks:
            - name: intended to fail
              block:
               - name: remove a file
                 shell:
                    cmd: /usr/bin/rm /proc/meminfo
              rescue:
                - name: create a file
                    shell:
                      cmd: /usr/bin/touch /tmp/rescuefile
              always:
                - name: always write a message to logs
                  shell:
                   cmd: /usr/bin/logger hello
``` 

### Dealing with failures:

- ansible looks at the exit status of a task to determine wether it has failed

- when any task fails ansible aborts the rest of the play on that host and continues with the next host

- different solutions can be used to change that behavior

- use ignore_errors in a task to ignore failures

- use force_handlers to force a handler that has been triggered to run, even if (another) task fails

- force_handlers only work if one of the task with same notify fails

- if the precedence task fails in playbook ansible will skip the next tasks and prompt an error so use ignore_errors !!

- ignore_errors specifed in tasks or block level !!

- if last one fails handler will be triggered

- as ansible only looks at the exist status of a failed task it may think a task was successfully where is not the case

- to be more specific use failed_when to specify what to look for in command output to recognize a failure

- example of a playbook:

```
        name: failed_when demo
        hosts: all
        tasks:
            - name: run a script
              command: echo hello world
              register: command_result
              failed_when: "'world' in  command_result.stdout"
            - name: see if we get here
              debug:
                msg: hello
```
- Managing the changed status may be important as handlers trigger on the changed status

- the result of a command can be registered and the registered variable can be scanned for specific text to determine that a change has occured.

- example of a playbook:

```
         name: failed_when demo
        hosts: all
        tasks:
            - name: check local date
              command: date
              register: command_result
              changed_when: false #prevent ansible from changing the sttus to changed as this command will always be runned without errors
            - name: see if we get here
              debug:
                var: command_result.stdout
```
###  Using modules to manipulate files
- different modules are available for Managing files:
    - lineinfile: ensure that a line is in a file, useful for changing a single line in a file
    - blockinfile: manipulates multi-line blocks of txt in files
    - copy: copies a file from a local or remote machine to a location on a managed host
    - fetch: used to fech a file from a remote machine and store it on the management node
    - file: sets attributes to files and can also create and remove files,symbolic links and more
- example of playbook:

```
                    name: create a file
                    hosts: all
                    tasks:
                        - name: create a file
                          file:
                            path: /tmp/removeme
                            owner: ansible
                            mode: 0640
                            state: touch
                            setype: public_content_rw_t
```
- another playbook example:

```
                 name: file copy module
                    hosts: all
                    tasks:
                        - name: copy file
                          copy:
                            src: /etc/hosts
                            dest: /tmp/
                        - name: add some line to /tmp/hosts
                          blockinfile:
                           path: /tmp/hosts
                           block: |
                            192.168.4.110 host1.example.com
                            192.168.4.120 host2.example.com
                           state: present
                            state: touch
                        - name: verify file checksum 
                          stat:
                            path: /tmp/hosts
                            checksum_alogrithm: md5
                          register: result
                        - debug:
                            msg: " the content of /tmp/hosts is {{ result.stat.checksum }} "
                        - name: fetch a file
                          fetch:
                            src: /tmp/hosts
                            dest: "my.hosts"
```

## Managing Selinux file context:
- to manage Selinux context you can use modules :
    - file : set attributes to files, including Selinux context and can also create and remove files , symbolic link and more 
    - sefcontext: manages selinux file context in the selinux policy (but not on files)

- Notice that file sets selinux context directly on the file like the chcon command and not in the policy

- example of playbook:

```
     - name: show selinux
       hosts: all 
       tasks:
         - name: install required packages
           yum:
             name: policycoreutils-python-utils
             state: present
         - name: set selinux context
           sefcontext:
             target: /tmp/removeme
             setype: tmp_t
             state: present
           notify:
             - run restorecon
       handlers:
         - name: run restorecon
           command: restorecon -v /tmp/removeme
```
###  Using Jinja2 templates: 

- lineinfile and blockinfile can be used to apply modification to files

- for more advanced modification use Jinja2  templates

- while using templates the target files are automatically customized using variable and facts

- in a Jinja2 templates you will find multiple elements:
    - data
    
    - variable
    
    - expressions
    
    - control structures

- the variables in the templates are replaced with their values when the Jinja2 template is rendered to the target file on the managed host

- if using variables they can be specified using the vars section of the playbook

- you can also use ansible facts as variables

- to prevent administrators from overwriting files that are managed by ansible set the ansible_managed string 
    - first in ansible.cfg set ansible_managed= ansible managed
    
    - on top of the Jinja2 template set the {{ ansible managed }} variable

- example of a playbook and template:

```
             ---
        - name : deploy vsftpd
          hosts: ansible2.example.com
          vars:
            anonymous_enable: yes
            local_enable: yes
            write_enable: yes
            anon_upload_enable:yes
          tasks:
            - name: install vsftpd
              yum:
                name: vsftpd
            - name: enable vsftpd
              service:
                name: vsftpd
                enabled: true
            - name: use template to copy FTP config
              template:
                src: vsftpd.j2 # ansible will look for this file in the current directory
                dest: /etc/vsftpd/vsftpd.conf
        ...
```

 #### ./templates/vsftpd.j2 file: 

```
 #{{ ansible_managed }}
#DO NOT MAKE LOCAL MODIFICATIONS TO THIS FILE AS THEY WILL BE LOST

anonymous_enable: {{ anonymous_enable }}

local_enable: {{ local_enable }}

write_enable: {{ write_enable }}

anon_upload_enable: {{ anon_upload_enable }}

dirmessage_enable= yes

connect_from_port_20= yes

#my ip address = {{ ansible_facts['default_ipv4]['address'] }} this is a fact!!
```

###  using control structures in Jinja2 templates:
- in Jinja2 templates control structures can be used to organize the template in an optimal way 

- the for statment can be used to iterate through a variable and use all values in the variable

- the if statment can be used to have template work with a variable if another variable is defined

- example:

```
    {% for user in users %}
       {{ user }}
    {% endfor %}
```    
- example 2:

```
   {% for myuser in users if not myuser == "root" %}
    User number {{ loop.index }} - {{ myuser }}
      {% endfor %}
```

- example 3: With the following for statement, all hosts in the myhosts group from the inventory would be listed in the file.

```
    {% for myhost in groups['myhosts'] %}
    {{ myhost }}
    {% endfor %}
```
- In the following example, the value of the result variable is placed in the deployed file only if the value of the finished variable is True

```
  {% if finished %}
  {{ result }}
  {% endif %}
 ``` 
- construct /etc/hosts file from a template:
 
 ``` 
   {% for host in groups['all'] %}
  {{ hostvars['host']['ansible_facts']['default_ipv4']['address'] }} # get ip address from fact for each hosts
   {{ hostvars['host']['ansible_facts']['fqdn'] }} # get host fqdn
    {{ hostvars['host'] ['ansible_facts']['hostname'] }} # get hostname
  {% endfor %}
```
- Jinja2 provides filters which change the output format for template expressions (for example, to JSON).  

    -  There are filters available for languages such as YAML and JSON.

    - The to_json filter formats the expression output using JSON 

    - the to_yaml filter formats the expression output using YAML

    - example:
```
        {{ output | to_json }}
        {{ output | to_yaml }}
```

- Additional filters are available, such as :

- the to_nice_json and to_nice_yaml filters, which format the expression output in either JSON or YAML human readable format

- example:

```
      {{ output | to_nice_json }}
      {{ output | to_nice_yaml }}
```

- Both the from_json and from_yaml filters expect strings in either JSON or YAML format, respectively, to parse them.

- example:

```  
    {{ output | from_json }}
    {{ output | from_yaml }}
```
###  Understanding directory structures best practice:

- when working with ansible it's recomended to use project directories so that contents can be organized in a consisten way 

- each project directory may have its own ansible.cfg, inventory as wel as playbook

- if the directory grows bigger variable files and other include files may be used 

- Roles can be use to standardize and easily re_use specific parts of ansible

- ansible documentation describes best practices some highlights:

    - on top in the directory use site.yml as the master playbook

    - from site.yml call specific playbooks for specific types of host (webservers.yml,dbservers.yml etc):

```
    - name: Play 1
      hosts: localhost
      tasks:
      - debug:
      msg: Play 1
    - name: Import Playbook
      import_playbook: play2.yml # import another playbook in our main playbook
```
- consider using different inventory files to differentiate between production and staging phases

- use groups_vars/ and host_vars/ to se host related variables

- use Roles to standardize common tasks

===> in ansible you can even include a task file like this one: 

```
            - name: Install web server
              hosts: webservers
              tasks:
              - include_tasks: webserver_tasks.yml # you can use also import_tasks directive
              ```
       ## example of task file:
       ```
           - name: Installs the httpd package
               yum:
                 name: httpd
                 state: latest
           - name: Starts the httpd service
               service:
                name: httpd
                state: started
```
###  Understanding ansible Roles:
- ansible playbooks can be very similar : code used in one playbook can be useful in other playbooks also

- to make it easy to re-use code roles can be used.

- A Role is a collection of tasks variables files templates and other resources in a fixed directory structure that can easily be included from a playbook

- roles should be written in a generic wat such that play specifics can be defined as variables in the play and overwrite the defualt variables in the play and overwrite the default variables that should be set in the role

- using roles makes working with large projects more manageable

- role folder structure: 

```
            [user@host roles]$ tree user.example
              user.example/
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
```
- The default roles_path on Red Hat Enterprise Linux includes /usr/share/ansible/roles inthe path, so Ansible should automatically find those roles when referenced by a playbook.

- Ansible might not find the system roles if roles_path has been overridden in the current Ansible configuration file, if  the environment variable  ANSIBLE_ROLES_PATH is set, or if there is another role of the same name in a directory listed earlier in roles_path.

- Understanding the struct
    
    - defaults: The main.yml file in this directory contains the default values of role variables that can be overwritten when the role is used.
    
    - files: This directory contains static files that are referenced by role tasks.
    
    - handlers: The main.yml file in this directory contains the role's handler definitions.
    
    - meta: The main.yml file in this directory contains information about the role, including author, license, platforms,
      and optional role dependencies.
   
    - tasks: The main.yml file in this directory contains the role's task definitions.
   
    - templates:This directory contains Jinja2 templates that are referenced by role tasks.
   
    - tests: This directory can contain an inventory and test.yml playbook that can be used to test the role.
   
    - vars: The main.yml file in this directory defines the role's variable values. Often these variables are used for internal purposes within the role. These variables have high precedence, and are not intended to be changed when used in a playbook.
 
  ===>Not every role will have all of these director
 
- Roles should not have site-specific data in them. They definitely should not contain any secrets like passwords or private keys
- Roles can be obtained in many ways:
    - you can write your own roles
   
    - for RedHat enterprise linux the rhel-system-roles package is available
   
    - the community provides roles through the ansible galaxy for standard roles:
      
    - ansisble galaxy is a public website where community provided role are defined
   
    - website
  
- Roles can be stored at a default location and from there can easily be used from playbooks:
    - ./roles has highest precedence

    - ~/.ansible/roles is checked after that

    - /etc/ansible/roles is checked next

    - /usr/share/ansible/roles is checked last

- roles are referred to from playbooks

- when roles are used they will run before any task that is defined in the playbook because:

- When a role is added to a play, role tasks are added to the beginning of the tasks list

- example:

```
      ---
      - name: role demo
        hosts: remote.example.com
        roles:
         - role1
         - role2
      ...
      ---
      - name: role demo
        hosts: remote.example.com
        roles:
         - role1
         - role2
           var1: cow
           var2: goat
```
- to control the execution order you can use 
  
  - pre_tasks : Any task listed in this section executes before any roles are executed. If any of these tasks notify a handler,those handler tasks execute before the roles or normal tasks.
  
  - post_tasks: Plays also support a post_tasks keyword. These tasks execute after the play's normal tasks, and any handlers 
     they notify, are run.
  
  - The following play shows an example with pre_tasks, roles, tasks, post_tasks and handlers:
  
```
    - name: Play to illustrate order of execution
      hosts: remote.example.com
      pre_tasks:
         - debug:
             msg: 'pre-task'
           notify: my handler
      roles:
       - role1
      tasks:
          - debug:
              msg: 'first task'
            notify: my handler
      post_tasks:
        - debug:
           msg: 'post-task'
          notify: my handler
      handlers:
       - name: my handler
         debug:
            msg: Running my handler
```
- In the above example, a debug task executes in each section to notify the my handler handler. 

- The my handler task is executed three times:
    
    - after all the pre_tasks tasks execute
    
    - after all role tasks and tasks from the tasks section execute
    
    - after all the post_tasks execute
- Roles can be added to a play using an ordinary task, not just by including them in the roles section of a play:
    
    -  Use the include_role module to dynamically include a role
    
    -  Use the  import_role module to statically import a role.

- example:

```
      - name: Execute a role as a task
        hosts: remote.example.com
        tasks:
          - name: A normal task
            debug:
               msg: 'first task'
          - name: A task to include role2 here
            include_role: role2
```
###  using ansible galaxy for standard roles:

- ansible galaxy is a public website where community provided role are defined
- to install a role use the ansible-galaxy install <role> command .

###  Using ansible galaxy command line tools:

- ansible-galaxy search will search for roles:
    
    - if and argument is provided ansible-galaxy will search for this arguments in the role description
    
    - use options --author , --platforms and --galaxy-tags to narraw down search results
    
    - ansible-galaxy search 'install mariadb' --platforms el : search for role to install mariadb in redhat platform
    
    - ansible-galaxy info: provides information about roles:
    
    - ansible-galaxy info f500.mariadb55
    
    - ansible-galaxy install: downloads a role and installs it in ~/.ansible/roles
    
    - after download these roles can be used in playbooks, like any other role 
    
    - ansible-galaxy list:shows installed roles
    
    - ansible-galaxy remove: can be used to clean up and remove roles
    
    - ansible-galaxy init: creates a directory structure that can be used to start developing your own role:
        
        - it interacts with the ansible galaxy website API
        
        - use --offline to work offline 
        
        - specify username and role as arguments
        
        - ansible-galaxy init --offline user.myrole
  
- requirements file :
    - You can also use ansible-galaxy to install a list of roles based on definitions in a text file
    - For example, if you have a playbook that needs to have specific roles installed, you can create a
      roles/requirements.yml file in the project directory that specifies which roles are needed.
    - For example, a simple requirements.yml to install geerlingguy.redis might read like this:
        - src: geerlingguy.redis  # The src attribute specifies the source of the role, in this case the geerlingguy.redis   role from Ansible Galaxy
```        
          version: "1.5.0"        #  he version attribute is optional, and specifies the version of the role to install, in this case 1.5.0
```   
  - You should specify the version of the role in your requirements.yml file,especially for playbooks in production.If you do not specify a version,you will get the latest version of the role.
  
  - To install the roles using a role file, use the -r REQUIREMENTS-FILE option:
  
      - ansible-galaxy install -r roles/requirements.yml -p roles
  
  - You can use ansible-galaxy to install roles that are not in Ansible Galaxy. You can host your own proprietary or internal roles in a private Git repository or on a web server.

  - example:

```  
            # from Ansible Galaxy, using the latest version
            - src: geerlingguy.redis
            # from Ansible Galaxy, overriding the name and using a specific version
            - src: geerlingguy.redis
             version: "1.5.0"
             name: redis_prod
            # from any Git-based repository, using HTTPS
            - src: https://gitlab.com/guardianproject-ops/ansible-nginx-acme.git
             scm: git
             version: 56e00a54
             name: nginx-acme
            # from any Git-based repository, using SSH
            - src: git@gitlab.com:guardianproject-ops/ansible-nginx-acme.git
             scm: git
             version: master
             name: nginx-acme-ssh
            # from a role tar ball, given a URL;
            # supports 'http', 'https', or 'file' protocols
            - src: file:///opt/local/roles/myrole.tar
              name: myrole
```            
- Note:
 If the role is hosted in a source control repository, the scm attribute is required. The ansiblegalaxy command is 
 capable of downloading and installing roles from either a Git-based or mercurial-based software repository.
 A Git-based repository requires an scm value of git, while a role hosted on a mercurial repository requires a value of hg.
 If the role is hosted on Ansible Galaxy or as a tar archive on a web server, the scm keyword is omitted. The name keyword 
 is used to override the local name of the role. The version keyword is used to specify a role's version. The version keyword
 can be any value that corresponds to a branch, tag, or commit hash from the role's software repository.

###  Creating custom roles:
      
- use ansible-galaxy init myrole to create the role directory structure

- each role should have its own version repository

- dont put sensitive information in the role

- dont forget to edit README.md and the meta/main.yml to contain documentation about your role 

- roles should be dedicated to one task/function , use multiple roles to manage multiple tasks/functions

- the meta/main.yml can be used to define role dependencies

- dependencies listed in that file will be installed automatically when this role is used

-  Using conditional roles:

    - conditional roles call a role dynamically using the include_role module

-  conditional roles can be combined with conditional statments:

    - this makes it so a role will be only run if conditional statement is true

    - use include_role in a task statement to do so 

    - example:

```
      ---
      - hosts: lamp
        tasks:
        - include_role:
            name: lamp
          when: "ansible_facts[os_family] == 'RedHat'"
```
###  Managing order of execution:

- Role tasks are always executed before playbook tasks

- we already discussed about this in previous example see above section just search pre_tasks keyword in this file 

###  Understanding rhel systems roles 

- rhel system roles are provided to configure standard rhel operations

- rhel system roles have been provided since rhel 7.4 and can be used to configure rhel 6.10 and later

- install the rhel-system-roles package to use them 

- rhel system roles are derived from the ansible linux system roles project which available through ansible galaxy

- currently the following rhel system roles are provided

    - rhel-system-roles.kdump: configures the kdump crash recovery service
    
    - rhel-system-roles.network: configures network interfaces
    
    - rhel-system-roles.selinux: manages all aspects of selinux
    
    - rhel-system-roles.tumesync: is used to set up network time protocol or precision time protocol
    
    - rhel-system-roles.postfix: is used to configure a host as a postfix mta
    
    - rhel-system-roles.firewall: configures a firewall
    
    - rhel-system-roles.tuned: configures the tuned service 

- Additional rhel system roles are likely to be introduced

### installing rhel system roles

- use yum install rhel-system-roles to install them 

- the roles are installed to the /usr/share/ansible/roles directory

- look for example yaml files in the role directories

###  Using rhel  selinux system roles:

- in some cases to apply selinux changes (such as switch between enabled and disabled mode ) a reboot is required

- the selinux role doesnt reboot the hosts 

- the role will set the selinux_reboot_required variable to true and fail if a reboot is required

- this is used in a block/ rescue structure whrer the play is failing if the variable is not set to true 

- it the variable is set to true the host is rebooted and the role is started again 

- you can see some example in documentation

- the rhel system role for selinux can do several things:

    - set enforcing or permissive mode

    - set selinux file context

    - run restorecon 

    - set booleans

    - set selinux user mappings

- to configure selinux from the role set at least the following variables:

    - selinux_state ; for managing state permissive or enforcing

    - selinux_booleans: to manage boolean

    - selinux_fcontexts: to manage file context

    - selinux_restore_dirs: restor defualt contxt to files

    - selinux_ports: to manage ports  

===> see the example in the documentation

###  Using rhel time sync system role:

- rhel-system-roles.tumesync can be used to manage time synchronization

- the role itself is configured to work with different variables of which timesync_ntp_servers is the most important one

- items in this variable are made up of different attributes of which two are common:
    - hostname: show the hostname of the time server
    
    - iburst: specifies that fast iburst synchronization should be used

- time timezone variable is also important and sets the current timezone to be used

- a defualt playbook is available in /usr/share/doc/rhel-system-roles/timesync

### addressing hosts pattern:

- by default hosts are addressed with their host name as specified in the inventory file 

- ip address can also be used

- host groups are common and are defined in inventory 

  - group all is implicit and doesnt have to defined

  - group ungrouped is also implicit and addresses all hosts that are not members of group 

- wildcards can be used : hosts: '*' is equivalent to hosts:all

- example of some wildcards:

```
  - hosts: '*.example.com 
  - hosts: '192.168.* 
  - hosts: 'web*  
  - hosts: web1,db1,192.168.4.  
  - hosts: web,&eastcoast # hosts that are memebers of group web and eastcoast  
  - hosts: web,!web1 #hosts memebers of group web and not memeber of web  
  - hosts: all,!web # all hosts except group web 
```

### configuring parallelism 

- Understanding processing order:
    
    - plays are executed in order on all hosts referred to and normally ansible will start the next task
      if this successfully completed on all managed hosts
    
    - ansible can run on multiple managed hosts simultaneously but by default the maximum number of simultaneous hosts is 
      limited to 5.
    
    - set forks = n in ansible.cfg to change the maximum number of simultaneous hosts 
    
    - alternatively use -f nn to specify the max number of forks as argument to the ansible[-playbook] command
    
    - the defualt of 5 is very limited so you can set this parameter much higher in particular if most of the work is 
      done on the managed host and not on the control node (be careful when you are running playbook for some remote hosts thatdoesnt have python (router e.g) the work will be done in the control node)
- Managing Rolling Updates:
    - the default behavior of running one task on all hosts and next to proceed to next task means that in cluster enviroment
      you may have all hosts temporarily being unavailable , use serial keyword in the playbook to run hosts through the entire 
      play in batches.
    - example:
```
                ---
                - name: Rolling update
                hosts: webservers
                serial: 2
                tasks:
                - name: latest apache httpd package is installed
                yum:
                name: httpd
                state: latest
                notify: restart apache
                handlers:
                - name: restart apache
                service:
                name: httpd
                state: restarted
```

===>In the example below, Ansible executes the play on two managed hosts at a time, until all managed hosts have
 been updated. Ansible begins by executing the tasks in the play on the first two managed hosts. If either or both
 of those two hosts notified the handler, then Ansible runs the handler as needed for those two hosts. When the play
 execution is complete on these two managed hosts, Ansible repeats the process on the next two managed hosts. Ansible
 continues to run the play in this way until all managed hosts have been updated.

===> Suppose the webservers group in the previous example contains five web servers that reside behind a load balancer.
 With the serial parameter set to 2, the play will run up to two web servers at a time. Thus, a majority of the five web
 servers will always be available.In contrast, in the absence of the serial keyword, the play execution and resulting
 handler execution would occur across all five web servers at the same time. This would probably lead to a service outage 
 because web services on all the web servers would be restarted at the same time.

====>Important Note:

For certain purposes, each batch of hosts counts as if it were a full play running on a subset of hosts. This means 
that if an entire batch fails, the play fails, which causes the entire playbook run to fail.
In the previous scenario with serial: 2 set, if something is wrong and the play fails for the first two hosts processed,
then the playbook will abort and the remaining three hosts will not be run through the play. This is a useful feature
because only a subset of the servers would be unavailable, leaving the service degraded rather than down.

==> The serial keyword can also be specified as a percentage. This percentage is applied to the total number of hosts in the play to determine the rolling update batch size.

### Managing ansible logs:

- by default ansible is not configured to log its output anywhere

- set log_path in ansible.cfg to write logs to a specific destination
    
    - create this file in the project directory as /var/log is not writable by the ansible user and will only work when running the playbook with sudo 

- when using log file you should use log rotation 

###  Using Check mode:
- use ansible-playbook --check on a playbook to perform a check mode, this will show what would happen when running the playbook without actually changing anything
    
    - modules in the playbook must support check mode 
    
    - chck mode doesnt always work well in conditionals

- set check_mode:yes within a task to always run that specific task in check mode :
 
    - this is useful for checking individual tasks 
    
    - when this setting is set to no for a task this task will never run in check mode 

- you can use check mode on tempate , add --diff to ansible playbook run to see diffrences that ould be made 
   by template files on a managed hosts :

    - ansible-playbook --check --diff myplaybook.yml   

###  Understanding modules related to Software management:

- package: distribution agnostic module to manage packages

- win_package: to manage winodws package

- apt: for apt

- yum : for yum 

- yum_repository: manage yum repositories

- package_facts: return information about packages as fact

- rpm_key: add or remove gpg key from an rpm package database

- redhat_susbcription: uses to subscription_manager command to manage subscription

###  Understanding modules related to user management:

- user : manages user

- group: manage groups

- pamd: configure PAM

- authorized_key: copies ssh public keys from ansible control to the target user .ssh/authorized_key
    
    - will only copy the keys and does not generate them 

- if you want to generate ssh key for user use module user and option generate_ssh_key

###  Understanding modules for managing process and tasks:

- cron: uses cron to schedule a job 

- at: uses at to run a future job 

- service: manages services

- service_fact: uses information about services as facts

- systemd: maages systemd services

###  Understanding modules for managing storage:

- parted: run the parted utility

- lvg: create lvm volume group

- lvol: creates lvm logical volume

- filesystem: managed filesystems

- mount : manages mounts

- vdo: manage vdo storage

###  Understanding modules for network management:

- nmcli: used to manage many parameters for network devices

- hostname: can be used to set hostname in managed host

- firewalld: can be used to manage firewalld rules

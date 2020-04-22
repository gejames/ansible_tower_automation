# Automating Changes to Ansible Tower



A common path for many that are learning to automate with Ansible is to start small. Pick a short and mundane task and automate it. Do you need to change the admin passwords on all your Cisco switches at once? Let Ansible handle it. As your skill level grows, more complex tasks can be added. You will probably start using a version control system to manage your playbooks. Overtime, you may want to orchestrate the deployments of your playbooks with Ansible Tower.  This allows you set up job templates for HR to run their own playbooks without even knowing what a module is. You don't have to worry about the new junior network admin doing a debug on the wrong router.  Life is good.  

But a new problem arises.  As your use of Tower grows and becomes more complex, how are you keeping track of changes within Tower?  You may have set up a Job Template for testing a new playbook and then never deleted it.  Is an auditor going to understand why you used a non-standard credential?  What if the name of a playbook changes? Did you remember to update it in your Job Templates?  

The answer, of course, is to automate and document these changes with Ansible.  We can then use our existing version control process to track those changes.  Tower becomes another node to automate. Just like when we learned Ansible, we will start small.  We will create a playbook that configures a Job Template in Ansible Tower and then track our changes with git.



##
Start by installing ansible-tower-cli. This is an open source project that we can leverage to make changes for us in Tower.   You can install it on Tower at the shell or another system running Ansible, as long as it can reach Tower by ssh.

 ```
 $ pip install ansible-tower-cli
 ```

We also need a credentials file so tower-cli knows how to login to Tower.  Use your favorite text editor to create a file called tower_cli.cfg at the root of your project and add in your Tower username and password

```
$ vim tower_cli.cfg

username: admin
password: p4ssw0rd
```

Next, let’s create a simple playbook that will add a Job Template to Tower. Again, we are starting small. We’ll assume you have an existing playbook to change the username and password on a Cisco device and have the corresponding inventory, project, and credential assets already setup in Tower.  The actual playbook does not matter.  Make sure to check the file into your version control system!




```
---
- name: SETUP TOWER JOB TEMPLATE
  hosts: localhost
  gather_facts: false

  tasks:
    - name: CREATE CISCO USERNAME UPDATE JOB TEMPLATE
        tower_job_template:
        name: CHANGE CISCO USER PASSWD
        job_type: "run"
        inventory: "cisco devices"
        project: "Cisco Project"
        playbook: "change_cisco_user_passwd.yml"
        credential: "cisco_creds"
        state: present
        survey_enabled: no
        tower_config_file: ./tower_cli.cfg
```

Let git know to add the file to our repo.

```
$ git add job_template.yml
```
And commit our file.
```
$ git commit -m "added job_template.yml file"
```



##
Now, let’s say a few months later you decide to standardize all playbook names to start with the vendor name.

```"change_cisco_user_passwd.yml"```  becomes ```"cisco_user_passwd.yml"```

We update our job_template playbook with the new name.
```
...
 project: "Cisco Project"
 playbook: "cisco_user_passwd.yml"
 credential: "cisco_creds"
...
```
Commit your changes with a comment so we have a paper trail of why the change was made.
```
$ git add job_template.yml
$ git commit -m "updated playbook name to standard of vendor name first"
```










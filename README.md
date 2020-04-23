# Automating Changes to Ansible Tower

## Why should I automate changes to Ansible Tower? 

As the complexity of any system grows, so does the need to document any changes to that system.  If you have been using the Ansible Automation Platform for a while now, you may have built up a large collection of playbooks and are using Ansible Tower to centrally orchestrate their deployment.  You've probably configured a few workflows and built surveys to extend the reach of Tower beyond your IT organization. 

As your needs have grown, how are you keeping track of changes within Tower? Are you tired of writing the same survey everytime a new playbook requries it?  Is an auditor going to understand why you used a non-standard credential? Or perhaps you are a consultant that deploys Tower for many clients and needs a way to confugre Tower quickly. Wouldn't it be great if we could use our existing knowledge of Ansible to automate, and in turn doucment, those changes and re-use assets we already have?  

In this article, I will deomonstrate how we can leverage the open source tool tower-cli and write playbooks that will do just that.   

## Starting Small

A frequent path for many that are learning Ansible is to start small.  We shall do the same here.  We'll pick a mundane task and automate it. 

One of the more common assests in Tower is the job template.  This is what we create when we want to run a playbook. But, do you recall why you created them all?  Did you remember to update all the job templates when a playbook was renamed?  Lets use Ansible and git to create the job template and track when it is changed.

## Requirements

Start by installing ansible-tower-cli. This is the open source project that we can leverage to make changes for us in Tower. For simplicity, we'll install it on our Tower server at the cli.

 ```
 $ pip install ansible-tower-cli
 ```

We also need a credentials file so tower-cli knows how to login to Tower.  Use your favorite text editor to create a file called tower_cli.cfg at the root of your project and add in your Tower username and password

```
$ vim tower_cli.cfg

username: admin
password: p4ssw0rd
```

We could use tower-cli to create our job template from the command line, but we want to write a playbook to do that. Ansible provides many modules that can interfact with tower-cli.  For our purposes, we'll be using the tower_job_template module.  Again, we are starting small. We’ll assume you have an existing playbook to change the username and password on a Cisco device and have the corresponding inventory, project, and credential assets already setup in Tower.  The actual playbook does not matter.  Make sure to check the file into your version control system!


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


We now have a reusable playbook we can use to create job templates and we've documented why it was updated when our playbook name changed.















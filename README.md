# Automating Changes to Ansible Tower

## Why should I automate changes to Ansible Tower? 

As the complexity of any system grows, so does the need to document changes to that system. If you have been using the Ansible Automation Platform for a while now, you may have built up a large collection of playbooks and are using Ansible Tower to centrally orchestrate their deployment.  You have probably configured a few workflows and built surveys to extend the reach of Tower beyond your IT department. Most likely you have a robust DevOps process and are using a VCS,  such as git, to track changes to  your playbooks.  

For those new to the Ansiblle Automation Platform, Tower is the orchestration tool that provides a powerful web UI and API to control your Ansible playbooks.   Think of it as a central point to administer all the assests that are needed to run your Ansible jobs. Login credentials, inventories, and project repositories can all be configured and reused as needed.  Tower also provides scheduling and role based access control.   

As an organization's use of Tower grows, you can see how it will become increasingly important to track and document changes to those assests. Is an auditor going to understand why you used a non-standard credential for a job? Or perhaps you are a consultant that deploys Tower for many clients and need a way to confugre it quickly. Are you tired of writing the same survey everytime a new job template requries it?  

Wouldn't it be great if we could use our knowledge of Ansible and our existing DevOps process to automate, and in turn document, those changes?  If we can write a playbook that configures those job templates, credentials, or inventories for us then we have a self-docmenting system to configure Tower.   We can share our playbooks with collegues so they can quickly deploy the same jobs.  If deploying Tower to the cloud, we have a way to add all our assets automatically and seamlessly.   

In this article, I will demonstrate how we can leverage the open source tool tower-cli and write playbooks that will do just that.   

## Starting Small

A frequent path for many that are learning Ansible is to start small.  We shall do the same here.  We'll pick a mundane task and automate it. 

One of the more common assests in Tower is the job template.  This is what we create when we want to run a playbook. However, do you recall why you created them all?  Did you remember to update all the job templates when a playbook was renamed?  

Lets use Ansible and git to create the job template and track when it is changed.

## Requirements

Start by installing ansible-tower-cli. This is the open source project that we can leverage to make changes for us in Tower. For simplicity, we'll install it on our Tower server at the cli.  It is possible to run it from another system, but additional steps are required.  Refer to the Ansible documentation [here](https://docs.ansible.com/ansible-tower/3.5.3/html/towerapi/tower_cli.html) for a deeper dive into tower-cli.

Install tower-cli with the below command

 ```
 $ pip install ansible-tower-cli
 ```

We also need a credentials file so tower-cli knows how to login to Tower.  Use your favorite text editor to create a file called tower_cli.cfg at the root of your project and add in your Tower username and password.  You can also add this file to your .gitignore so it's not tracked.

```
$ vim tower_cli.cfg

username: admin
password: p4ssw0rd
```

We could use tower-cli to create our job template from the command line, but we want to write a playbook to do that. Ansible provides many modules that can interface with tower-cli.  For our purposes, we'll be using the tower_job_template module.  Again, we are starting small. We’ll assume you have an existing playbook to change the username and password on a Cisco device and have the corresponding inventory, project, and credential assets already setup in Tower.  The actual playbook does not matter.  Make sure to check the file into git!


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

At this point, the playbook will go through your normal process for deploying new playbooks.  All fully documented.  


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


If you would like to learn more about the complete list of moduesl we can use to configure Tower, you can find them [here] 












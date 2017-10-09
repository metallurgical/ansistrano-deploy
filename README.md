## Ansistrano-deploy
Deployment guidance using ansible + capistrano

## Requirement(Tested)
- Linux OS(Linux Mint) - Local
- Ubuntu 16 LTS - Linode Server
- Ansible
- Ansistrano v2.2.0
- Git
- SSH

## Introduction
Deploy scripting language has never been easy, but with the help of Ansistrano, we can deploy so much language(PHP, Ruby, Python, etc) into our staging/production server with just a few configuration that need to setup. Lucky to us, we can use both **ansistrano.deploy** and **ansistrano.rollback** for deployment and rollback respectively.

## Concept
Before proceed to step by step, its better if we know how the end result of deployment should end. After the deployment process's done, we should have a few folder exist/created in our **target(where to deploy to)** as below.

 - **current**(symlink to release folder)
   - The symlink folder that point out to latest release's timestamp that exist in *release* folder. So at this point, if we're using apache, we need to setup up *document root* point to *current* folder instead of *release*.
 - **release**
   - Having release(timestamp) folder that contain our application's code. This folder will have a few folders depend on how many release we want to keep before the clean up process occured. So this folder actually stored our application's code either **fetching** from git repository or **copying** from local development to staging/production server using **rsync** command which is depend on our ansible's configuration.
 - **shared**
   - This folder contains shareable file or variables that we can re-used between release. Maybe some logging file which is being shared by each release.
   
## Installation
Ansistrano is an Ansible role distributed globally using Ansible Galaxy. In order to deploy your apps with Ansistrano, you will need to install `ansible` using following command:

```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```

After installation of ansible, your machine should have `/etc/ansible` folder(this folder actually can exist in any place, even also can placed it inside our local development directory - per project). Then install ansistrano roles(this roles will available in installed machine):

```
$ ansible-galaxy install carlosbuenosvinos.ansistrano-deploy carlosbuenosvinos.ansistrano-rollback
```

## Deployment setup
This step only convered deployment with **git**, so ensure you have tested/working project inside respositories(github, gitlab, bicbucket, etc). Then, let replicate some of these by example. 

Assume you have local development reside in this `/opt/lampp/htdocs/<my-project>`, at some point we can create `ansible` directory in our local development project too along with other codes. And should become like this `/opt/lampp/htdocs/<my-project>/ansible`. Let assume we have tree structure like in our **local** machine like below:

```
-- /opt/lampp/htdocs/<my-project>
 |- ansible <-- contain our playbook for deployment
 |- index.php <-- contain php code
 |- composer.json <-- contain project dependencies(soon we'll create task for executing composer install after one of ansible life cycle's done)
```

Inside `ansible` directory, create `hosts` file if not exist, otherwise edit it. Host file is the file that contains our remote/production/staging's ip address or domain. Later we'll reference this file inside our `playbook`. Here is example of hosts file looks like :

```
[staging_server] <-- this is group name, we can put more than one server
172.x.x.x

[production_server] <-- this is group name, we can put more than one server
172.x.x.x
```

After that, create playbook file with the name `deploy.yml`, put following code:

```
---
- name: Deploy php application to staging server
  hosts: staging_server <-- name of the group inside hosts file or just put `all` if want to deploy all of the servers
  remote_user: <sudo user in target server> <-- this user should able to use git command(fetch code inside remote repositories)
  
  vars:
    ansistrano_deploy_to: "/var/www/<my-project>" <-- this is our root directory in target server
    ansistrano_version_dir: "release" <-- this folder's name will be created when deployment process is finish
    ansistrano_current_dir: "current" <-- this folder's name will be created when deployment process is finish
    ansistrano_current_via: "symlink" <-- current folder will symlink to release folder(latest release's timestamp)
    ansistrano_keep_release: 2 <-- how many times should we keep release versioning before ansible clean up older release
    ansistrano_deploy_via: "git" <-- method to deploy, we use git
    ansistrano_git_repo: git@gitlab.com:<username>/<my-project>.git <-- git endpoint(using ssh, not http based)
    ansistrano_git_branch: master <-- default branch to pull
    ansistrano_git_identity_key_remote_path: "/home/metallurgical/.ssh/id_rsa" <-- tell ansible to use this private key for fetching from repositories(this key must be add into our remote repositories)

  roles:
   - carlosbuenosvinos.ansistrano-deploy <-- ansible role
   ```
   
   If everything's done configured, run following command:
   
   ```
   $ ansible-playboook -i /opt/lampp/htdocs/<my-project>/ansible/hosts /opt/lampp/htdocs/<my-project>/ansible/deploy.yml
   ```
   
   If everything's good, the ansible will do it job like `creating current, release and shared folder`, `pulling data from git repositories`, `create symlink`, `clean up older release` or else... And inside our remote/staging/production/target server now should contains this tree structure:
   
   ```
   - /var/www/<my-project>
   |- current -> ./release/20171009102230Z <-- symlink to latest release
   |- release -> 
     |- 20171009102230Z <-- this is the latest release(contain our code)
      |- index.php
      |- composer.json
      |- ... any files
     |- 20171009092230Z <-- this is the older release(contain our code)
      |- index.php
      |- composer.json
      |- ... any files
    
   |- shared
   ```
   
   For automated process, this setup can be setup directly into target machine. To make it possbile, setup CI/CD(continoues integration & continoues delivery) inside gitlab, create a task and make gitlab task run/execute ansible command for every push from local into remote repositories or every merge from feature branch into master branch.
   
   This is just a basic configuration. For more details proceed to https://github.com/ansistrano/deploy and http://docs.ansible.com/ documentation. Done!


  


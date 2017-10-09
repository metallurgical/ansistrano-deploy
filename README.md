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
   - The symlink folder that point out to latest release's timestamp that exist in *release* folder. So at this point, if we're using apache, we need to setup up *document root* point to *current* folder instead of *current*.
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

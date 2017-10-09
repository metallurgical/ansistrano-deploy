## Ansistrano-deploy
Deployment guidance using ansible + capistrano

## Requirement(Tested)
- Linux OS(Linux Mint) - Local
- Ubuntu 16 LTS - Linode Server
- Ansistrano v2.2.0
- Git
- SSH

## Introduction
Deploy scripting language has never been easy, but with the help of Ansistrano, we can deploy so much language(PHP, Ruby, Python, etc) into our staging/production server with just a few configuration that need to setup. Lucky to us, we can use both **ansistrano.deploy** and **ansistrano.rollback** for deployment and rollback respectively.

## Concept
Before proceed to step by step, its better if we know how the end result of deployment should end. After the deployment process's done, we should have a few folder exist/created in our **target(where to deploy to)** as below.

 - current(symlink to release folder)
   - The symlink folder that point out to latest release's timestamp that exist in *release* folder. So at this point, if we're using apache, we need to setup up *document root* point to *current* folder instead of *current*.
 - release
   - Having release(timestamp) folder that contain our application's code. This folder will have a few folders depend on how many release we want to keep before the clean up process occured. So this folder actually stored our application's code either **fetching** from git repository or **copying** from local development to staging/production server using **rsync** command which is depend on our ansible's configuration.
 - shared
   - This folder contains shareable file or variables that we can re-used. Maybe some logging file which is being shared by each release.

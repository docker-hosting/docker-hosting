Docker Hosting
==============

Docker hosting is a simple tool to manage a private hosting environment. You can create, backup, restore and administrate a lot of websites with ease.

## Installation
### Requirements
You need a linux server with docker and python 3 installed. If you use [Fedora Server] from version 26 and above the installation command down below will setup the requirements automatically.

> TIP:  
> We recommend you to use [Fedora Server] in combination with [mamiu/dotfiles] for the best hosting server experience.  
> After Fedora Server installation you just need to call
>
>     curl -sL install.miu.io | bash
>
> on the server or on your local machine (you'll be able to setup a remote host). This setup script also asks you to configure your server as a *docker hosting* server.

### Installation
You just have to call the following command to install docker hosting on your server:

````bash
curl -sL "https://raw.githubusercontent.com/docker-hosting/docker-hosting/master/install/install.sh" | bash
````

## The concept
By default `/var/www` is the base path of *docker hosting*. Every directory (which contains a `docker-compose.yml`) within `/var/www` represents a website or web application you want to host.  
For example you have a wordpress blog [http://my-blog.com](#). Now you have to create a new directory in `/var/www`. The name doesn't matter, but `my-blog.com` would be meaningful. Since *docker hosting* is based on [docker] (especially [docker-compose]) you need a `docker-compose.yml` file inside `/var/www/my-blog.com/` that this directory will be recognised as a *docker hosting* website.

To simplify the creation of a new website [docker hosting] provides prebuild templates. All the template repositories are suffixed with `-template`. In the example above it would be [wordpress-template].  
This template could be installed with

````bash
git clone https://github.com/docker-hosting/wordpress-template /var/www/my-blog.com
````


## Docker hosting CLI
If you installed *docker hosting* as [described above](#installation) there's a `dh` command to administrate your websites.

### dh create
With `dh create` you can create a new website.
````bash
Usage:
    dh create <template_name> <website_name>

Arguments:
    <template_name>   One of the template repositories on the docker hosting
                      github account, but without the -template suffix.
    <website_name>    The name of the website.
````

The example above would be `dh create wordpress my-blog.com`.

### dh cd
With `dh cd` you can jump into a websites directory.
````bash
Usage:
    dh cd <website_name>

Arguments:
    <website_name>    The name of the website
````

### dh backup
With `dh backup` you can backup one ore more websites.
````bash
Usage:
    dh backup [options] <website_name>

Arguments:
    <website_name>    The name of the website

Options:
    -a, --all         Backup all websites. The <website_name> argument will be ignored.
    --website-only    Backup only the website(s) without the container data. All the
                      files inside the websites directory will be backed up but without
                      calling the backup script(s).
````

### dh restore
With `dh restore` you can restore one ore more websites.
````bash
Usage:
    dh restore [options] <website_name>

Arguments:
    <website_name>    The name of the website

Options:
    -a, --all         Restore all websites. The <website_name> argument will be ignored.
    --website-only    Restore only the website(s) without the container data. All the
                      files inside the websites directory will be restored but without
                      calling the restore script(s).
````


## Website configuration
*Websites* can be configured with a config file in the websites directory. The config file has to be called `docker-hosting.yml` (the `.yaml` extension works as well).

Here's an example of the default config file:
````yaml
version: '1'

backup:
    script: 'backup/backup.sh'
    archive: 'backup/backup.zip'
    keep_last: true
    cron_time: '0 1 * * *'

restore:
    script: 'restore/restore.sh'
    archive: 'restore/restore.zip'
````

### options
#### version
The version of the config file. Till now it's version **1**.
#### backup
In the backup section you can define parameters for the backup task.
#### backup/script
The path and name of the backup script relative to the websites directory. This script is called if `dh backup` command is executed or if the current time matches the `cron_time` option. If there's no `script` option under `backup` and no `backup/backup.sh` backup script, only the websites directory will be backed up but not the docker container content.  
Default is `backup/backup.sh`.
#### backup/archive
If the backup script was executed successfully *docker hosting* will backup this archive to the backup storage, configured in the `docker-hosting/docker-hosting.yml` config file.  
Default is `backup/backup.zip`.
#### backup/keep_last
If `keep_last` is set to `true` docker hosting will allways keep the last backup on the *docker hosting* server.  
Default is `true`.
#### backup/cron_time
With the `cron_time` option you can specify the frequency when the website will be backed up.  
Default is `0 1 * * *` (every day at 1am).  
Here is a good [cron generator](http://crontab-generator.org/).

#### restore
In the restore section you can define parameters for the restore task.
#### restore/script
The path and name of the restore script relative to the websites directory. This script is called if `dh restore` command is executed. If there's no `script` option under `restore` and no `restore/restore.sh` restore script, only the websites directory will be restored but not the docker container content.  
Default is `restore/restore.sh`.
#### restore/archive
Before the restore script is executed *docker hosting* will restore this archive from the backup storage, configured in the `docker-hosting/docker-hosting.yml` config file.  
Default is `restore/restore.zip`.


## Docker hosting configuration
*Docker hosting* can be configured with a config file in the `/var/www/docker-hosting` directory. The config file has to be called `docker-hosting.yml` (the `.yaml` extension works as well).

Here's an example of the default config file:
````yaml
version: '1'
backup-storage:
    google-cloud-storage:
        active: true
        bucket: 'my-bucket-name'
        target: '/target/location'
        credentials: 'keyfile.json'
    amazon-s3:
        # not yet available
    ftp:
        # not yet available

backup-defaults:
    script: 'backup/backup.sh'
    archive: 'backup/backup.zip'
    keep_last: true
    cron_time: '0 1 * * *'

restore-defaults:
    script: 'restore/restore.sh'
    archive: 'restore/restore.zip'
````

### options
#### version
The version of the config file. Till now it's version **1**.
#### backup-storage
In the `backup-storage` section you can specify one or more backup storages, where the backups will be stored. On a backup process *docker hosting* will backup the data to every storage with the flag `active: true`.
#### google-cloud-storage/bucket
The bucket where the backup data will be stored.
#### google-cloud-storage/target
The target location for the backups.
#### google-cloud-storage/credentials
Json key file of the google service account credentials. Have a look at: [Generating a service account credential][1]


## Technical background
All the magic with proxying the domains is done by [nginx-proxy]. The *docker hosting* CLI is written in python 3 and the rest are just simple conventions for the directory structure and config files.


[docker]: https://www.docker.com/
[docker-compose]: https://docs.docker.com/compose/
[docker hosting]: https://github.com/docker-hosting
[wordpress-template]: https://github.com/docker-hosting/wordpress-template
[nginx-proxy]: https://github.com/jwilder/nginx-proxy
[Fedora Server]: https://getfedora.org/server/
[mamiu/dotfiles]: https://github.com/mamiu/dotfiles
[1]: https://cloud.google.com/storage/docs/authentication#generating-a-private-key

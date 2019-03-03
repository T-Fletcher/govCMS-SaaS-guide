# govCMS SaaS project

**Current as of 21 February 2019.**

You're reading the doco! Good on you, keep going!

This project allows you to spin up a new govCMS Drupal 7 SaaS site locally using Docker, then import your site over it for local development. It assumes basic web developer knowledge and experience using a command line. 

**Before you can clone anything from projects.govcms.gov.au, you must have an account created by the govCMS team, with access to the appropriate projects.** Your project manager can request this by emailing support@govcms.gov.au. While anyone can create a new govCMS site, importing a govCMS client site requires appropriate access to the site project repository to have already been granted by the Department of Finance. 

**Running commands from this documentation**

When running commands listed in these instructions, anything wrapped in `${curly brackets like this}` represents a placeholder that you need to swap with your own information, so something like this:

    cd ${my-project-name}

would actually be run as:

    cd  mygreatproject

**Be sure to read the '[Known issues and workarounds](#known-issues)' section at the end of this document.**

## Contents

- [Official source material](#official-source-material)
- [Requirements](#requirements)
- [Development Rules](#development-rules)
- [Setting up a site](#setup)
- [Image updates](#image-updates)
- [Importing an existing site](#importing-an-existing-site)
- [Adding development-only modules](#adding-development-only-modules)
- [Shutting down your computer without losing the work inside your containers](#shutting-down-your-computer-without-losing-the-work-inside-your-containers)
- [Pushing commits](#pushing-commits)
- [Known issues and workarounds](#known-issues-and-workarounds)
- [Notes](#notes)

### <a name="official-source"></a>Official source material

You don't need to read it for this project, but sometimes it's nice to know the background :) 

1. The vanilla GovCMS (7.x-3.x) Distribution is available at [Github Source](https://github.com/govcms/govcms) and as [Public DockerHub images](https://hub.docker.com/r/govcms)
2. Those GovCMS images are then customised for Lagoon and GovCMS, and are available at [Github Source](https://github.com/govcms/govcmslagoon) and as [Public DockerHub images](https://hub.docker.com/r/govcmslagoon)
3. Those GovCMS lagoon images are then retrieved by the `govcms-site` scaffold repository.

Documentation on debugging Docker for Windows issues lives on the [Amazeeio GitHub profile](https://github.com/amazeeio/docs/blob/master/local_docker_development/troubleshooting.md).


## <a name="requirements"></a>Requirements

Version numbers are noted where applicable. 

**Everyone**

* [Docker Desktop](https://docs.docker.com/install/) v18.09 min - A container management system. You only need the Community Edition, but will need to create a Docker account and note your Docker ID prior to using it. 
* [Git](https://git-scm.com/downloads) v1.8 min - a code version control program. This can be installed either on its own (Mac/linux) or with [Git for Windows](https://gitforwindows.org/), or alternatively with a GUI such as [SourceTree](https://www.sourcetreeapp.com/). Mac users with Xcode or Xcode Command Line Tools installed may already have Git. Git also comes with [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

**Linux/Mac users**

* [Ruby](https://www.ruby-lang.org/en/) - A dynamic programming language, allows running of `pygmy`
* [pygmy](https://docs.amazee.io/local_docker_development/pygmy.html#installation) v0.9.10 min - A miniature local web server (You might need `sudo` for this depending on your ruby configuration)

**Windows users**

* Windows 10 Pro or Enterprise (Docker can't run on Windows Home edition as it doesn't come with Hyper-V)
* [Amazee Docker for Windows](https://github.com/amazeeio/amazeeio-docker-windows) - A set of additional Docker containers related to networking that run in Windows, in the absence of `pygmy` (it's just set of containers, not an Amazee-specific version of the program 'Docker for Windows'). Cloning instructions are in the '[Clone the project repo](#clone-it)' section.


**Optional**

* A package manager (this will vary depending on your OS). Linux ships with one out of the box. Some examples are:
    - Linux: `npm` (from NodeJS), `apt`, `dpkg` (default on Debian) or `yum`
    - Mac: `homebrew`, `nix`, `pkgsrc`
    - Windows: NodeJS provides `npm` on Windows
* [Ahoy](http://ahoy-cli.readthedocs.io/en/latest/#installation) - A shortcut manager for saving your precious fingers; all Docker shortcuts can be viewed in `.ahoy.yml`
* [NodeJS](https://nodejs.org/en/) - Allows the use of `npm` - Node Package Manager, to run developer tooling
* [Portainer](https://portainer.io/install.html) - A nice web UI for managing Docker containers



## <a name="dev-rules"></a>Development rules

* You should create your theme(s) in folders under `/themes`
* Tests specific to your site can be committed to the `/tests` folders
* The files folder is not (currently) committed to GitLab.
* Do not make changes to `docker-compose.yml`, `lagoon.yml`, `.gitlab-ci.yml`, `.ahoy.yml`, `README.md`, or the Dockerfiles under `/.docker` - Altering these files will result in the project being unable to deploy to GovCMS SaaS. These files are locked in GitLab, so attempting to push changes will fail anyway. 
* Some files can be extended/overridden by including additional files (see the '[Adding development modules](#dev-modules)' section)
* You should never need to change anything _inside_ a Docker container. Directly altering the contents of a container compromises it, meaning when a new instance of the project is spun up, those alterations won't be present.
* **Check for image updates every day!** To ensure you are running the latest environment, and therefore that your local work runs in a setup that accurately reflects the production environment, be sure to [pull down the latest images from the container registry](#image-updates) every day.   



## <a name="setup"></a>Setting up a site

### <a name="connect"></a>1. Connect to projects.govcms.gov.au

**Before you can clone anything from projects.govcms.gov.au, you must have an account created by the GovCMS team. Your project manager can request this by emailing support@govcms.gov.au**.

**Note**: You may experience connectivity issues depending on your internet connection. Government internet connections run traffic through firewalls and gateways/filters, and block certain ports. This may also vary depending on whether connecting to Wifi or a physical LAN (ethernet) connection. SSH access often requires `port 22` to be available, and HTTPS often uses `port 443`. 

For this reason, developers must often rely on external connections that do not have these restrictions, such as wireless 4G dongles or mobile hotspotting, to connect to external services.

You can either connect to `projects.govcms.gov.au` via HTTPS using a Personal Access Tokens (PAT) or via SSH using SSH keys. Using SSH keys is generally easier, as once set up they automatically authenticate you without needing to enter a password every time.

There's two scenarios where HTTPS is used:

- If you use HTTPS for the repository URL when cloning, and
- When [retrieving a copy of the production website database container from the GitLab Container Registry](#importing-database).

Creating the personal access token is done via the GitLab web interface for projects.govcms.gov.au. The name of the token can be whatever you prefer, and the scope should be set to `read_registry`. **Note that you need to keep a copy of the token when you generate it because you cannot access it later.**

Check out the GitLab guides on:

* [creating and using SSH keys](https://docs.gitlab.com/ee/ssh/#generating-a-new-ssh-key-pair).
* [creating PATs](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) and [using them in Private Repositories](https://projects.govcms.gov.au/help/user/project/container_registry#using-with-private-projects).  

If using SSH, you can test and debug your connectivity with this: 

    ssh -Tv git@projects.govcms.gov.au

The first time you connect to a new Host, you may get a message like this. Just enter 'yes' to continue:

    The authenticity of this host has not been verified, do you want to add this host to the list of known hosts? 

If you create a new SSH key pair, and they do NOT use the default SSH key names `id_rsa` and `id_rsa.pub`, you may need to update your `~/.ssh/config` file to [use those particular keys for that particular connection](https://www.stackoverflow.com/questions/2419566/best-way-to-use-multiple-ssh-private-keys-on-one-client). If SSH still refuses the connection, try using your existing `id_rsa` key. 


### <a name="clone-it"></a>2. Clone the project repo

Once you're all connected, navigate to wherever you want to keep the project and clone it down. This command will pull down the project and place it in a directory called '${project_name}':

    SSH:    git clone git@projects.govcms.gov.au:${profile_name}/${project_name}.git ${project_name}
    HTTPS:  git clone https://projects.govcms.gov.au/${profile_name}/${project_name}.git ${project_name}

**Mac users only**

Confirm the project root path is in [Docker's file sharing config](https://docs.docker.com/docker-for-mac/#file-sharing). File sharing is required for volume mounting if the project lives outside of the `/Users` directory.

**Windows users only**

In addition to the project repo, also clone the `amazee-docker-windows` repo - this can live outside the project root.

    git clone https://github.com/amazeeio/amazeeio-docker-windows amazee-docker-windows

This is a public repo, so you don't need a PAT to access it. Attempting to use an SSH version of the URL will fail too, unless you have an account on Amazee's GitHub profile.

### <a name="build-it"></a>3. Build it and run it

**Make sure you don't have anything running on port `80` on the host machine (like an Apache web server from XAMP, WAMP, LAMP etc, or Skype).** 

If you do, you may need to change the port it runs on or turn it off. Windows users may also need to keep port `443` available for the `haproxy` Docker container.

You can check what is running on these ports with:

        Mac/Linux:  sudo lsof -i -P -n | grep LISTEN
        Windows:    netstat -anb | findstr :80

1. Ensure Docker is running. You can do this by running `docker container ps` in your command line. If you have to start it, it may take a while. 

2. Navigate into the project root

2. Start the local network

    **Mac/Linux users** 

        pygmy up
        
    Ubuntu 18 users may need to append `--no-resolver`

    **Windows users** (This bit must be run from within the `amazee-docker-windows` root)


    i. Create a new network for Docker. This only needs to be done once after Docker is installed, you can then start, stop, rebuild and destroy containers without setting up another:
    
        docker network create amazeeio-network
    
    ii. Start up the `amazee-docker-windows` containers:

        docker-compose up -d
        
    If Docker throws an error, run `docker-compose restart` to reboot your containers. If that doesn't work, you may need to quit Docker altogether and run it as an Administrator. See the [Amazeeio GitHub profile](https://github.com/amazeeio/docs/blob/master/local_docker_development/troubleshooting.md) for help.

3. Build and start the Docker containers (must be run from within the project root):

        Mac/Linux:  ahoy up
        Windows:    docker-compose up -d; docker-compose exec -T cli govcms-deploy

    The `govcms-deploy` command enables the `stage_file_proxy` module from witin the `cli` container.

4. Install the GovCMS Drupal profile via Drush into the new website container (this container is known as 'test'):

        Mac/Linux:  ahoy install
        Windows:    docker-compose exec -T test drush si -y govcms

    **Note**: This must be done before importing a backup of another site. 

5. Update the user account password using Drush:

        Mac/Linux:  ahoy drush upwd admin --password='${your-new-password}'
        Windows:    docker-compose exec -T test drush upwd admin --password='${your-new-password}'

6. Login to your newly-installed Drupal site using the domain this spits out (should be `${project_name}.docker.amazee.io`; the full URL prompts a password reset):

        Mac/Linux:  ahoy login
        Windows:    docker-compose exec -T test drush uli



## <a name="image-updates"></a>Images updates
Once your site is running correctly, you should check for updates to the images on which it runs regularly. To check for updates:

        Mac/Linux:  ahoy pull
        Windows:    docker image ls --format \"{{.Repository}}:{{.Tag}}\" | grep govcmslagoon/ | grep -v none | xargs -n1 docker pull | cat

If any updates are found, you'll need to rebuild your containers.



## <a name="importing-existing"></a>Importing an existing site

You can install a base govCMS site from this project, then import the files and database of your production site.

**Important!** 

- For imported backups to work, your local site must already have a vanilla govCMS site installed, otherwise you'll get a 503 error when loading the site. This is because the installation process includes scripts that must be run in multiple containers. See [Step 5 of Build it and Run it](#build-it)
- GovCMS Dashboard databases are automatically sanitized, so your normal login won't work! See the [Notes section](#notes) at the bottom for more information. 


### <a name="importing-database"></a>Importing a database

You can either import a local database file via Drush, or rest your fingers while Docker retrieves the latest production site database backup and imports it for you. The ability to take direct MySQL backups on demand is not currently supported.

#### Importing a local database dump

1. Download a `mysql` container backup from the [govCMS Dashboard](https://dashboard.govcms.gov.au). The resulting file should be saved into your defaul downloads location.

2. Navigate to the downloaded file location, and extract the MySQL container backup from the `tar` file:

        tar -xf ${backup_name}.tar 

3. Navigate to the project location, and import the database into the test container


        Mac/Linux:  ahoy mysql-import ${path/to/my/local-database}.sql
        Windows:    docker-compose exec test bash -c 'drush sql-drop' && docker-compose exec -T test bash -c 'drush sql-cli' < ${path/to/my/local-database}.sql

    **Mac/Linux users**: If running `ahoy mysql-import` gives you an error, the command may not exist in your `.ahoy.yml` file. If this is the case, just run the `docker-compose` version of the command instead.

    **Windows users**: You may get this error: `The input device is not a TTY. If you are using mintty, try prefixing the command with 'winpty'`.
    If this appears, just run the command again with `winpty ${command}`
    
3. Flush the caches and refresh any asset locations

        Mac/Linux:  ahoy drush cc all
        Windows:    docker-compose exec -T drush cc all


#### Importing the latest backup of the current production website database from the GitLab Container Registry

This option allows Docker to retrieve this container (which will probably take a while). Using this approach requires rebuilding the containers, therefore destroying the current local site's database. 

GitLab automatically saves the nightly production site database backups as containers in a private Container Registry. This allows the entire container to be restored including all settings and configuration, databases, tables etc, rather than just the database dump itself.

To retrieve the live site's most revent database container backup: 

1. In the file `.env`, uncomment the last line:
        
        MARIADB_DATA_IMAGE=gitlab-registry-production.govcms.amazee.io/${profile_name}/${project_name}/mariadb-drupal-data

2. Log into the Docker Contianer Registry with your GitLab Personal Access Token (your account password won't work)

        docker login gitlab-registry-production.govcms.amazee.io -u ${your GitLab login email} -p ${your Personal Access Token}
        
    **Windows users**: If you get an error, try prefixing the command above with `winpty `

3. Rebuild the containers

        Mac/Linux:  ahoy up
        Windows:    docker-compose up -d --build

This will import the latest database backup image from the Amazee Docker Registry over the top of the existing site.

If it works, your local site URL will load a copy of your prodution site. If this fails, you should just see a fresh govCMS site. If this happens, you can try manually pulling the latest database image backup, then rebuilding the containers:

    docker pull gitlab-registry-production.govcms.amazee.io/${profile_name}/${project_name}/mariadb-drupal-data
    docker-compose up -d --build

Once complete, you'll need to use [Drush to make your user account an administrator](#notes), [enable `stage_file_proxy`](#importing-files) and [other development modules](#dev-modules) ([See Step 6 of 'Build it and Run it'](#build-it)) etc. 


### <a name="importing-files"></a>Importing files

Files can be included in several ways.

1. Use the `stage_file_proxy` module to dynamically download any files needed to load each page you view on your Docker site (preferred method); or
2. Download a dump of the files into the local file system - this method uses [Docker Volumes](https://docs.docker.com/storage/volumes/) and will take up far more space locally depending on the size of the filebase.


#### To set up Stage File Proxy

The Stage File Proxy module is already included in govCMS SaaS, it just needs to be enabled. This can either be done via Drush or by running a govcms script.

**Drush:**

    Mac/Linux:  ahoy drush en -y stage_file_proxy
    Windows:    docker-compose exec -T drush en -y stage_file_proxy

**govCMS script:**

    docker-compose exec -T cli govcms-deploy

If you have already imported a SaaS production site database, Stage File Proxy should already have the internal production domain set as the source of the files. No further action required! 

If not, you'll need to set it up manually. This requires your [User account to have `administrator` access](#notes).

1. In a web browser, log into the site with an account that has administrator access
2. If it's not already enabled, Visit `${site-url}/admin/modules` and enable the `stage_file_proxy` module
3. Visit `${site-url}/admin/config/system/stage_file_proxy`
4. Enter the production site URL into the origin site field and save. You may want to use an internal URL that bypasses any caching systems to ensure you get the latest versions of files (if you have one set up).

Once Stage File Proxy is running, it will download and save copies of any assets requested by pages loaded in the Docker site, and save them automatically into the local `/files` directory on the Host machine. Docker maps this directory as a Volume to a corresponding location inside the containers.


#### Including a file dump in local Docker Volumes

'Volumes' in Docker are folders on the host machine that are mapped to a corresponding location inside one or more containers. Anything that happens to either the container folder or the host machine folder is immediately replicated in the others. 

Altering or adding new Volumes to containers requires updating the project's `docker-compose.yml` file, which is locked from editing. 

Instead, several local volumes already exist for the project, under `/files` and `/tests`. Any files added into these directories will be immediately available to the Docker containers.

The `/files` directory is mapped to `${site-url}/sites/default/files`.

e.g. adding a PDF file to `${project-root}/files/myfile.pdf` will make that file available under `${project_name}.docker.amazee.io/sites/default/files/myfile.pdf`. This is what Stage File Proxy does.

You *can* add a copy of the entire production site filebase, however this is a heavy-handed solution when the majority of files included probably won't be required for your development tasks.


## <a name="dev-modules"></a>Adding development-only modules

For _new_ modules to work i.e. ones not included in the govCMS installation profile, they need to be present in several Docker containers. Downloading them via Drush alone won't work, and you may get errors when trying to use them. 

To propagate a module across several locations in several containers, we can set up additional Docker Volumes via a `docker-compose.override.yml` file. 

**This file is ignored by the GitLab deployment pipeline, so it cannot break anything if/when pushed to the remote repository.**

This technique maps an additional local folder to multiple containers, using a [Docker Volume](#importing-files).

**Note: this technique involves rebuilding your containers, so any work within will be destroyed.**

1. Create a new file in the project root called `docker-compose.override.yml`, and paste in the following code:
```
version: '2.3'

services:
  cli:
    volumes:
      - ./dev_modules:/app/sites/default/modules/dev_modules:${VOLUME_FLAGS:-delegated}
  php:
    volumes:
      - ./dev_modules:/app/sites/default/modules/dev_modules:${VOLUME_FLAGS:-delegated}
  test:
    volumes:
      - ./dev_modules:/app/sites/default/modules/dev_modules:${VOLUME_FLAGS:-delegated}
```
2. Create a folder called `dev_modules` in the project root. **Remember to add it to gitignore, so your dev modules don't get saved in the project repository.**

3. Rebuild the Docker containers: 

        Mac/Linux:  ahoy down; ahoy build
        Windows:    docker-compose down; docker-compose up -d --build

4. Download the desired module to the `dev_modules` folder inside the `test` container (alternately, you can manually download modules into `dev_modules` locally and unzip them there, and they should also appear in the containers): 

        Mac/Linux:  ahoy drush dl ${module_name} --destination='/app/sites/default/modules/dev_modules'
        Windows:    docker-compose exec -T test drush dl ${module_name} --destination='/app/sites/default/modules/dev_modules'

    If you see a message showing a different path, check the [Known Issues](#known-issues).

5. You should be able to enable your new module from the Modules UI (if you are an Admin) or via drush: 

        Mac/Linux:  ahoy drush en ${module_name}
        Windows:    docker-compose exec -T test drush en ${module_name}


## <a name="shutting-down"></a>Shutting down your computer without losing the work inside your containers

Docker containers will remain intact unless you explicitly delete them. This means you can turn of your development machine without losing your work.

**NOTE**: Running `ahoy down` or `docker-compose down` will DESTROY your containers and any changes within (importantly, the database). **Use with caution!!**

To stop your containers for later use, run:

    Mac/Linux:  ahoy stop
    Windows:    docker-compose stop

To start them again later, run:

    Mac/Linux:  ahoy restart
    Windows:    docker-compose up -d

When the containers are stopped, you can safely shut down your computer without losing work.  

Changes made to files *outside* the containers, such as in `/files` and `/themes` will remain intact if the containers are stopped or deleted. 



## <a name="pushing-commits"></a>Pushing commits

Before pushing anything back up to GitLab, you should confirm your local Git user name and email. If this differs from those used by your GitLab account, your commits will show as originating from a different user (but will stil be permitted). 

You can check your global Git user account details by inspecting `user.name` and `user.email` via:
    
    git config --global --list

You can then check your _local_ Git details with:

    git config --local --list

You can then specify different user details for specific repositories using this:

    git config --local user.name '${your GitLab account username}'
    git config --local user.email ${your GitLab account email}



## <a name="known-issues"></a>Known issues and workarounds

1. This process only applies to the `7.x-3.x` branch of GovCMS
2. ~~Currently (Nov 2018), all local projects utilise the same LOCALDEV_URL. The URL used is hardcoded in. GovCMS is aware and working on a fix. To access different sites, shut down the containers of all except the one you want to see at that URL.~~
3. The container 'test' cannot have its name changed. This prevents Drupal from being able to connect to the database for some reason.
4. When logging into the site for the first time, the 'Reset password' page does not allow resetting the password, complaining  `Password may only be changed in 24 hours from the last change`. See [Step 6 of Setup](#) for the workaround. 
5. ~~Attempting to import database dumps from the govCMS Dashboard using `ahoy mysql-import ${database.sql}` will fail, as currently (10 Dec 2018) the Docker configuration only works with databases called `drupal`. See 'Importing a database, Step 2' below for the workaround.~~

**Issues running Docker on Windows**

1. Windows users may get build errors when running `docker-compose up -d`, noting the filepaths listed in the dockerfiles are not valid. This relates to how Windows interprets POSIX filepaths (which are `formatted/like/this`, as opposed to Windows paths, `which\look\like\this`). 

    [The fix](https://community.quantrocket.com/t/errors-on-deploying-quantrocket-mount-denied-nthe-source-path-var-run-docker-sock-var-run-docker-sock-nis-not-a-valid-windows-path/258/3) is to add this line to the project's `.env` file: `COMPOSE_CONVERT_WINDOWS_PATHS=1`. 

    Alternately, you can switch Docker between using Windows and Linux containers under Docker's settings. Changing this settings requires Docker to pull down the appropriate Container versions, then reboot.  

2. If you have Docker set to start automatically when Windows boots up, you may get an error like this when starting your containers:
    
    `Error starting userland proxy: mkdir /port/tcp:${host-ip}:${host-port}:tcp:${container-ip}:${container-port}: input/output error`
  
    This is a [known bug in Docker](https://github.com/docker/for-win/issues/1038), that relates to Windows Fast Boot. Either disable Fast Boot, or just restart Docker once Windows loads. Annoying but effective. 

3. If you're using Git Bash on Windows as your CLI, you may get an error like this when referencing locations inside Containers, such as when [Adding development modules](#dev-modules):

    `The directory C:/Program Files/Git/app/sites/default/modules/dev_modules does not exist.`

    This is a [known bug in Docker](https://github.com/docker/toolbox/issues/673). Git Bash misinterprets the path you specify in the command; use another CLI to run these commands, like Command Prompt.

4. If you have Docker set to automatically start when Windows boots, it may not start in Administrator mode, even if you have the program itself set to do so. You'll need to shut it down and manually start it in Admin mode. 

5. Even if you _disable_ the setting 'Start Docker Desktop when you log in', Docker may start at boot regardless. Check your Startup apps list (`Ctrl + Shift + Esc` > `Startup tab`) to see if Windows is forcing it to start.



## <a name="notes"></a>Notes 
- Windows users can find the `docker` commands listed in this guide inside the `.ahoy.yml` file.
- Unless you import a database dump from another site, the out-of-the-box govCMS site will only contain the user 'admin'.
- If you import a database dump from a site where your user account is NOT an administrator, you can become one by assigning your account the administrator role running:

        Mac/Linux:  ahoy drush urol 'administrator' ${account email, user ID or 'username in quotes'}
        Windows:    docker-compose exec -T test drush urol 'administrator' ${account email, user ID or 'username in quotes'}

    The super admin user ID will always be '1'.

- If you import a database from the govCMS Dashboard, it will be automatically sanitized. Emails and passwords will have changed, so to change them you must use the User ID or username.
- If a Docker build seems to be taking forever, check a password prompt from Docker hasn't opened somewhere requesting access to your hard drive. There's no progress indicator for the build steps, so it can be hard to tell if anything is happening. 


## TODO 

* Explain how to use drush
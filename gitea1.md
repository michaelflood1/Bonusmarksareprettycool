# Side Project : Remote Self Hosted Git Repository

[What is Gitea](#what-is-gitea)

[Initial set up](#initial-set-up)

[1. Task 1: Install packages for gitea and mariadb](#1-task-1-install-packages-for-gitea-and-mariadb)

[2. Task 2: MariaDB Initial Set-up](#2-task-2-mariadb-initial-set-up)

[3. Task 3: Initial Gitea Set-up](#3-task-3-initial-gitea-set-up)

- [3a. app.ini](#3a-appini)

- [3b. Find mysql in your script using nano](#3b-find-mysql-in-your-script-using-nano)

- [3c. Adjust Configuration to your preferences](#3c-adjust-configuration-to-your-preferences)

- [3d. Save and check work](#3d-save-and-check-work)

[4. git user creation + gitea/git chmods](#4-git-user-creation--giteagit-chmods)

- [4a. Choose a frontend framework](#4a-choose-a-frontend-framework)

- [4b. Create a git user with the following](#4b-create-a-git-user-with-the-following)

- [4c. Apply needed permissions](#4c-apply-needed-permissions)

[5. Task 5: Systemctl commands](#5-task-5-systemctl-commands)

- [5a. sudo systemctl start gitea](#5a-sudo-systemctl-start-gitea)

- [5b. sudo systemctl status gitea](#5b-sudo-systemctl-status-gitea)

- [5c. sudo systemctl stop gitea](#5c-sudo-systemctl-stop-gitea)

- [5d. sudo systemctl restart gitea / sudo systemctl reload gitea](#5d-sudo-systemctl-restart-gitea--sudo-systemctl-reload-gitea)

- [5e. sudo systemctl enable gitea](#5e-sudo-systemctl-enable-gitea)

- [5f. sudo systemctl disable gitea](#5f-sudo-systemctl-disable-gitea)

- [5g. sudo journalctl -u gitea](#5g-sudo-journalctl--u-gitea)

[6. Task 6: App.ini -- more configggg](#6-task-6-appini--more-configggg)

- [6a. Find [server] in your editor](#6a-find-server-in-your-editor)

- [6b. Configure to your preference](#6b-configure-to-your-preference)

- [6c. Find [service] in app.ini](#6c-find-service-in-appini)

[7. Task 7: finishing set up through local](#7-task-7-finishing-set-up-through-local)

- [7a. Initial connection use2shell 1 local one remote](#7a-initial-connection-use2shell-1-local-one-remote)

- [7b. Create your user](#7b-create-your-user)

[8. Task 8: SSH setup](#8-task-8-ssh-setup)

- [8a. Open your profile settings tab](#8a-open-your-profile-settings-tab)

- [8b. Click SSH/GPG Keys](#8b-click-sshgpg-keys)

- [8c. Add your key](#8c-add-your-key)

[9. Task 9: Git](#9-task-9-git)

- [9a. Create a repository](#9a-create-a-repository)

- [9b. Clone the repo to local](#9b-clone-the-repo-to-local)

- [9c. Pushing](#9c-pushing)

[Conclusion](#conclusion)

[Useful references](#useful-references)



### What is Gitea

Gitea is a platform for hosting and managing Git repositories much like Github.

The main differences are:

- that github is a central service that GITHUB has full control over all of your data and often contains a fee.

- Gitea is a free open-source service(FOSS) that you can set up to be as public or as private as you desire for any company or personal project.

- Gitea can be customized since you control it while Github is just whatever the company decides you get to do.

- Gitea takes far less resources from your machine to operate



## Initial set up

### 1. Task 1: Install packages for gitea and mariadb

    sudo pacman -S gitea
    sudo pacman -S mariadb
   

### 2. Task 2: MariaDB Initial Set-up

##### 2a. **optional** add an address that your server will listen to
    
    This command sets a specific IP address that your database will pay attention to.

    Defaults to 127.0.0.1 **AKA** localhost

    If you are using an ssh connection to work with your gitea/mariadb you can skip this as localhost is better for our needs.

        bind-address = 203.0.113.3

##### 2b. Initial db login

```bash
        sudo mariadb -u root -p

        or

        sudo mysql -u root -p 
        # this is an old command mariadb is the more current format
```
##### 2c. Create gitea user

```sql
        SET old_passwords=0;
        CREATE USER 'gitea'@'%' IDENTIFIED BY 'password';

         -- '%' signifies the ip you want to use, i used 'gitea'@'localhost'
         -- make sure you remember the password we will use it later
```
##### 2d. Create your 'gitea' database
    
```sql

        CREATE DATABASE gitea CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_bin'; 
        -- initiates database named gitea

        GRANT ALL PRIVILEGES ON 'gitea'.* TO 'gitea'@'localhost';
        FLUSH PRIVILEGES;
        exit

        -- you can test that you have connection to the new gitea database by running the following from your shell
        
        mysql -u gitea -h 203.0.113.3 -p  
``` 


### 3. Task 3: Initial Gitea Set-up

##### 3a. app.ini

Go to \etc\gitea\app.ini of whatever user installed the packages from step 1

        cd \etc\gitea\app.ini
    
this file is your life blood take your time with it or spend 5 hours backtracking like me 
    
:^)

##### 3b. Find **mysql** in your script using the superior text editor known as nano

    sudo nano app.ini

    Ctrl+f | mysql | ⎆ ENTER

##### 3c. Adjust Configuration to your preferences.

|;; default| Mine|
|--|--|
|DB_TYPE = mysql|DB_TYPE = mysql|
|HOST = 127.0.0.1:3306 ; |HOST = 127.0.0.1:3306 |
|NAME = gitea|NAME = gitea |
|USER = root|USER = gitea|
|;PASSWD = ;Use PASSWD = `your password` |PASSWD = Nanobetterthanvim #**step2c**| 
|;SSL_MODE = false ; |SSL_MODE = disable|
|;CHARSET_COLLATION = ; |*empty*|
|*empty*|PATH = /var/lib/gitea/data/gitea.db|
|*empty|SCHEMA = gitea|
|*empty*|LOG_SQL = false|

##### 3d. save and check work

    sudo systemctl enable gitea
    sudo systemctl start gitea
   

### 4. git user creation + gitea/git chmods

#####    4a. Choose a frontend framework.

make sure git is installed 

    sudo pacman -S git

#####    4b. create a git user with the following 

check if gits made

        sudo grep 'git' /etc/passwd

example proper git user created

        git:x:972:972:git daemon user:/:/usr/bin/git-shell

if you do not already have a git daemon created run

        sudo useradd --system --group --shell /bin/bash --no-create-home --disabled-login --disabled-password git

    # makes a **system** users (no loggyinny) creates a group with the same name as user, bin/bash is the specified shell, disables a home directory as this is a background service, disables login for security. disabling password stops the user from authenticating with a password if they somehow manage to log in.


#####    4c. Apply needed permissions

check these locations

        ls -l 
                gitea:gitea /var/lib/gitea
                gitea:gitea /etc/gitea
                gitea:gitea /var/log/gitea
        
if gitea is not the owner

            sudo chown -R gitea:gitea path
 

### 5. Task 5: Systemctl commands

#####    5a. sudo systemctl start gitea
- initializes server

#####    5b. sudo systemctl status gitea
- returns a detailed informational sheet of current status

#####    5c. sudo systemctl stop gitea
- closes the server

#####    5d. sudo systemctl restart gitea / sudo systemctl reload gitea
- *restart* toggles on off for the server 
- *both* **veryneeded** anytime you make changes to servers files i.e app.ini run this to see results
- *reload* adds new settings without toggling on/off

#####    5e. sudo systemctl enable gitea
- runs on boot

#####    5f. sudo systemctl disable gitea
- disables run on boot

#####    5g. sudo journalctl -u gitea
- shows logs for gitea service
- add f flag at end to see real time logs


### 6. Task 6: App.ini -- more configggg

##### 6a. find [server] in your editor

It will most likely look like this

    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
                        [server]
    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

##### 6b. Configure to your preference

you can add many different arguments into this. here is how i set mine up

; ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

[server]

SSH_DOMAIN = 24.144.87.45                                              
DOMAIN = 24.144.87.45                                                  
HTTP_PORT = 3000                                                      
ROOT_URL = http://24.144.87.45:3000/                                   
APP_DATA_PATH = /var/lib/gitea/data                                   
DISABLE_SSH = false                                                    
SSH_PORT = 22                                                          
LFS_START_SERVER = true                                                
LFS_JWT_SECRET = OysE0lNMBnFovH3G4CeF4zSX_6jEAEZHkXiYj99PEcY
OFFLINE_MODE = false 
; ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

|option|explained|
|--|--|
|SSH_DOMAIN = 24.144.87.45|defines the ip address for ssh address i used my remote servers ip|  
|DOMAIN = 24.144.87.45 |defines ip address of the gitea web service interface|
|HTTP_PORT = 3000| defines which port the http server listens on|
|ROOT_URL = http://24.144.87.45:3000/| base url of the gitea instance  |
|APP_DATA_PATH = /var/lib/gitea/data| sets the location to save server data i.e repositories|
|DISABLE_SSH = false| If true you will not be able to use ssh to affect the server|
|SSH_PORT = 22| basic port ssh uses. can adjust to other ports i.e 2222 22 is just useful since it is an open port by default for most computers|
|LFS_START_SERVER = true| enables the large file server to be started. if you are working with large repos enable this| 
|LFS_JWT_SECRET = OysE0lNMBnFovH3G4CeF4zSX_6jEAEZHkXiYj99PEcY | secret key used to verify web tokens|
|OFFLINE_MODE = false| if set to true gitea will not access external services|
|BUILTIN_SSH_SERVER_USER = gitea| makes it so gitea handles all ssh requests to the server. **ensure the user is the one with ownership over your /var/lib/gitea directory|


##### 6c. find [service] in app.ini

[service]
REGISTER_EMAIL_CONFIRM = false
ENABLE_NOTIFY_MAIL = false
DISABLE_REGISTRATION = false
ALLOW_ONLY_EXTERNAL_REGISTRATION = false
ENABLE_CAPTCHA = false
REQUIRE_SIGNIN_VIEW = false
DEFAULT_KEEP_EMAIL_PRIVATE = false
DEFAULT_ALLOW_CREATE_ORGANIZATION = true
DEFAULT_ENABLE_TIMETRACKING = true
NO_REPLY_ADDRESS = noreply.localhost


|option|explained|
|---|---|
|REGISTER_EMAIL_CONFIRM = false|if true new accounts get a verification email|
|ENABLE_NOTIFY_MAIL = false| sends admin updates on changes|
|DISABLE_REGISTRATION = false| disallows new users|
|ALLOW_ONLY_EXTERNAL_REGISTRATION = false| no accounts made use OAUTH to connect|
|ENABLE_CAPTCHA = false| verification if people are human, but if you are human you keep false|
|REQUIRE_SIGNIN_VIEW = true|users need to be signed in to access|
|DEFAULT_KEEP_EMAIL_PRIVATE = true| security keeps emails hidden|
|DEFAULT_ALLOW_CREATE_ORGANIZATION = false| only admin can make orgs|
|DEFAULT_ENABLE_TIMETRACKING = true| maintains strict timechecks on activity|
|NO_REPLY_ADDRESS = noreply.localhost| sets email address for sending notifications|




### 7. Task 7: finishing set up through local

#####    7a. initial connection **use2shell 1 local one remote* 
        
    **LOCAL**
    
        ssh -X 'connection'
        # -X flag sets X11 forwarding to true enabling the local GUI to view the remote servers graphics based processes

    **REMOTE**

        sudo systemctl status gitea
        # if not running start it
        sudo systemctl start gitea
    
    **LOCAL**

        Start-Process "firefox.exe" "http://<IP_ADDRESS:3000>"
        # starts firefox at specified ip :3000 is the port for your remotes localhost
        
        #you should now see a gitea page

##### 7b. create your user 

    1. read the initial page and if everything looks good accept it

    2. create a user with whatever method you have allowed # the first user is the admin of the server.

    3. you will now be at a page that is very similar to github.

    4. click your user ppicture at the top right if you were the first user there will be a site administration button

    5. read and learn the uses of the maintenence options. any time you add an ssh key or users it is a good practice to run it.


### 8. Task 8: ssh setup
#####    8a. open your profile settings tab
    
#####    8b. Click SSH/GPG Keys

#####   8c. Add your key

1. click add key
    
2. name it however you want

3. copy your public key into the box
        
        cat ~/path/to/key | clip
    
4. you will then be asked to verify.

        !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

        USE WINDOWS BASH OR SUFFER 40 MINUTES OF WONDERING IF YOU ARE DUMB
        
        windows bash prints the letters in a more lettery way which gitea verification requires

        even though powershell prints the same letters those letters are not as lettery and thus are worthless

        !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

4. copy the commands gitea gives you input them into windows bash and copy the returned info into your gitea verification box
    
5. if all done correctly a green box appears at the top that says the verification worked


### 9. Task 9: Git

#####   9a. create a repository

    do this from the website of the server unless you have added this line into your api in the [service] section

                ENABLE_PUSH_TO_CREATE = true

#####   9b. clone the repo to local

    copy the repo ssh or https addres and run

        git clone ssh://john@example.com/path/to/my-project.git 

        cd my-project 

        or

        git clone http://24.144.87.45:3000//path/to/my-project.git 

        cd my-project 
         

#####   9c. Pushing

        now that we have a git repo clone we make our changes then

        save

        git add .

        git commit -m 'commit message'

        git push -u --verbose origin "master"/"main" # whichever you use

        # we use flag -u to set the tracking branch to our local branch

        # --verbose on first push in case we need to debug

        

### Conclusion

We know have a working remotely hosted gitea server that we personally own and run.

### useful references

[gitea.com](https://about.gitea.com/)

[atlassian](https://www.atlassian.com/git/tutorials/setting-up-a-repository/git-clone/)

[gitea-github](https://github.com/go-gitea/gitea/blob/main/custom/conf/app.example.ini/)

[arch-gitea-tutorial](https://gist.github.com/mekatronik-achmadi/2fcfdce81745a7c62089fcb7d5368911)










### Troubleshooting tips

##### LOCAL - GITEA publickeydenied

use the below commands to set your commands using ssh to autamatically trigger a flag

this is useful since you cannot flag ssh -i while using git commands

    $env:GIT_SSH_COMMAND="ssh -v"  | verbose
    $env:GIT_SSH_COMMAND = "ssh -i C:\path\to\your\do-key -v"

if you can pull and push using http connection you may need to add a secondary config that anagrams how your first registers your ssh "name"

and make sure you have identitiesonly, this forces your identify file to be read

    Host gitea
        HostName "IPOFSERVER"
        User gitea
        PreferredAuthentications publickey
        IdentityFile pathtokey
        IdentitiesOnly yes
        port 22

    Host "IPOFSERVER"
        User gitea
        PreferredAuthentications publickey
        IdentityFile pathtokey
        IdentitiesOnly yes
        Port 22

since pushing to giteas ssh requires 

    git push origin master "user"@{domain}

we need to resolve both the user as well as the server ip

can also just run

Host IPOFSERVER gitea
    User gitea
    PreferredAuthentications publickey
    IdentityFile pathtokey
    IdentitiesOnly yes
    Port 22

if you dont plan on having multiple users for the remote server on your local machine

mariadb

things to check if anything goes wrong database wise

/etc/mysql/my.cnf

```sql
    mariadb -u gitea -p gitea
    mysql -u [username] -p -h [hostname] local connection to mysql table i.e gitea@localhost
        SHOW GRANTS FOR 'gitea'@'localhost';
        SELECT User, Host FROM mysql.user;
        \s
        use
        SHOW DATABASES; 
```

Generic issues

  bash vs powershell ssh verifying

    USE WINDOWS BASH BECAUSE GITEA DOESNT RECOGNIZE VERIFICATION VIA POWERSHELL

sudo su - user 

    imitates your current status as 'user'
    good way to check things in that users home dir

gitea server to port 3000

    curl http://localhost:3000 

returns any data getting pushed through :3000 local host : 3000
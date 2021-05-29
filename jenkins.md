# Udemy course

https://github.com/ricardoandre97/jenkins-resources
https://www.udemy.com/course/jenkins-from-zero-to-hero/learn

# section-3 Getting started

* Dashboard
* new-item
    * you can give a title
    * so-far we did a shell command.
* The dashboard contains the list of jobs.
* You click on a job and then you can click 'build now' to build the job.
    * click 'configure' to reconfig the job.
* Dashboard -> Job -> particulra-build-of-job
* You can add string parameter, list parameter
* Boolean parameter come as "true"

# section-4 .. ssh plugin

* Dashboard -> manage-jenkins -> manage-plugins
* Dashboard -> manage-jenkins -> configure-system -> ssh -> add-remote-hosts
* Dashboard -> manage-jenkins -> configure-system -> credentials -> (Stores:Jenkins) -> Global credentials -> Add-Credentials

# section-5 : AWS and mysql

* this line shows sql container is ready.
```
mysqld: ready for connections.
```
* get mysql cli
```
mysql -u root -p
```
* db commands
create database testdb;
show databases;
use testdb;
create table info (name varchar(20), lastname varchar(20), age int(2));
show tables;
desc info;
insert into info value('ricardo','gonzales',21);
select * from info;

## installing mqsql client and aws cli

* These are stiched in the docker-file for remote-host
```
yum install mysql

pip install awscli --upgrade
```

## AWS

* Created a free-tier. This should be deleted after an year!
* Use service S3
    * Just create one bucket. Give it a name. I gave 'lakshman-jenkins-mysql-backup'
* Use service IAM (Identity and Access Management)
    * Create user
    * Name : lakshman / Programmatic Access
    * Attach existing policies directly
        * Full-S3-Access (real-world . probably not a good idea)
    * No-Tags
    * Create User
    * There is a access-key-id and secret-access-key,console-login-link

## Manually backup mysql

mysqldump -u root -p -h db_host testdb  > /tmp/db.sql

```
# export AWS_ACCESS_KEY_ID=...
# export AWS_SECRET_ACCESS_KEY=...
# aws s3 cp /tmp/db.sql s3://lakshman-jenkins-mysql-backup/db.sql
```

# Section-6: Ansible

* Note that the jenkins/jenkins image's user is jenkins.
* We edited the jenkins image to run our Docker-image which is atop jenkins.
    * Note that since the data-folder is same, although we switched image, it just works!

## Inventory

* any file name.. we use hosts
  hosts
  ```
  [all:vars]

  ansible_connection = ssh

  [test]

  test1 ansible_host=remote_host ansible_user=remote_user ansible_private_key=/path
  ```

* I had a problem with control-dir too long. Added a $HOME/.ansible.cfg
    ```
    [ssh_connection]
    control_path = /tmp/%%h-%%p-%%r
    ```

* running ansible
    ```
    #ansible -i <inventory-file> -m <module> <module-args>
    ansible -i hosts -m ping test1
    ```

## Playbooks

* yaml format
* Eg file
    ```
        - hosts: test1
          tasks:
           - name: descriptive explanation
             template:
               src: locally-present-jinja-template-file
               dst: remote-localtion-in-host
           - shell: command
           - debug:
              msg: "{{AN_ANSIBLE_VARAIABLE}}"       # Note the variable-notation in ansible. -quotes and double-braces
    ```
    For course the file is now in `jenkins_home/ansible/play.yaml`
* invoking
    * I get a bunch of py-errors. but the thing works though
    ```
    ansible-playbook -i hosts play.yml -e "VARIABLE=value"
    ```

## Installing jenkins plugin

* Same as usual.
    * Dashboard -> Manage jenkins -> Manage plugins -> Available -> Ansible
    * Install/Restart

## Running Ansbile

* freestyle project
* Choose ansible playbook under build-type
    * you give the path to playbook
    * and path to inventory-file

* Colorizing output
    * Install the ansi-color plugin
    * in jenkins-project -> build-environment -> xterm color
    * in jeninks-project -> ansible-playbook-project -> colorize output

## Db

create database people;
use people;
create table register (id int(3), name varchar(50), lastname varchar(50), age int(3));
show tables;
desc register;
insert into register values(1,'ricardo','gonzales',21);

# Section-7 security

* ManageJenkins -> Configure GLobal Security
    * enable/disable security -- I dont see this. May be its gone in new versions?
    * allow users to sign up
        * all users have root-like access
* Install the role base auth plugin
* Manage-roles
    * Create new roles.
        * add what that roles can do.
* assign roles to users

* OVerall-read, build-read, build-create
* Item rules allow to pick up patterns to reduce subset

# section-8 Jenkins Tips & Tricks

* Environment variables
    * https://wiki.jenkins.io/display/JENKINS/Building+a+software+project
    * lists variables that are available in all jobs

* Build -> Global Properties -> Environment Variables
    * we can add our own global variables

* Jenkins URL
    * configure -> jenkins URL

* Build-configure
    * build periodically
        * schedule - cron expression
            * H inlieu of mintue will execute the job one by one at 0th minute

* Firing a jenkins script remotely.
    * Install the strict-crumb-issuer plugin
    * Get the crumb and use the crumb to fire
    ```
    crumb=$(curl -u "jenkins:1234" -s 'http://jenkins.local:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)')
    curl -u "remote_user:1234" -H "${crumb}" -X POST http://localhost:8080/job/test-env-vars/build\?delay\=0sec
    curl -u "jenkins:1234" -H "$crumb" -X POST  http://jenkins.local:8080/job/backup-to-aws/buildWithParameters?MYSQL_HOST=db_host&DATABASE_NAME=testdb&AWS_BUCKET_NAME=jenkins-mysql-backup
    ```
#section-9 jenkins and emil

* PLugin responsible is mailer plugin - installed as part of auto-suggested plugins

* Amazon - AWS Simple email service
    * verify a email address
* In jenkins, goto "E-mail Notification" at the end of configure page
    * smtp-server is from AWS "SMTP settings"
    * username/pass is from create smtp creditials in AWS

* post-build actions
    * send mail

#section-10 Jenkins & maven

* Install the maven plugin

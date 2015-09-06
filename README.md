# udacity-fullstack-p5

# SSH
* IP Address: 52.88.149.231
* SSH Port: 2200
* private key for the grader
To login you will use a line something like:
  `ssh -i ~/.ssh/grader.rsa grader@52.88.149.231 -p 2200`
where the `grader.rsa` file is the file that contains the private key for the grader account. This key will be provided in the Notes To Reviewer.

# Application URL
* http://ec2-52-88-149-231.us-west-2.compute.amazonaws.com
This was obtained by entering the ip address (in SSH above) into [this website](http://www.hcidata.info/host2ip.cgi).

# Configuration Changes
The following are the (hopefully) reproducible steps to set up the server. The steps are grouped together by the rows in the rubric, and the instructions associated with each row are listed, along with the steps to implement each piece of the instructions and rubric.

## User Management
### Instructions
* Create a new user named `grader`
* Give the grader the permission to sudo

### Rubric
* Meets: `Remote login of root user has been disabled, a remote user that can sudo to root has been defined, user passwords are set securely.`
* Exceeds: N/A

### Procedure
1. ssh in as root.
2. `adduser student` and give them the password `student` for the moment.
3. give `student` sudo priveleges.
  1. `nano /etc/sudoers.d/student`
  2. Add the line `student ALL=(ALL) NOPASSWD:ALL`
  3. Save the file and close it.
4. Passwords have been disabled for SSH, so create a login keypair for student: on your LOCAL machine (NOT! the server):
  1. `ssh_keygen`; when prompted use the default directory, but change the filename to `student`.
  2. Copy the public key to the server
    1. mkdir `/home/student/.ssh`
    2. touch `/home/student/.ssh/authorized_keys`
    3. ON YOUR LOCAL MACHINE cat the `student.pub` file created in 4.1, and copy the entire output.
    4. ON THE SERVER
      1. paste the output from 4.2.3 into `/home/student/.ssh/authorized_keys`.
      2. `chmod 700 /home/student/.ssh`
      3. `chmod 644 /home/student/.ssh/authorized_keys`
      4. `chown student:student /home/student/.ssh`
      5. `chwon student:student /home/student/.ssh/authorized_keys`
5. ssh in as `student`; all work from here on will be done as student, not root.
6. Repeat steps 2 throgh 4 substituting `grader` for `student`. NOTE: you will need to sudo many of the commands because you are not root.
  * NOTE: if you use a passphrase when you create `grader` be sure to write it down (assuming you want someone to be able to login with that key).
  * You now have a a user named `grader` who can remotely log in and has permission to sudo.
7. Disable remote root login by editing `/etc/ssh/sshd_config` and changing `PermitRootLogin` to `no`. [Source](https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user)

## Security

### Instructions
* Update all currently installed packages
* Change the SSH port from 22 to 2200
* Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

### Rubric
* Meets: `Key-based SSH authentication is enforced, SSH is hosted on a non-default port, applications have been updated to most recent updates.`
* Exceeds: `The firewall has been configured to monitor for repeat unsuccessful login attempts and appropriately bans attackers; cron scripts have been included to automatically manage package updates.`

### Procedure
1. Upgrade packages to the current versions: `sudo apt-get update && sudo apt-get upgrade -q -y` - this works pretty well, but you still get a grubmenu prompt that you have to handle manually.
2. Add daily unattended package updates [Source](https://www.howtoforge.com/how-to-configure-automatic-updates-on-debian-wheezy):
  1. `apt-get install unattended-upgrades`
  2. `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades` and un-comment the line `"${distro_id}:${distro_codename}-updates";` so that updates are automagically installed. (Note: the `security` line should already be un-commented, but if it's not, un-comment that as well.)
  3. `sudo nano /etc/apt/apt.conf.d/02periodic` and add the following lines: `
// Enable the update/upgrade script (0=disable)
APT::Periodic::Enable "1";

// Do "apt-get update" automatically every n-days (0=disable)
APT::Periodic::Update-Package-Lists "1";

// Do "apt-get upgrade --download-only" every n-days (0=disable)
APT::Periodic::Download-Upgradeable-Packages "1";

// Run the "unattended-upgrade" security upgrade script
// every n-days (0=disabled)
// Requires the package "unattended-upgrades" and will write
// a log in /var/log/unattended-upgrades
APT::Periodic::Unattended-Upgrade "1";

// Do "apt-get autoclean" every n-days (0=disable)
APT::Periodic::AutocleanInterval "7";`
3. Change the SSH port from 22 to 2200:
  1. `sudo nano /etc/ssh/sshd_config`
  2. Change `Port 22` to `Port 2200`
  3. `sudo service ssh restart`
4. Configure the firewall:
  1. `sudo ufw default deny incoming`
  2. `sudo ufw default allow outgoing`
  3. `sudo ufw limit 2200`
  4. `sudo ufw allow 80`
  5. `sudo ufw allow 123`
  6. `sudo ufw enable`, and answer `y` at the prompt.
  7. `sudo reboot`
## Application Functionality
### Instructions
* Configure the local timezone to UTC
* Install and configure Apache to serve a Python mod_wsgi application
* Install and configure PostgreSQL:
  * Do not allow remote connections
  * Create a new user named `catalog` that has limited permissions to your catalog application database
* Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

### Rubric
* Meets: `Web-server has been provided to serve the provided application, database server has been configured to serve data, VM can be remotely logged into.`
* Exceeds: `The VM includes monitoring applications that provide automated feedback on application availability status and/or system security alerts.`

### Procedure
1. Configure local timezone to UTC
  1. `sudo nano /etc/timezone`
  2. if it's not already, change the timezone to `Etc/UTC` and save the file.
  3. `sudo service cron restart`
2. Install and configure Apache to serve a Python mod_wsgi application: `sudo apt-get -y install apache2 libapache2-mod-wsgi`
3. Install and configure PostgreSQL
  1. `sudo mkdir /var/postgres`
  2. `export PGDATA`
  3. `PGDATA=/var/postgres`
  4. `sudo apt-get -y install postgresql-9.3`
  5. `sudo chown postgres:postgres /var/postgres`
4. Create a new user named `catalog` that has limited permissions to your catalog application database
  1. `sudo useradd -p catalog catalog`
  2. `sudo --user=postgres createuser -DRS catalog`
  3. `sudo --user=postgres psql -c "ALTER USER catalog WITH PASSWORD 'catalog'"`
  4. `sudo --user=postgres psql -c "CREATE DATABASE catalog"`
  5. `sudo --user=postgres psql -c "GRANT ALL ON DATABASE catalog TO catalog"`
5. Install git: `sudo apt-get -y install git`
6. Clone repository from project 3 (but with the fork that uses postgresql instead of sqlite)
  1. `sudo mkdir /catalog`
  2. `sudo git clone -b PostgreSQL https://github.com/barefootlance/udacity-fullstack-p3.git /catalog`
7. Install libraries needed for the project to run
  1. `sudo apt-get -y install python-pip libpq-dev python-dev`
  2. `sudo pip install psycopg2` (needed to access postgresql)
  3. `sudo pip install -r /catalog/fullstack-nanodegree-vm/vagrant/catalog/requirements.txt`
8. Setup the catalog database `sudo -u catalog python /catalog/fullstack-nanodegree-vm/vagrant/catalog/database_setup.py`
9. Change the local authentication method for postgresql from peer to md5
  1. `sudo nano /etc/postgresql/9.3/main/pg_hba.conf`
  2. Page down to the table, then change the Method column for Local from peer to md5.
10. Create the wsgi file to launch the application:
  1. `sudo mkdir /var/www/catalog`
  2. `sudo nano /var/www/catalog/catalog.wsgi`
  3. Insert the following lines (remember, INDENTATION COUNTS!):
<br>`import sys`
<br>`sys.path.insert(0, "/catalog/fullstack-nanodegree-vm/vagrant/catalog")`
<br>`from project import app as application`
<br>`application.secret_key = 'ultra_secret_key'`
11. Create the apache webserver configuration file:
  1. `sudo nano /etc/apache2/sites-enabled/000-default.conf`
  2. Delete the entire contents
  3. Paste in the following lines:
<br>`<VirtualHost *:80>`
<br>`  WSGIDaemonProcess catalog`
<br>`  WSGIScriptAlias / /var/www/catalog/catalog.wsgi`
<br>`  ErrorLog ${APACHE_LOG_DIR}/error_catalog.log`
<br>`  CustomLog ${APACHE_LOG_DIR}/access_catalog.log combined`
<br>`  DocumentRoot /catalog/fullstack-nanodegree-vm/vagrant/catalog`
<br>`  <Directory /var/www/catalog/>`
<br>`    WSGIProcessGroup catalog`
<br>`    WSGIApplicationGroup %{GLOBAL}`
<br>`    Order allow,deny`
<br>`    Allow from all`
<br>`  </Directory>`
<br>`  ServerAlias ec2-52-88-149-231.us-west-2.compute.amazonaws.com`
<br>`</VirtualHost>`
  4. `sudo apache2ctl restart`

# udacity-fullstack-p5

# Project

## User Management
### Instructions
* Create a new user named `grader`
* Give the grader the permission to sudo
### Rubric
* Meets: `Remote login of root user has been disabled, a remote user that can sudo to root has been defined, user passwords are set securely.`
* Exceeds: N/A
### Procedure


## Security
### Instructions
* Update all currently installed packages
* Change the SSH port from 22 to 2200
* Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
### Rubric
* Meets: `Key-based SSH authentication is enforced, SSH is hosted on a non-default port, applications have been updated to most recent updates.`
* Exceeds: `The firewall has been configured to monitor for repeat unsuccessful login attempts and appropriately bans attackers; cron scripts have been included to automatically manage package updates.`
### Procedure

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


## Configuration File Comments
### Instructions
* N/A
### Rubric
* Meets: `Comments are present and effectively explain longer code procedures.`
* Exceeds: N/A

## Documentation
### Instructions
* N/A
### Rubric
* Meets: `A README file is included detailing all the steps to successfully run the application.`
* Exceeds: N/A
### Procedure

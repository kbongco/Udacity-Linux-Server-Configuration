# Udacity-Linux-Server-Configuration
This is the third project of Udacity's Full Stack Nanodegree. In this project, we were tasked to 
deploy a previous project done earlier in the Udacity Full Stack Nanodegree, which is the 
<a href="https://github.com/kbongco/Video-Game-Item-Catalog">Item Catalog</a>. In this project, 
using Amazon Lightsail and a Linux distribution on a virtual machine we learned how to host the
web application. 

<h2>Some important information</h2>
IP Address: 18.234.199.216<br>
Accessible SSH port: 200<br>
Visit: <a href="http://18.234.199.216">http://18.234.199.216</a> To see it deployed!

<h2>Tools Used</h2>
<ul>
<li>Amazon Lightsail</li>
<li>Virtual Machine</li>
<li>Python</li>
<li>PSQL</li>
<li>Git</li>
</ul>

<h2>Setting up the Server</h2>
First an instance must be set up on Amazon Lightsail via your local machine. 
<ol>
<li>After creating an instance, you can download a private key which is found on the 
account page. The private key is found in SSH keys, and the key can be downloaded 
from there. The file that will have a <b>.pem</b> extension</li>
<li>Open up the file, and copy the text and create a file called lightsail_key.rsa 
or any other file and place it in the .ssh directory</li>
<li>Run <b>chmod 600 ~/.ssh/lightsail_key.rsa</b>. This will give permissions to access
the file.</li>
<li>Run <b>ssh -i ~/.ssh/lightsail_key.rsa ubuntu@xx.xxx.xxx.xx</b> . The xx.xxx.xxx.xx refers
to the ip address listed on your instance. You will be logged into the instance. 
</ol>

<h4>Update everything</h4>
<ol>
<li> Run <b>apt-get update</b> This will update all the existing packages.</li>
<li> Run <b>apt-get upgrade</b> This will upgrade all the installed packages</li>
</ol>

<b>Configuring the UFW (Uncomplicated Firewall)</b>
<ol>
<li>Before configuring the firewall check the status of the firewall by running the command
<b>sudo ufw status</b>.</li>
<li>Open up the <b>sshd_config</b> file by running the command <b>sudo nano /etc/ssh/sshd_config</b>
On the line asking about what port number to listen to change it from 22 to 2200. Then restart
the service using <b>sudo service ssh restart</b></li>
<li>Run <b>sudo ufw default deny incoming</b>. This will set the firewall to block 
everythign coming in. </li>
<li>Run <b> sudo ufw default allow outgoing</b>. This will set the firewall to 
allow everything outgoing. </li>
<li>Run <b>sudo ufw allow ssh</b>. This will allow SSH.</li>
<li>Run <b>sudo ufw allow 2200/tcp</b> to allow connections for port 2200, this will help 
it run SSH</li>
<li>Run <b>sudo allow www</b> This will allow basic HTTP server</li>
<li>Run <b>sudo ufw allow 123/udp</b>. This will set the ufw firewall to allow NTP.</li>
<li>Run <b>sudo ufw deny 22</b></li>
<li>Run <b>sudo ufw enable</b></li>
<li>Run <b>sudo ufw status</b></li>
<li>Update the Amazon Lightsail firewall on the browser by changing the firewall configuration,
remove 22, and enable only 80 (TCP), 123(UDP), and 2200(TCP).</li>
<li>The connection will reset and now you can only log in using the command: <b> ssh -i ~/.ssh/lightsail_key.rsa p -2200 ubuntu@xx.xxx.xxx.xx</b></li>
</ol>

<h2>Creating the user grader</h2>
One of the tasks given was to create a user called grader. This user will be given sudo 
permissions for the reviewer. 
<ol>
<li>Run <b>sudo adduser grader</b></li>
<li>A password will be asked for as well as the information. Enter the password for this user
and if you want, the other information can be left blank</li>
<li>Run the command <b>su - grader</b> and enter the password to see if it goes through!
These steps are taken to give the grader sudo permissions:<br>
<ol>
<li>Create a new file under the sudoer directory using <b>sudo nano /etc/sudoers.d/grader</b>
Add the following test: grader ALL=(ALL:ALL) then save it.</li>
</ol>
</ol>

<h2>Allow the grader to log into the virtual machine</h2>
<ol>
<li>On your local machine run <b>ssh-keygen</b>.</li>
<li>Choose a file name for the key pair. I named mine grader_key. You will be asked
a passphrase and other information, the passphrase will be provided in the extras.</li>
<li>Log into your virtual machine and create a <b>.ssh</b> directory using <b>mkdir .ssh</b></li>
<li>Run the command <b>touch .ssh/authorized_keys</b></li>
<li>Go back to your local machine and run <b>cat ~/.ssh/grader_key.pub</b> Copy the code
and go back to your virtual machine and enter it in <b>.ssh/authorized_keys</b></li>
<li>Change the owner permissions using <b>chmod 700 .ssh</b> and <b>chmod 644 .ssh/authorized_keys</b></li>
<li>You can now log in as a grader using: <b>ssh -i ~/.ssh/grader_key -o 2200 grader@xx.xxx.xxx.xx</b></li>
</ol>

<h2>Install Fail2ban</h2>
<ol>
<li>Install Fail2Ban using <b>sudo apt-get install fail2ban</b></li>
<li>Install sendmail for email using <b>sudo apt-get install sendmail iptables-persistent</b></li>
<li>Create a copy of a file using <b>sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local</b></li>
<li>Change the settings in the file you just copied with:<br>
    set bantime = 600<br>
    destemail = youremail@domain<br>
    action = %(action_mwl)s<br>
</li>
<li>In that file, under <b>sshd</b> change <b>port = ssh</b> to 2200</li>
<li>Restart the service using <b>sudo service fail2ban restart</b> and you will get an email
which lets you know its running!</li!>
</ol>

<h2>Preparing to Deploy!</h2>
NOTE: If the timezone is not UTC you can use sudo dkpg-reconfigure tzdata to change the 
timezone. 

<h3>Installing WSGI and Apache</h3>
<ol>
<li>Install Apache using <b>sudo apt-get install apache2. If apache is working 
you will see the ubuntu page when you go to <a href="http://18.234.199.216"></a>.</b></li>
<li>Install mod_wsgi using <b>sudo apt-get install libapache2-mod-wsgi python</b></li>
<li>Enable mod_wsgi using <b>sudo a2enmod wsgi</b></li>
<li>Restart apache using <b>sudo service apache2 restart</b></li>
<li>Check if python is installed by typing <b>python</b></li>
</ol>

<h3>Install the various Python Packages and PSQL </h3>
Install any of the following packages that are needed in order to run your webapp:<br>
<ul>
<li><b>sudo apt-get install python-pip</b></li>
<li><b>sudo apt-get install python-Flask</b></li>
<li><b>sudo apt-get install python-httplib2</b></li>
<li><b>sudo apt-get install python-request</b></li>
<li><b>sudo apt-get install python-oauth2</b></li>
<li><b>sudo apt-get install python-sqlalchemy</b></li>
<li><b>sudo apt-get install python-psycopg2</b></li>
</ul>
Also, install POSTgreSQL with <b>sudo apt-get install postgresql</b>
NOTE: Check to make sure that PSQL does not allow remote connections! 

<h3>Creating a new PSQL for catalog</h3
<ol>
<li>Switch to the user postgres by running <b>sudo su - postgres</b></li>
<li>Connect to PSQL by typing <b>psql</b></li> 
<li>Create the catalog user by running <bCREATE ROLE catalog WITH LOGIN;</b></li>
<li>Give the catalog user the ability to create databases by <b>ALTER ROLE catalog CREATEDB;</b></li>
<li>Next, give the catalog user a passowrd by running <b>\password catalog.</b> </li>
<li>Check to see if the catalog user was created by running <b>\du</b> </li>
<li>A table will appear showing all the roles added check on the cataog role if it is able to create</li>
<li>Exit by using <b>\q</b> and log out of the postgres user using <b>exit.</b></li>
<li>Next create a user named catalog using <b>sudo adduser catalog.</b> to give sudo permissions.</li>
<li>Create a database while logged in as catalog using <b>creaddb catalog.</b> </li>
<li>Run psql and run <b>\l to see that the new database was created then quit and switch back to the grader user.</b></li>
</ol>

<h3>Create a User called catalog and database!</h3>
<ol>
<li>Create a user called catalog using: <b>sudo adduser catalog</b></li>
<li>Give catalog sudo access by going into <b>sudo visudo</b> and adding catalog ALL=(ALL:ALL) under
the root line.</li>
<li>Log in as catalog using <b>sudo su - catalog</b></li>
<li>exit using <b>exit</b></li>
</ol>

Note: If you do not have git installed for these next steps, install git using
<b>sudo apt-get install git</b>

<br>
<h3>Cloning from github and modifiying!!</h3>
<ol>
<li>While logged in as grader, create <b>/var/www/project</b> change to that directory 
then clone the <a href="https://github.com/kbongco/Video-Game-Item-Catalog">Video game Catalog</a>
using git clone.</li>
<li>Change to the directory where it was cloned and change the owner using <b>chwon -R gradergrader catalog</b></li>
<li>In the project.py file, change the line that says app.run(host='0.0.0.0' port=8000) to app.run()</li>
<li>In the database_setup.py, project.py, and gamedatabase.py files change the line for create_engine to 
<b>create_engine('postgresql://catalog:passwordhere@localhost/catalog')</b>. In the project.py 
file in the create_engine there is a check_same_thread, in the line which needs to be removed
to run properly</li>
</ol>

<h3>Configure a wsgi file and Virtual Host</h3>
The WSGI file must be in the same directory as the rest of your app files!!
<ol>
<li>Create the .wsgi file in the directory /var/www/project by using the command 
<b>sudo nano project.wsgi</b></li>
<li>The following lines of code will be added in:<br>
<b>import sys<br>
import logging<br>
logging.basicConfig(stream=sys.stderr)<br>
sys.path.insert(0,"/var/www/project")<br>
<br>
from project import app as application<br>
application.secret_key = 'Add your secret key'<br></b>
</li>
</li>Next, create the virtual host config file using <b>sudo nano /etc/apache2/sites-available/project.py</B>
and enter the following lines of code: <br>
<br>
<b><VirtualHost *:80><br>
                ServerName 18.234.199.216<br>
                ServerAdmin whoa.its.chibi@gmail.com<br>
                WSGIScriptAlias / /var/www/project/projec$<br>
                WSGIDaemonProcess project user=grader hom$<br>
                <Directory /var/www/project/><br>
                        WSGIProcessGroup project<br>
                        WSGIApplicationGroup %{GLOBAL}<br>
                        WSGIScriptReloading On<br>
                        Order allow,deny<br>
                        Allow from all<br>
                </Directory><br>
                ErrorLog ${APACHE_LOG_DIR}/error.log<br>
                LogLevel warn<br>
                CustomLog ${APACHE_LOG_DIR}/access.log co$<br>
</VirtualHost><br></b>
</li>
<li>Enable the virtual host using <b>sudo a2ensite catalog</b></li>
<li>Restart apache using <b>sudo service apache2 reload</b></li>
<li>Disable the default apache page using <b>sudo a2dissite 000-default.conf</b></li>
</ol>

<h3>Set up the database and run it! </h3>
<ol>
<li>Run <b>sudo python database_setup.py<b></li>
<li>Run <b>sudo python gamedatabase.py</b> If successful it will say "games and platforms added!</li>
<li>Restart apache one last time using <b>sudo service apache2 reload</b></li>
<li>Go to http://18.234.199.216/ to see the application running!</li>
<li>If there are errors check out the apache error log with <b>sudo tail /var/log/apache2/error.log</b></li>
</ol>

<h3>Sources, Various Links that helped with this project!</h3>
<ul>
<li>Various Github Repositories</li>
<ul>
<li><a href="https://github.com/kongling893/Linux-Server-Configuration-UDACITY">kongling893/Linux-Server-Configuration-UDACITY</a></li>
<li><a href="https://github.com/boisalai/udacity-linux-server-configuration">boisalai/udacity-linux-server-configuration</a></li>
<li><a href="https://github.com/iliketomatoes/linux_server_configuration">iliketomatoes/linux_server_configuration</a></li>
<li><a href="https://github.com/yiyupan/Linux-Server-Configuration-Udacity-Full-Stack-Nanodegree-Project">yiyupan/Linux-Server-Configuration-Udacity-Full-Stack-Nanodegree-Project
</a></li>
</ul>
<li><a href="https://www.udacity.com/course/configuring-linux-web-servers--ud299">Udacity Configuring Linux Web Servers</a></li>
<li><a href="https://www.ionos.com/community/server-cloud-infrastructure/apache/how-to-fix-http-error-code-500-internal-server-error/">How to Fix HTTP error code "500 Internal server error</a></li>
<li><a href="https://mudspringhiker.github.io/deploying-a-flask-web-app-on-lightsail-aws.html">How I deployed a flask app using Lightsail on AWS</a></li>
<li><a href="https://www.jakowicz.com/flask-apache-wsgi/">Running a Flask application under the Apache WSGI module</a></li>
<li><a href="https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys">SSH Essentials Working with SSH servers, clients and keys</a></li>

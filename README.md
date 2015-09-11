# Project-5_Linux-Server-Configuration
2015.09.09
Athor: Mikhail Rozhkov


This is a Project 5 - Linux-Server-Configuration under the Udacity Nanodegree program "Full Stack Web Developer"


i. The IP address and SSH port so your server can be accessed by the reviewer.
---------------------------- 
	37.143.11.88 
	SSH port 22


ii. The complete URL to your hosted web application.
---------------------------- 
	http://37.143.11.88:3000/


iii. A summary of software you installed and configuration changes made.
---------------------------- 
	Ste 1: Set up users 

		Create user:
			adduser student

		Install some utilities':
			apt-get install sudo
			apt-get install man
			apt-get install nano


		Provide student root priveleges 
			gpasswd -a student sudo

		Set user password to expire 
			passwd -e student

		Setup SSH authentification 
			Generate SSH key pair on LOCAL machine !!! 
				ssh-keygen
			Place public key to the remote server 
			log into remote server as 'student': ssh student@37.143.11.88
				mkdir .ssh (within 'home' directory)
				touch .ssh/authorized_keys (create file to store pub keys)
				nano .ssh/authorized_keys (open file in editor and copy there my public key, save)
				chmod 700 .ssh (set permission)
				chmod 644 .ssh/authorized_keys
			
			All done! Log in as a student user, authorizing by ssh public key:
				ssh student@37.143.11.88 -p 22 -i [PATH_TO_PUBLIC_KEY]

				ssh student@37.143.11.88 -p 22 -i ~/.ssh/linux_ihc

			Change the default SSH port
				sudo nano /etc/ssh/sshd_config
				sudo service ssh restart
				(check login using ssh pub-key)

			Disable password login:
				sudo nano /etc/ssh/sshd_config
				-> set 'PasswordAuthentification' to 'no'
			Restart the service
				sudo service ssh restart


		Install required Linux packages 
			sudo apt-get install finger
			sudo apt-get install ufw
			sudo apt-get install python
			sudo apt-get install git

	Step 2: Install python-pip
			sudo apt-get install python-pip 
		
	Step 3: Install virtualenv
			sudo pip install virtualenv (into FlaskApp folder)

	Step 4: Setup virtual environment. Install requried libraries (used in the FlaskApp:
		
		Create and activate virtual environment:
			sudo virtualenv venv
			source venv/bin/activate 

			(to deactivate virtual environemnt use 'deactivate' command)

		Install additional python libraris:
			sudo pip install bleach
			sudo pip install oauth2client
			sudo pip install requests
			sudo pip install httplib2
			sudo pip install flask
			sudo apt-get install sqlite3 libsqlite3-dev
			sudo pip install sqlalchemy


	Step 5:  Create folder 'projects'
		mkdir projects
		cd projects
		git clone .... FlaskProject

	Step 6: Udpate App OAuth2 settings 
		1) Update Facebook MyApp settings (https://developers.facebook.com)
			-> add http://37.143.11.88:3000/ to 'Site URL'
		2) Update App settings in Google Developer Console (https://console.developers.google.com)
			-> add http://37.143.11.88:3000/  to 'Authorized JavaScript origins'

	Step 7: Add Facebook and Google dev IDs and SECRETs into the files: 
		-> client_secrets_fb.json
			sudo nano client_secrets_fb.json
		-> client_secrets_gl.json
			sudo nano client_secrets_gl.json

	Step 8: Update App's IP and port in 'finalproject.py'
		app.run(host = '0.0.0.0', port = 8080)
		becomes
		app.run(host = '37.143.11.88', port = 3000)

	Step 9: Populate database and test run
		run database_setup.py to create the database
			sudo python database_setup.py
		run lotsofprojects.py to populate the database
			sudo python lotsofprojects.py
		test run finalproject.py and navigate to 37.143.11.88:3000 in your browser
			sudo python finalproject.py


	Step 10: Update security settings on server

		Configure firewall using 'ufw':
			sudo ufw default deny incoming
			sudo ufw default allow outgoing
			sudo ufw allow 53185/tcp
			sudo ufw allow 3000/tcp (port used to configure App in finalproject.py)
			sudo ufw enable 

		To enable logging use:
			sudo ufw logging on

		(To reset everything: sudo ufw reset)


	Step 11: Running uWSGI via Upstart

		Install uWSGI: 
			(source venv/bin/activate)
			sudo pip install uwsgi

		First, set up the directories the service will use.
			# Create a directory for the UNIX sockets
			sudo mkdir /var/run/flask-uwsgi
			sudo chown www-data:www-data /var/run/flask-uwsgi

			# Create a directory for the logs
			sudo mkdir /var/log/flask-uwsgi
			sudo chown www-data:www-data /var/log/flask-uwsgi

			# Create a directory for the configs
			sudo mkdir /etc/flask-uwsgi

		Next, create the config file.
			Create flask-uwsgi.conf: 
				cd /etc/init
				sudo touch flask-uwsgi.conf

			Add lines below to the flask-uwsgi.conf: sudo nano flask-uwsgi.conf
				start on startup
				start on runlevel [2345]
				stop on runlevel [06]

				script
				    cd /home/student/projects/Project-3_Item-Catalog
				    exec python finalproject.py
				end script

		Next, create the init script:
			Create flask-uwsgi.ini: 
				cd /etc/flask-uwsgi
				sudo touch flask-uwsgi.ini

			Add lines below to the flask-uwsgi.conf: sudo nano flask-uwsgi.ini
			(Set uid and gid to the numeric uid and gid for www-data.)
				[uwsgi]
				socket = /var/run/flask-uwsgi/flask-uwsgi.sock
				home = env
				wsgi-file = /home/student/projects/Project-3_Item-Catalog/finalproject.py
				callable = app
				master = true
				; www-data uid/gid
				uid = 1
				gid = 1
				die-on-term = true
				processes = 4
				threads = 2
				logger = file:/var/log/flask-uwsgi/flask-uwsgi.log
		
		Next, start uWSGI:
			sudo service flask-uwsgi start
			(to stop: sudo service flask-uwsgi stopt)

		Next, check that uWSGI is running.
			sudo less /var/log/flask-uwsgi/flask-uwsgi.log


	Step 12: Additional security settings

		Install Fail2Ban on Ubuntu 14.04
			sudo apt-get update
			sudo apt-get install fail2ban

		Configure Fail2Ban with your Service Settings
			We need to copy this to a file called jail.local for fail2ban to find it:	
				sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
			
			Once the file is copied, we can open it for editing to see how everything works:
				sudo nano /etc/fail2ban/jail.local
				ignoreip = 127.0.0.1/8
				bantime = 600
				findtime = 600
				maxretry = 3
				destemail = mikhail_job@mail.ru  (email address that should receive ban messages)
				sendername = Fail2Ban  ("From" field in the email)
				mta = sendmail (mta parameter configures what mail service will be used to send mail)



iv. A list of any third-party resources I've used of to complete this project.
---------------------------- 
	1. Initial Server Setup with Ubuntu 14.04 (https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)
	2. How To Edit the Sudoers File on Ubuntu and CentOS (https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos)
	3. Additional Recommended Steps for New Ubuntu 14.04 Servers (https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers)
	4. UFW - Uncomplicated Firewall (https://help.ubuntu.com/community/UFW)
	5. How To Deploy a Flask Application on an Ubuntu VPS (https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
	6. Flask with uWSGI + Nginx (https://github.com/mking/flask-uwsgi)
	7. How To Protect SSH with Fail2Ban on Ubuntu 14.04 (https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)
	8. Quickstart for Python/WSGI applications (http://uwsgi-docs.readthedocs.org/en/latest/WSGIquickstart.html)
	9. Upstart. Get started (http://upstart.ubuntu.com/getting-started.html)
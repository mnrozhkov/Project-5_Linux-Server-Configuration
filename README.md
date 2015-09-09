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
	
	Create user

		user add student

	Provide student root priveleges 
	
		gpasswd -a student sudo

	Setup SSH authentification 
		Generate SSH key pair on LOCAL machine !!! 
			ssh-keygen
		Place public key to the remote server 
			log into remote server as 'student': ssh student@37.143.11.88
			sudo mkdir .ssh (within 'home' directory)
			sudo touch .ssh/authorized_keys (create file for storing pub keys)
			sudo nano .ssh/authorized_keys (open file in editor and copy there my public key, save)
			chmod 700 .ssh (set permission)
			chmod 644 .ssh/authorized_keys
		All done! Log in as a student user, authorizing by ssh public key:
			ssh student@37.143.11.88 -p 22 -i [PATH_TO_PUBLIC_KEY]
		Disable password login:
			sudo nano /etc/ssh/sshd_config
			-> set 'PasswordAuthentification' to 'no'
		Restart the service
			sudo service ssh restart


	Install required Linux packages 

		sudo apt-get nano
		sudo apt-get install sudo
		sudo apt-get install man
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

		Install additional python libraris:

			sudo pip install bleach
			sudo pip install oauth2client
			sudo pip install requests
			sudo pip install httplib2
			sudo pip install flask
			sudo pip install sqlite3
			sudo pip install sqlalchemy

	Step 5: Udpate App OAuth2 settings 
		1) Update Facebook MyApp settings (https://developers.facebook.com)
			-> add http://37.143.11.88:3000/ to 'Site URL'
		2) Update App settings in Google Developer Console (https://console.developers.google.com)
			-> add http://37.143.11.88:3000/  to 'Authorized JavaScript origins'

	Step 6: Add Facebook and Google dev IDs and SECRETs into the files: 
		-> client_secrets_fb.json
		-> client_secrets_gl.json

	Step 7: Update App's IP and port in 'finalproject.py'
		app.run(host = '0.0.0.0', port = 8080)
		becomes
		app.run(host = '37.143.11.88', port = 3000)


	Step 8: Update security settings on server

		Configure firewall using 'ufw':
			sudo ufw default deny incoming
			sudo ufw default allow outgoing
			sudo ufw allow ssh
			sudo ufw allow 3000 (port used to configure App in finalproject.py)
			sudo ufw enable 

		To enable logging use:
			sudo ufw logging on


iv. A list of any third-party resources I've used of to complete this project.
---------------------------- 
	1. Initial Server Setup with Ubuntu 14.04 (https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)
	2. How To Edit the Sudoers File on Ubuntu and CentOS (https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos)
	3. Additional Recommended Steps for New Ubuntu 14.04 Servers (https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers)
	4. UFW - Uncomplicated Firewall (https://help.ubuntu.com/community/UFW)
	5. How To Deploy a Flask Application on an Ubuntu VPS (https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
#Setup django,uwsgi,nginx,PostgreSQL on ubuntu 16.04 LTS

5a.
--->> install python, pip 
-------------------------
[Note: Python was there in my machine. So make sure you have python already bcs it normally exists. If it exists only install pip]
   
	sudo apt-get update 
	sudo apt-get install python-pip
    sudo -H pip install --upgrade pip 

--->> install django , create a new django app 
----------------------
	pip install django 
	cd /var/www 
	django-admin startproject django_app0
	cd django_app0/ 
	python manage.py migrate 
	--->> Create superuser account for admin
	python manage.py createsuperuser 
		username: [enter username] 
		Email Address:[enter email address]
		password:[enter password] 
    
--->>Allow access to external hosts--Add IP- In this case IP address where Django is installed 
	
    vi django_app0/settings.py 

	ALLOWED_HOSTS = ['your-ip-address', 'localhost'] and 
	add STATIC_ROOT = os.path.join(BASE_DIR, 'static/') 
	
     python manage.py collectstatic 
--->>run django application with the following:	
	
    python manage.py runserver 0.0.0.0:8080

--->> install uwsgi
-----------------------------------------

	sudo -H pip install uwsgi	
	sudo mkdir -p /etc/uwsgi/sites 
	sudo vi /etc/uwsgi/sites/django_app0.ini 

		[uwsgi]
		project = django_app0
		uid = $USER
		base = /home/%(uid)
		chdir = %(base)/%(project)
		home = %(base)/Env/%(project)
		module = %(project).wsgi:application

		master = true
		processes = 5

		socket = /run/uwsgi/%(project).sock
		chown-socket = %(uid):www-data
		chmod-socket = 660
		vacuum = true

 --->> DONT FORGET TO USE YOUR USERNAME INSTEAD OF '$USER'
 
	sudo vi /etc/systemd/system/uwsgi.service 	
		
        [Unit]
		Description=uWSGI Emperor service
		[Service]
		ExecStartPre=/bin/bash -c 'mkdir -p /run/uwsgi; chown $USER:www-data /run/uwsgi'
		ExecStart=/usr/local/bin/uwsgi --emperor /etc/uwsgi/sites
		Restart=always
		KillSignal=SIGQUIT
		Type=notify
		NotifyAccess=all

		[Install]
		WantedBy=multi-user.target
	

----> Start the script 
	  
      sudo service uwsgi start 

--->> install nginx server
---------------------

	sudo apt update 
	sudo apt install nginx 
	sudo vi /etc/nginx/sites-available/django_app0 
	
		server {
		    listen 80;
		    server_name localhost;

		    location = /favicon.ico { access_log off; log_not_found off; }
		    location /static/ {
			root /home/pc/django_app0;
		    }

		    location / {
			include         uwsgi_params;
			uwsgi_pass      unix:/run/uwsgi/django_app0.sock;
		    }
		}

	sudo ln -s /etc/nginx/sites-available/django_app0 /etc/nginx/sites-enabled 
--->>check configuration 
    
    sudo nginx -t 
    restart nginx 	
	sudo systemctl restart nginx 
    nginx firewall allow  
	
	sudo ufw allow 'Nginx Full' 
--->>to start the service automatically when system is booting : 
	
	sudo systemctl enable nginx 
	sudo systemctl enable uwsgi 

--->>delete the default page on /etc/nginx/sites-enable/default	
	
	sudo rm /etc/nginx/sites-enabled/default 
	
	

--->> install PostgreSQL on server

	sudo apt-get update 
	sudo apt-get install postgresql postgresql-contrib 
	sudo systemctl start postgresql
	sudo systemctl enable postgresql 
	
5b.

--->> 2 apps on same server 

--->> using VirtualEnv and VirtualEnvWrapper
      install virtualenv and virtualenvwrapper. These will create virtual environment  and workflow stablity respectively   
	
    sudo -H pip install virtualenv virtualenvwrapper
	echo "export WORKON_HOME=~/Env" >> ~/.bashrc	
	echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc	
	source ~/.bashrc	 
   
--->>Follow below steps to create and run app1/app2 on the same machine.
   --->>create virtual environment use mkvirtualenv--->this will create a virtual environment, install Python and pip, and    activate the environment. 
   
 	mkvirtualenv app1
	mkvirtualenv app2
	
--->>to switch to app1/app2 virtual environment use:

    workon app2
    workon app1

--->to exit/leave the virtual environment:
	
    deactivate 

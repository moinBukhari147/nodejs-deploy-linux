# nodejs-deploy-linux
How to deploy the the node.js project on Linux in production.

*Setting up Node.js Application on Linux with PM2 and Nginx*

# Installation

- Open a terminal.
- Update your package index to ensure you have the latest version information:
    ```bash
    sudo apt update
    ```

- Install Node.js if you haven't already. You can skip this step if Node.js is already installed:
    ```bash
    sudo apt install nodejs
    ```

- If NPM is not installed then also intall the npm:
	```bash
	sudo apt install npm
	```
    

- Install PM2 globally using npm (Node Package Manager):
    ```bash
    sudo npm install -g pm2
    ```

- Verify PM2 Installation:
    ```bash
    pm2 --version
    ```

# Cloning Repository and Dependency Installation

- Clone your Git repository or pull the latest changes if you've already cloned it:
    ```bash
    git clone <repository_url>
    ```

- Install the dependencies from the package.json file by omitting the dev dependencies:
    ```bash
    npm install --omit=dev
    ```

# Enviroment setup

- Create a ecosystem.config.js file that contains the enviroment variables used in project and app starting process:
  ```bash
  	sudo nano ecosystem.config.js
  ```
  
- Add the follwoing code below and update this accoring to your requirements and setting:
  ```bash
	module.exports = {
	  apps : [{
	    name: "app-name",
	    script: './src/app.js',
	    instance: 1, 		# Number of instance not more than avaiables cores
	    exec_mode: "cluster",
	    autorestart: true,
	    watch: ".",
	    max_memory_restart: "200M",

		  env: {
		  NODE_ENV: "production",
		  PORT: 3034,
		  DATABASE_URL: 'postgresql:#username:password@localhost:5432/',  # In my case I'm using the postgres, you have set DB_name & URL accordingly
		  DATABASE_NAME: 'db_name',
		  # JWT Secret Key
		  JWT_SECRET_KEY: 'your_jwt_secrect_key_if_uisng_jwt_token_authentication'
		
		  # Email credentials
		  EMAIL_PASS: 'email_creadentials_if_uisng_email_',
		
		  # Google Credentials
		  GOOGLE_CLIENT: 'google_client_id',        # if using the google oauth
		  GOOGLE_CLIENT_SCERET: 'client_secret',    # if using the google oauth
		
		  # Firebase Credentials
		  AGORA_ADMIN_SDK: 'absolution_path_of_firebase_admin_sdk',    # if seding the notification uisng the firebase
		
		  # Facebook Credentials
		  FACEBOOK_CLIENT_ID: 'facebook_app_id',      # if using the facebook login
		  FACEBOOK_CLIENT_SECRET: 'facebook_secret',  # if using the facebook login
		
		  # Payments
		  DS_MERCHANT_MERCHANTCODE: 'merchant_uinquie_id_or_code',    # if using the payment
		  MERCHANT_SECRET_KEY: 'merchant_secrect_key',                # if using the payment
		
		  # ADD or update the enviroments varables names according to the needs
	        }
	  }
	]
	};
    ```

# Database installation
Before stating the node.js server using the pm2 first of all we have to install and configure the database.
 - if the database is already installed skip this step otherwise:
   ```bash
		sudo apt update
	sudo apt install postgresql postgresql-contrib
	sudo service postgresql start      # start the service
	sudo systemctl enable potgresql    # restart autmatically when the system restarts
		sudo service postgresql status     # check the status of postgresql running or not
   ```
- To check which version of postgresql is installed:
  ```
	psql --version
  ```
- Postgres user is by default is created on installing the PostgreSQL. Update the password of the postgres using commands below:

  ```bash
	sudo -u postgres psql    # swtich ot postgres user to access PostgreSQL
  	ALTER USER postgres  WITH PASSWORD 'new_password';
  	\q    # quit
  ```

- By default PostgreSQL comes with default databse postgres. If you wanted to create the databse then:
```bash
	sudo -u postgres psql
	CREATE DTABASE database_name;]
	\c your_database     # connect to you databse to peform any action using the psql shell like creating table ect.
	\q
```

- if dealing with point data and need to install the postgis extention the follow this:

  ```bash
	# first find the version of postgresql install the command is given above.
	sudo apt install postgis postgresql-<version_of_postgres>-postgis-3
  	sudo -u postgres psql
  	\c your_database   # switch to the desired db
  	CREATE EXTENSION postgis;
  	sudo systemctl restart postgresql  # to restart the postgresSQL after updates.
  	\q
  ```

  You can follow the all the steps of or the desired steps accoridng to you need. Now the databse base has beed configured we are reaady to start the process.
  
# Starting Node.js Application with PM2

- Start the nodejs application using the command given below

```bash
pm2 start ecosystem.config.js --env production     # rename the .js to .cjs if you using the type: module i.e (import export method)
```


## Applying PM2 Changes

- After making changes to your PM2 process configuration, save the changes and reload the PM2 process to apply them:

```bash
pm2 save
pm2 reload my-node-app
```

# Managing Firewall Rules with UFW

The Uncomplicated Firewall (UFW) is a user-friendly interface for managing firewall rules on Linux systems.

## Installation

- If UFW is not installed on your system, you can install it using the following command:

```bash
sudo apt install ufw
sudo ufw status
```

- Allow Incoming Traffic to Specific Port
```bash
sudo ufw allow 80     # allow all http trafic
sudo ufw allow 443    # allow all https trafic
```

- Once you have allowed the necessary ports, enable UFW to start enforcing the rules
```bash
sudo ufw enable
```

- You can check the status of UFW to ensure that it's active and allowing traffic to the specified ports
```bash
sudo ufw status
```

# NginX Configuration

## Installation

```bash
	sudo apt update
	sudo apt install nginx
	sudo systemctl start nginx
	sudo systemctl enable nginx
	sudo systemctl status nfinx
 ```
Check that nginx running usig the browser by entering the your ip in the browser and remove the s from https means use http:#ip

 ## Configure backend with reverse proxy nginx
 - Open the default file created by the nginx using the follwoing command:

   ```bash
	sudo nano /etc/nginx/sites-available/default
   ```
   
- Replace with the following code:
 
```bash
server {
    listen 80 ;
    server_name default_server;
    # Any ohter your nginx configuration 

    location / {
	proxy_pass http:#127.0.0.1:YOUR_APP_PORT;  		# Proxy requests to the backend server running on localhost:YOUR_APP_PORT
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
     	proxy_set_header Host $host;  			# Set the Host header to the client's original host
     	proxy_set_header X-Real-IP $remote_addr;  	# Set the X-Real-IP header to the client's IP address
     	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  	# Append client's IP addresses to X-Forwarded-For header
     	proxy_set_header X-Forwarded-Proto $scheme;  			# Set the X-Forwarded-Proto header to the client's protocol (http or https)
        proxy_cache_bypass $http_upgrade;
    }
}
```

- Test and relad nginx:
  ```bash
	sudo nginx -t    		# this check is there any synaxt error in the nginx configuration
	sudo systemctl reload nginx     # this relaods the nginx to refelct the changes
  ```

## Setup SSl and shifting to https
We can use the configuration done in the default file in "/etc/nginx/sites-enabled/" that we have done above
Or we can create the new file with the domain name and reconfigure
### if you wanted to use the default then skip these steps otherwise:
- create the file yourdomain.com in drectory "/etc/nginx/sites-available/" using the command below:
  ```bash
	sudo nano /etc/nginx/sites-available/yourdomain.com
  ``` 
- Add the follwoing the code that we have added in the default in this file:
  ```bash
	server {
    listen 80 ;
    server_name yourdomain.com;
    # Any ohter your nginx configuration 

    location / {
	proxy_pass http:#127.0.0.1:YOUR_APP_PORT;  		# Proxy requests to the backend server running on localhost:YOUR_APP_PORT
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
     	proxy_set_header Host $host;  			# Set the Host header to the client's original host
     	proxy_set_header X-Real-IP $remote_addr;  	# Set the X-Real-IP header to the client's IP address
     	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  	# Append client's IP addresses to X-Forwarded-For header
     	proxy_set_header X-Forwarded-Proto $scheme;  			# Set the X-Forwarded-Proto header to the client's protocol (http or https)
        proxy_cache_bypass $http_upgrade;
    	}
   }
  ```
Make sure to replace yourdomain.com with your actual domain name and YOUR_APP_PORT

- Enable the configuration for yourdomain.com:
  ```bash
	sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
  ```
- Disable the configuration for default:
  ```bash
  	sudo rm /etc/nginx/sites-enabled/default
  ```
- Verify nginx configuation and reload:
  ```bash
	sudo nginx -t
  	sudo systemctl reload nginx
  ```
  
- Install Certbot and Configure SSL
Certbot is used to obtain free SSL certificates from Let’s Encrypt.
```bash
	sudo apt install certbot python3-certbot-nginx
```
- Obtain SSL Certificates for Your Domain:
  ```bash
	sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
  ```
  -  Auto-Renew SSL Certificates:
  ```bash
	 Auto-Renew SSL Certificates
  	sudo certbot renew --dry-run
  ```

# Helping commands
- Display disk usage and provides output in human-reacable format
```
df -h
```

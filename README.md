# nodejs-deploy-linux
How to deploy the the node.js project on Linux in production.

*Setting up Node.js Application on Linux with PM2 and Nginx*

# Installation

- Open a terminal.
- Update your package index to ensure you have the latest version information:
    ```bash
    sudo apt-get update
    ```

- Install Node.js if you haven't already. You can skip this step if Node.js is already installed:
    ```bash
    sudo apt-get install nodejs
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

# Starting Node.js Application with PM2

- Use PM2 to start your Node.js application in cluster mode with a specified number of instances. For example, to start a single cluster with four instances and on port 4090:

```bash
pm2 start app.js --name my-node-app --instances 4 --env PORT=4090
```

### --name

- Description: Set the running app name that shows in the pm2 list.
- Usage: `--name <app_name>`

### --instances

- Description: Set the number of instances for the cluster.
- Usage: `--instances <number_of_instances>`

### --env PORT

- Description: Run your Node.js application on the specified port.
- Usage: `--env PORT=<port_number>`

**Check and verify that you have specified the same PORT in `server.js`.**

## Applying PM2 Changes

- After making changes to your PM2 process configuration, save the changes and reload the PM2 process to apply them:

```bash
pm2 save
pm2 reload my-node-app
```
- You can check the status of your PM2 processes to ensure that your Node.js application is running in cluster mode on port 4090:
```bash
- pm2 status
```

# Managing Firewall Rules with UFW

The Uncomplicated Firewall (UFW) is a user-friendly interface for managing firewall rules on Linux systems.

## Installation

- If UFW is not installed on your system, you can install it using the following command:

```bash
sudo apt-get install ufw
```

- Allow Incoming Traffic to Specific Port
```bash
sudo ufw allow 4090/tcp
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
```bash
server {
    listen 80;
    server_name example.com;
    your nginx configuration;

    location / {
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://your-ip-address:4090;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

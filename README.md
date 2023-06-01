# Reverse-proxy-using-Nginx
Step by step guides to implement reverse proxy on website using NGINX.

Nginx deployed and running on a GCP VM on port 80.

Nginx can be deployed as a docker container or on the OS itself, it does not matter.

Configure the Nginx to reverse proxy traffic to a backend server for 2 specific URIs.

The backend server is running HTTPS. The nginx reverse proxy needs to manage the TLS traffic between itself and the backend server to ensure there are no SSL errors.

Also, we need to ensure that the URL of the existing website is maintained in the browser when the nginx proxy is communicating to the backend server. (We don’t need a simple redirect)
Nginx to proxy all other traffic to a local node JS application. (All URI that are not reverse proxied over)

Step 1 – Configuring Node.Js PPA

Node.js releases are available in two types, one is the LTS release, and the other is the current release. Choose any one version of your choice or as per the project requirements. Let’s add the PPA to your system to install Nodejs on Ubuntu.

Use Current Release – During the last update of this tutorial, Node.js 19 is the current Node.js version available

sudo apt install -y curl

curl -sL https://deb.nodesource.com/setup_19.x | sudo -E bash – 

Use LTS Release – Instead of the current release, it’s a good idea to use a stable version. During the last update of this tutorial, Node.js 18 is the latest LTS release available.

sudo apt install -y curl 

curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash - 

Step 2 – Install Node.js on Ubuntu

You have successfully configured Node.js PPA in your Ubuntu system. Now execute the below command to install Node on Ubuntu using apt-get. This will also install NPM with node.js. This command also installs many other dependent packages on your system

sudo apt install -y nodejs

Step 3 – Check Node.js and NPM Version

After installing node.js verify and check the installed version

node -v or npm -v

Step 4 – Create Demo Web Server (Optional)

This is an optional step. Suppose you want to test your node.js install. Let’s create a web server with “Hello World!” text. Create a file server.j

sudo nano server.js

server.js 

var http = require('http');

http.createServer(function (req, res) {

res.writeHead(200, {'Content-Type': 'text/plain'});

res.end('Hello World\n');

}).listen(3000, "127.0.0.1");

console.log('Server running at http://127.0.0.1:3000/');

Save the file and close. Then start the Node application using the following command.

node server.js 

debugger listening on port 5858

Server running at http://127.0.0.1:3000/

You can also start the application with debugging enabled with the following commands.

node --inspect server.js

Debugger listening on ws://127.0.0.1:9229/938cf97a-a9e6-4672-922d-a22479ce4e29

For help, see: https://nodejs.org/en/docs/inspector

Server running at http://127.0.0.1:3000/

The web server has been started on port 3000. Now access http://127.0.0.1:3000/ URL in browser.


Step 1: Install Nginx

The first step is to install Nginx on your server. This can typically be done using your operating system’s package manager (e.g. apt-get on Debian-based systems, dnf on Red Hat-based systems).

Sudo apt update && sudo apt install nginx

Sudo dnf install nginx.

Step 2: Configure the Backend Application

A backend application should be listening on some other port. For example, I have created a sample node.js application that serves incoming requests using the Node express module. This application is listening on localhost and port 3000

node server.js

Output

debugger listening on port 5858

Server running at http://127.0.0.1:3000/

Step 3: Configure the Nginx Server Block
Nginx uses server blocks to configure individual websites. We need to create a new server block configuration file for our reverse proxy.
sudo nano /etc/nginx/conf.d/reverse-proxy.conf

Add the following configuration to the server block configuration file:

	server {
    listen 80;
    server_name example.com;
 
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

listen 80; defines the port on which Nginx will listen for incoming connections.

server_name example.com; is the domain name that will be used to access the reverse proxy.

proxy_pass http://backend-server; tells Nginx to forward the incoming requests to the specified backend server.

proxy_set_header Host $host; and proxy_set_header X-Real-IP $remote_addr; and proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; are used to set headers that the backend server can use to identify the original client.

Step 4: Restart Nginx

Before restarting the Nginx service, test the configuration files using the following command:

sudo nginx -t 

If the configuration test is successful, restart Nginx to apply the changes:

sudo systemctl restart nginx 


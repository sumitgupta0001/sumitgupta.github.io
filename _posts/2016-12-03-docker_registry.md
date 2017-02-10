---
layout: post
title:  "Docker Registry"
date:   2016-12-03 02:59:59
categories: docker_registry
---

Hello friends,
Today , we will see how Docker Registry works.

<h2>Introduction</h2>

Docker is a great tool for deploying your servers. Docker even has a public registry called Docker Hub to store Docker images. While Docker lets you upload your Docker creations to their Docker Hub for free, anything you upload is also public. This might not be the best option for your project.

This guide will show you how to set up and secure your own private Docker registry. By the end of this tutorial you will be able to push a custom Docker image to your private registry and pull the image securely from a different host.

To know the basics of Docker registry follow my blog on
<a href="https://sumitgupta0001.github.io/xyz/2016/11/30/docker.html">Docker Basics .</a>

<h2>Prerequisites</h2>

To complete this , we will need the following:

2 Ubuntu Droplets: one for the private Docker registry and one for the Docker client

A non-root user with sudo privileges on each Droplet 

Docker and Docker Compose installed on both the servers

<h2>Installing Package for Added Security</h2>

To set up security for the Docker Registry it's best to use Docker Compose. This way we can easily run the Docker Registry in one container and let Nginx handle security and communication with the outside world in another. You should already have it installed from the Prerequisites section.

Since we'll be using Nginx to handle our security, we'll also need a place to store the list of username and password combinations that we want to access our registry. We'll install the apache2-utils package which contains the htpasswd utility that can easily generate password hashes Nginx can understand:

<code>sudo apt-get -y install apache2-utils</code>

<h2>Installing and Configuring the Docker Registry</h2>

Since the Docker registry itself is an application with multiple components, we'll use Docker Compose to manage our configuration.

To start a basic registry the only configuration needed is to define the location where your registry will be storing its data. Let's set up a basic Docker Compose YAML file to bring up a basic instance of the registry.

First create a folder where our files for this tutorial will live and some of the subfolders we'll need:

<code>mkdir ~/docker-registry && cd $_</code>
<code>mkdir data </code>

Next , to fix security issues ,  set up a copy of Nginx inside another Docker container and link it up to our Docker registry container.

Let's start by creating a directory to store our Nginx configuration:

<code>mkdir ~/docker-registry/nginx</code>

Now ,Using your favorite text editor, create a docker-compose.yml file:

<code>nano docker-compose.yml</code>

Add the following contents to the file:

{% highlight ruby%}
registry:
  image: registry:2
  ports:
    - 127.0.0.1:5000:5000
  environment:
    REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
  volumes:
    - ./data:/data
nginx:
  image: "nginx:1.9"
  ports:
    - 5043:443
  links:
    - registry:registry
  volumes:
    - ./nginx/:/etc/nginx/conf.d:ro
{% endhighlight%}

Great! At this point you've already got a full Docker registry up and running and listening on port <code>5000</code>. The interesting bit here is the <code>links</code> section. It automagically set up a "link" from one Docker container to the another. When the Nginx container starts up, it will be able to reach the <code>registry</code> container at the hostname <code>registry</code> no matter what the actual IP address the registry container ends up having. (Behind the scenes Docker is actually inserting an entry into the <code>/etc/hosts</code> file in the <code>nginx</code> container to tell it the IP of the <code>registry</code> container).

The <code>volumes:</code> section is similar to what we did for the <code>registry</code> container. In this case it gives us a way to store the config files we'll use for Nginx on our host machine instead of inside the Docker container. The <code>:ro</code> at the end just tells Docker that the Nginx container should only have read-only access to the host filesystem. 

Running docker-compose up will now start two containers at the same time: one for the Docker registry and one for Nginx.

We need to configure Nginx before this will work though, so let's create a new Nginx configuration file.

Create a registry.conf file:

nano ~/docker-registry/nginx/registry.conf


Copy the following into the file:

{% highlight ruby %}

upstream docker-registry {
  server registry:5000;
}

server {
  listen 443;
  server_name myregistrydomain.com;

  # SSL
  # ssl on;
  # ssl_certificate /etc/nginx/conf.d/domain.crt;
  # ssl_certificate_key /etc/nginx/conf.d/domain.key;

  # disable any limits to avoid HTTP 413 for large image uploads
  client_max_body_size 0;

  # required to avoid HTTP 411: see Issue #1486 (https://github.com/docker/docker/issues/1486)
  chunked_transfer_encoding on;

  location /v2/ {
    # Do not allow connections from docker 1.5 and earlier
    # docker pre-1.6.0 did not properly set the user agent on ping, catch "Go *" user agents
    if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
      return 404;
    }

    # To add basic authentication to v2 use auth_basic setting plus add_header
    # auth_basic "registry.localhost";
    # auth_basic_user_file /etc/nginx/conf.d/registry.password;
    # add_header 'Docker-Distribution-Api-Version' 'registry/2.0' always;

    proxy_pass                          http://docker-registry;
    proxy_set_header  Host              $http_host;   # required for docker client's sake
    proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
    proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header  X-Forwarded-Proto $scheme;
    proxy_read_timeout                  900;
  }
}

{% endhighlight %}

If at this stage you run docker-compose up and pass the curl command like this:
 
<code>curl http://localhost:5043/v2/</code>

then you will see the output as:

<code>{}</code>

If things are working correctly you'll see some output in your docker-compose terminal that looks like the below as well:

{% highlight ruby %}

registry_1 | time="2015-08-11T10:24:53.746529894Z" level=debug msg="authorizing request" environment=development http.request.host="localhost:5043" http.request.id=55c3e2a6-4f34-4b0b-bc57-11c814b4f4d3 http.request.method=GET http.request.remoteaddr=172.17.42.1 http.request.uri="/v2/" http.request.useragent="curl/7.35.0" instance.id=55634dfc-c9e0-4ec9-9872-6f4930c17759 service=registry version=v2.0.1
    registry_1 | time="2015-08-11T10:24:53.747650205Z" level=info msg="response completed" environment=development http.request.host="localhost:5043" http.request.id=55c3e2a6-4f34-4b0b-bc57-11c814b4f4d3 http.request.method=GET http.request.remoteaddr=172.17.42.1 http.request.uri="/v2/" http.request.useragent="curl/7.35.0" http.response.contenttype="application/json; charset=utf-8" http.response.duration=8.143193ms http.response.status=200 http.response.written=2 instance.id=55634dfc-c9e0-4ec9-9872-6f4930c17759 service=registry version=v2.0.1
    registry_1 | 172.17.0.21 - - [11/Aug/2015:10:24:53 +0000] "GET /v2/ HTTP/1.0" 200 2 "" "curl/7.35.0"
    nginx_1    | 172.17.42.1 - - [11/Aug/2015:10:24:53 +0000] "GET /v2/ HTTP/1.1" 200 2 "-" "curl/7.35.0" "-"

{% endhighlight %}

<h2>Setting Up Authentication</h2>

Now that Nginx is proxying requests properly let's set it up with HTTP authentication so that we can control who has access to our Docker registry. To do that we'll create an authentication file in Apache format (Nginx can read it too) via the htpasswd utility we installed earlier and add users to it.

Create the first user as follows, replacing USERNAME with the username you want to use:

<code>cd ~/docker-registry/nginx</code>

<code>htpasswd -c registry.password USERNAME</code>

Create a new password for this user when prompted.

Next, we need to tell Nginx to use that authentication file.

Open up ~/docker-registry/nginx/registry.conf in your favorite text editor:

<code>nano ~/docker-registry/nginx/registry.conf</code>

Uncomment the two lines that start with auth_basic as well as the line that starts with add_header by removing the # character at the beginning
of the lines.

We've now told Nginx to enable HTTP basic authentication for all requests that get proxied to the Docker registry and told it to use the password file we just created.

Now , our curl command will work with:

<code>curl http://USERNAME:PASSWORD@localhost:5043/v2/</code>

<h2>Setting Up SSL</h2>

At this point we have the registry up and running behind Nginx with HTTP basic authentication working. However, the setup is still not very secure since the connections are unencrypted. You might have noticed the commented-out SSL lines in the Nginx config file we made earlier.

Let's enable them. First, open the Nginx configuration file for editing:

Uncomment the following SSL lines:

  # ssl on;
  # ssl_certificate /etc/nginx/conf.d/domain.crt;
  # ssl_certificate_key /etc/nginx/conf.d/domain.key;

Save the file. Nginx is now configured to use SSL and will look for the SSL certificate and key files at /etc/nginx/conf.d/domain.crt and /etc/nginx/conf.d/domain.key respectively. Due to the mappings we set up earlier in our docker-compose.yml file the /etc/nginx/conf.d/ path in the Nginx container corresponds to the folder ~/docker-registry/nginx/ on our host machine, so we'll put our certificate files there.

<h2>Signing Your Own Certificate</h2>

Since Docker currently doesn't allow you to use self-signed SSL certificates this is a bit more complicated than usual — we'll also have to set up our system to act as our own certificate signing authority.

To begin, let's change to our ~/docker-registry/nginx folder and get ready to create the certificates:

<code>cd ~/docker-registry/nginx</code>

Generate a new root key:

<code>openssl genrsa -out devdockerCA.key 2048</code>

Generate a root certificate (enter whatever you'd like at the prompts):

<code>openssl req -x509 -new -nodes -key devdockerCA.key -days 10000 -out devdockerCA.crt</code>

Then generate a key for your server (this is the file referenced by ssl_certificate_key in our Nginx configuration):

<code>openssl genrsa -out domain.key 2048</code>

Now we have to make a certificate signing request.

<code>Note --></code> If you dont have your domain name then edit your /etc/hosts file, 
make a entry with (server-name) (server-ip). Eg. datacapture 192.168.X.X 

After you type this command, OpenSSL will prompt you to answer a few questions. Write whatever you'd like for the first few, but when OpenSSL prompts you to enter the <code>"Common Name" make sure to type in the domain name or your server name(mentioned above).</code>

<code>openssl req -new -key domain.key -out dev-docker-registry.com.csr</code>

For example, if your Docker registry is going to be running on the domain www.ilovedocker.com or (server-name), then your input should look like this:

{% highlight ruby%}
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:www.ilovedocker.com or (server-name)
Email Address []:
{% endhighlight %}

Next, we need to sign the certificate request:

<code>openssl x509 -req -in dev-docker-registry.com.csr -CA devdockerCA.crt -CAkey devdockerCA.key -CAcreateserial -out domain.crt -days 10000</code>

Since the certificates we just generated aren't verified by any known certificate authority (e.g., VeriSign), we need to tell any clients that are going to be using this Docker registry that this is a legitimate certificate. Let's do this locally on the host machine so that we can use Docker from the Docker registry server itself:

<code>sudo mkdir /usr/local/share/ca-certificates/docker-dev-cert</code>

<code>sudo cp devdockerCA.crt /usr/local/share/ca-certificates/docker-dev-cert</code>

<code>sudo update-ca-certificates</code>

Restart the Docker daemon so that it picks up the changes to our certificate store:

<code>sudo service docker restart</code>

<code>Warning: </code> You'll have to repeat this step for every machine that connects to this Docker registry! 

<h2>Starting Docker Registry as a Service</h2>

First let's remove any existing containers, move our Docker registry to a system-wide location and change its permissions to root:


<code>cd ~/docker-registry</code>
<code>docker-compose rm</code>   # this removes the existing containers 
<code>sudo mv ~/docker-registry /docker-registry</code>
<code>sudo chown -R root: /docker-registry</code>


Then use your favorite text editor to create an Upstart script:

<code>sudo nano /etc/init/docker-registry.conf</code>

Add the following contents to create the Upstart script :

{% highlight ruby%}
/etc/init/docker-registry.conf
description "Docker Registry"

start on runlevel [2345]
stop on runlevel [016]

respawn
respawn limit 10 5

chdir /docker-registry

exec /usr/local/bin/docker-compose up

{% endhighlight %}

Let's test our new Upstart script by running:

<code>sudo service docker-registry start</code>

You can verify that the server is running by executing:

<code>docker ps</code>

Upstart will log the output of the docker-compose command to /var/log/upstart/docker-registry.log.

<h2> Accessing Your Docker Registry from a Client Machine </h2>

To access your Docker registry from another machine, first add the SSL certificate you created earlier to the new client machine. The file you want is located at ~/docker-registry/nginx/devdockerCA.crt.

On the <code>registry server</code>, view the certificate:

<code>sudo cat /docker-registry/nginx/devdockerCA.crt</code>

On the <code>client machine</code>, create the certificate directory:

<code>sudo mkdir /usr/local/share/ca-certificates/docker-dev-cert</code>

Open the certificate file for editing:

<code>sudo nano /usr/local/share/ca-certificates/docker-dev-cert/devdockerCA.crt</code>

Paste the certificate contents.

Now update the certificates:

<code>sudo update-ca-certificates</code>

Now edit /etc/hosts file and make a entry as we did above:
(Registry server name) (Registry server IP)
E.g--> datacapture 192.168.X.X 

Restart Docker to make sure it reloads the system's CA certificates.

You should now be able to log in to your Docker registry from the client machine:

<code>docker login (Registry server name):5043</code>

<code>Note --></code>
Dont confuse with Registry server name , its same Registery server name that we used on Registry server, and used that same name on certification .

Voila, Login Succeeded.

At this point your Docker registry is up and running! Let's make a test image to push to the registry.

<h2>Publish to Your Private Docker Registry</h2>

We have seen earlier how to create a image, in Docker Basics, now suppose if you have a image with name elasticsearch, then tag the image as:

<code>docker tag elasticsearch (Registry server name):5043/elasticsearch</code>

After image tagging , push the image as :

<code>docker push (Registry server name):5043/elasticsearch</code>

So , finaly your first image is pushed to your private registry. 
Now, you can pull the same image from any server , by using the pull command.

<code>docker pull (Registry server name):5043/elasticsearch</code>

Thats all, for the Docker Registry.
Hope, you had fun. 
Happy coding :)
    


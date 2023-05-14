+++
title = "Journey of a Http Request From Cloudlare to Your Controller"
date = "2023-04-03T23:57:33+05:30"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["web", "backend"]
keywords = ["http", "web servers", "middlewares"]
description = "This article discusses about how production servers sit on the cloud and serve http requests"
showFullContent = false
readingTime = true
hideComments = false
color = "red" #color from the theme settings
Toc = true
+++

This is an oversimplified overview of how applications are deployed on production environments.

### What is a web server?

A web server only responds to http requests. Examples of web server include nginx, gunicorn, apache, Microsoft IIS. Web servers connect with application servers with using variety of protocols over tcp ports.
Python, flask and fastapi use wsgi protocol. Rails is built upon rack to serve application requests.
Web servers are more secure than application server. It even caches static data.
A web server can also act like a load balancer and reverse proxy. We discuss about proxies in next section. Web servers are very light on cpu and ram. Web server are very efficient and run single threadedly, they have built in ddos protection too 

### What is an Application Server or The Gateway?

An Application server sits between the web server and the application code, it acts as a Gateway for the application. Its job is to translate incoming requests (HTTP, FastCGI, uWSGI and more) to application code. It also performs the following operations
* Manage (spawn and monitor) processes and/or threads of the application
* Load balance requests between processes
* Perform reporting and logging

Python Web Applications (WSGI) commonly use uWSGI and Gunicorn as their application server.
Puma, Unicorn, Phusion Passenger are commonly used Ruby (Rack based) Applications.

### What is a Proxy server?

A proxy server is a server that sits between an server and a client. It relays client http requests to the server. Proxy servers are of two types - forward proxy and reverse proxy.

The forward proxy is like a VPN, it makes requests to the server on behalf of the client but it doesn't encrypt traffic like VPN. It can look at the content of the request and inspect it to determine if it should forward the request.

Reverse proxy masks the server from

### The journey

- #### Cloudflare

When a client requests a resources at my-production-app.in/api/products. The DNS resolves the application my-production-app.in to the ip provided by cloudflare. We configured our domain registrar - Godaddy to let users know that our app my-production-app.in sits at ip provided by cloudflare.

Cloudflare provides us with ssl certificates to enable communication over https. It acts like a forward proxy - it relays all the requests to our web server or a reverse proxy. It attaches some additional headers to our http request. Such as the header XClient_IP. It also provides additional security by masking the real IP of our web server.

- #### Load Balancer

A Load Balancer is a reverse proxy server, it redirects the incoming http requests to a number of web servers. In case of AWS these web servers could be EC2 instances or lambdas.  It ensures that no one server is overloaded, if any server goes offline it redirects request to remaining online servers, thus it prevents downtime.
It enables us to easily manage capacity by adding or removing servers according to needs.

- #### The Web server

It typically takes a few milliseconds for our http request to reach the web server. Our web server translates the http request from binary format in tcp to a protocol that our application server can understand. 
Its job is to regulate traffic and control bandwidth usage to prevent overloading of the application server.

- #### The Application Server

Our application server listening on the tcp port will receive a request and hands it over to separate process running our application code. The request goes through some middlewares in order and finally to our controller. Each middleware receives the http request object and passes it to the next middleware with or without modification.

Settings.py in django applications lists your middlewares. In Rails you can list the middlewares by running `RAILS_ENV=production bin/rails middlewares`.
Notice that rails router is the last middleware which your http request goes through. It then reaches your controller.

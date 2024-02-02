+++
title = "Journey of a Http Request From Cloudlare to Your Controller"
date = "2023-04-03T23:57:33+05:30"
author = ""
authorTwitter = "" #do not include @
cover = "journey.jpg"
tags = ["web", "backend"]
keywords = ["http", "web servers", "middlewares"]
description = "My understanding of how production servers sit on the cloud and serve http requests"
showFullContent = false
readingTime = true
hideComments = false
color = "orange" #color from the theme settings
Toc = true
+++

This is an oversimplified overview of how applications are deployed on production environments.

### What is a web server?

A web server only responds to HTTP requests. Examples of web servers include Nginx, unicorn, apache, and Microsoft IIS. Web servers connect with application servers using a variety of protocols over TCP ports.
Python, Flask and Fastapi use WSGI protocol. Rails is built upon the Rack to serve application requests.
A Web server is more secure than an application server. It even caches static data.
A web server can also act like a load balancer and reverse proxy. We discuss proxies in the next section. Web servers are very light on CPU and RAM. Web servers are very efficient and run single threadedly, they have built-in DDoS protection too.

### What is an Application Server or The Gateway?

An Application server sits between the web server and the application code, it acts as a Gateway for the application. Its job is to translate incoming requests (HTTP, FastCGI, uWSGI and more) to application code. It also performs the following operations
* Manage (spawn and monitor) processes and/or threads of the application
* Load balance requests between processes
* Perform reporting and logging

Python Web Applications (WSGI) commonly use uWSGI and Gunicorn as their application server.
Puma, Unicorn, Phusion Passenger are commonly used Ruby (Rack based) Applications.

### What is a Proxy server?

A proxy server is a server that sits between a server and a client. It relays client HTTP requests to the server. Proxy servers are of two types - forward proxy and reverse proxy.

The forward proxy is like a VPN, it makes requests to the server on behalf of the client but it doesn't encrypt traffic like a VPN. It can look at the content of the request and inspect it to determine if it should forward the request.

A reverse proxy sits in front of the origin server, unlike a forward proxy which sits in front of the client. It ensures that the origin server never communicates with a client directly. It is used to Load Balance primarily but it has other benefits too like - DDoS protection and caching.

### The journey

- #### Cloudflare

When a client requests a resource at my-production-app.in/api/products. The DNS resolves the application my-production-app.in to the IP provided by Cloudflare. We configured our domain registrar - Godaddy to let users know that our app my-production-app.in sits at ip provided by Cloudflare.

Cloudflare provides us with SSL certificates to enable communication over HTTP. It acts like a forward proxy - it relays all the requests to our web server or a reverse proxy. It attaches some additional headers to our HTTP request. Such as the header XClient_IP. It also provides additional security by masking the real IP of our web server.

- #### Load Balancer

A Load Balancer is a reverse proxy server, it redirects the incoming HTTP requests to several web servers. In the case of AWS, these web servers could be EC2 instances or lambdas.  It ensures that no one server is overloaded, if any server goes offline it redirects requests to remaining online servers, thus it prevents downtime.
It enables us to easily manage capacity by adding or removing servers according to needs.

- #### The Web server

It typically takes a few milliseconds for our HTTP request to reach the web server. Our web server translates the HTTP request from the binary format in TCP to a protocol that our application server can understand. 
Its job is to regulate traffic and control bandwidth usage to prevent overloading of the application server.

- #### The Application Server

Our application server listening on the TCP port will receive a request and hand it over to a separate process running our application code. The request goes through some middlewares in order and finally to our controller. Each middleware receives the HTTP request object and passes it to the next middleware with or without modification.

Settings.py in a Django application lists your middlewares. In Rails you can list the middlewares by running `RAILS_ENV=production bin/rails middlewares`.
Notice that the rails router is the last middleware that your HTTP request goes through. It then reaches your controller.

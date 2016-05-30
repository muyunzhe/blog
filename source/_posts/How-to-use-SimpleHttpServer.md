---
title: How to use SimpleHttpServer
comments: true
categories: technology
tags: python
date: 2016-05-24 18:26:40
---
In this post we will look at the built-in web server in Python.
<!--more-->
# What is it?
The SimpleHTTPServer module that comes with Python is a simple HTTP server that
provides standard GET and HEAD request handlers.

# Why should I use it?
An advantage with the built-in HTTP server is that you don't have to install
and configure anything. The only thing that you need, is to have Python installed.

That makes it perfect to use when you need a quick web server running and you
don't want to mess with setting up apache.

You can use this to turn any directory in your system into your web server
directory.

# How do I use it?
To start a HTTP server on port 8000 (which is the default port), simple type:
* python -m SimpleHTTPServer [port]

This will now show the files and directories which are in the current working
directory.
You can also change the port to something else:
`$ python -m SimpleHTTPServer 8080`

# How to share files and directories
1.in your terminal, cd into whichever directory you wish to have accessible via
browsers and HTTP.
  ```
  cd /var/www/
  $ python -m SimpleHTTPServer
  ```
After you hit enter, you should see the following message:
Serving HTTP on 0.0.0.0 port 8000 ...

2.Open your favorite browser and put in any of the following addresses:
```
http://your_ip_address:8000
http://127.0.0.1:8000
```
If you don't have an index.html file in the directory, then all files and
directories will be listed.

3.As long as the HTTP server is running, the terminal will update as data are
loaded from the Python web server.
You should see standard http logging information (GET and PUSH), 404 errors,
IP addresses, dates, times, and all that you would expect from a standard http
log as if you were tailing an apache access log file.

# Summary
In this post we showed how you with minimal effort can setup a web server to
serve content.

It's a great way of serve the contents of the current directory from the command
line

While there are many web server software out there (apache, nginx), using Python
built-in HTTP server require no installation and configuration.

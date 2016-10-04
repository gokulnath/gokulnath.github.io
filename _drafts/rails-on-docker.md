---
layout: post
title: Rails on Docker
comments: true
---



At work, I write programs in Ruby and for web development I use Rails. Deployment is a part of the work I do and setting up infrastructure is a task which all prefers to avoid.
We used to have [Chef](https://www.chef.io/) cookbooks, but as the company grew and split into many semi-independent teams it became impossible to maintain the Chef recipes and they became a forgotten past.

The team I work for is trying to use Docker for deployment. After some research, I found out about [passenger-docker](https://github.com/phusion/passenger-docker), the docker image maintained by [Phusion](https://www.phusionpassenger.com/) for rails deployments. I have been using Nginx with Passenger as a web server for serving rack based applications for years, so choosing a docker image maintained by the people behind Passenger was a no-brainer to me.

## Advantages offered by Docker
Containerization solutions like Docker solve one of the most tedious of tasks, the orchestration of infrastructure. With the help of docker, it is now possible to orchestrate just the docker environment, instead of managing installations of Nginx, Mysql, Redis, PHP, Apache, etc. Of course, you need to know how to use all those tools, but you do not need to install them individually and maintain them. So ideally, you can have a machine with just docker installed and all applications will run inside docker containers.

## Docker image configuration
Anything you want to do on a docker image for running the application is done via Dockerfile. This is especially easy to configure as the format of the file is `INSTRUCTION arguments` and there are only a limited number of instructions. The Dockerfile I came up with at last is below.

{% highlight ruby %}
FROM phusion/passenger-full:0.9.19

# Set correct environment variables.
ENV HOME /root
ENV APP_HOME /home/app/webapp

# Use baseimage-docker's init process.
CMD ["/sbin/my_init"]

# use port 80
EXPOSE 80

# Should start nginx.
RUN rm -rf /etc/service/nginx/down

# Nginx configs
RUN rm /etc/nginx/sites-enabled/default
ADD docker_config/webapp.conf /etc/nginx/sites-enabled/webapp.conf

# App installation
RUN mkdir $APP_HOME
WORKDIR $APP_HOME
ADD . $APP_HOME
RUN bundle install --deployment --local
RUN chown -R app:app $APP_HOME

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
{% endhighlight %}

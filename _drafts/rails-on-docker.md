---
layout: post
title: Rails on Docker
comments: true
---



At work, I write programs in Ruby and for web development I use Rails. Deployment is a part of the work I do and setting up infrastructure is a task which all prefers to avoid.
We used to have [Chef](https://www.chef.io/) cookbooks, but as the company grew and split into many semi-independent teams it became impossible to maintain the Chef recipes and they became a forgotten past.

The team I work for is trying to use Docker for deployment. After some research, I found out about [passenger-docker](https://github.com/phusion/passenger-docker), the docker image maintained by [Phusion](https://www.phusionpassenger.com/) for rails deployments. I have been using Nginx with Passenger as a web server for serving rack based applications for years, so choosing a docker image maintained by the people behind Passenger was a no-brainer.

## My workflow

I believe that I need to containarize only the application environment. My infrastructure usually consists of a load balancer, an application server and a database server. Dockerising it means that I install the load balancer (I use nginx itself) and database in their respective machins and install docker in the application server.

That means that only the web application runs in docker containers, configured with volums for persistent data.

### Steps for a docker instance deployments.
1. Commit source and tag it in git with a version.
2. Build docker for my private docker registry with the version used for git tagging.
3. Push to my private docker registry.
4. At last, change version in my production docker-compose.yml file and run it.

#### Building docker images in developement system.

Anything you want to do on a docker image for running the application is done via Dockerfile. This is especially easy to configure as the format of the file is `INSTRUCTION arguments` and there are only a limited number of instructions. The Dockerfile I came up with at last is below (most of it copied from Pphusion's readme](https://github.com/phusion/passenger-docker))

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
WORKDIR /tmp/
# Bundle install is done in tmp to make use of caching.
ADD Gemfile Gemfile
ADD Gemfile.lock Gemfile.lock
RUN bundle install --without sqlite test development
RUN mkdir $APP_HOME
WORKDIR $APP_HOME
ADD . $APP_HOME
RUN cp /tmp/.bundle $APP_HOME/ -r
ADD docker_config/env $APP_HOME/.env
RUN chown -R app:app $APP_HOME

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
{% endhighlight %}


The above file is only used in my local development machine. In production, the
only file I use is a docker-compose.yml

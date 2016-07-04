---
layout: post
title:  "Install Solr 4 on Tomcat 7"
date:   2014-02-01 18:39:14
categories: solr
---


In the company where I work, we deploy our applications on Tomcat. Unfortunately, Solr is bundled with a Jetty container, which is a pretty good and lightwheight container, but we don't want to maintain 2 different containers, especially in production, where ops guys don't like new things they don't know...

You'll find some wiki pages on how to install Solr on Tomcat :

* [Solr with Apache Tomcat](http://wiki.apache.org/solr/SolrTomcat)
* [Running Solr on Tomcat](https://cwiki.apache.org/confluence/display/solr/Running+Solr+on+Tomcat)

These links are useful but I found them incomplete or outdated. You may encouter a bunch of classpath errors and Log4j misconfiguration which is really confusing and discouraging when you're a Solr beginner.

I had some trouble trying to find an elegant and effective way to do it, and that's why I want to share it with you. It's probably not the best way, but it's very simple, very straightforward, with no side effect on Tomcat configuration.
It applies to Tomcat 7 and Solr 4.6, but may of course apply to other versions.


### Solr install

Download and unpack the latest version of Solr, somewhere on the disk.

```bash
cd /tmp
wget "http://mirrors.ircam.fr/pub/apache/lucene/solr/4.6.0/solr-4.6.0.tgz"
cd /opt
sudo tar zxvf /tmp/solr-4.6.0.tgz
sudo chown -R tomcat:tomcat solr-4.6.0
```

Here, I use /opt, but you can install it anywhere as long as Tomcat has access to it. Just to be sure, make Tomcat the owner, so it can read and write in Solr folders.


### Deployment

Now, let's deploy the Solr webapp on Tomcat. To do so, we will take advantage of an XML context configuration file, wich we will use to deploy and configure Solr.

In your Tomcat installation folder, create a new XML context configuration file in conf/Catalina/localhost. The base name of the file will define the Solr context path.

We first need to define ${solr.home}. Here, we use the example Solr configuration provided in the distribution. It is located in /opt/solr-4.6.1/example/solr.

To avoid startup errors, we need to add some jars to the webapp classpath, located in /opt/solr-4.6.1/example/lib/ext. But we don't want to pollute Tomcat's lib folder with Solr jars, so we just declare a virtual loader.

We also need to configure Log4J properly. Here we use the Log4J configuration file provided in Solr distribution in /opt/solr-4.6.1/example/resources, by adding the folder in Solr classpath.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context docBase="/opt/solr-4.6.1/dist/solr-4.6.1.war">
  <Environment
    name="solr/home"
    type="java.lang.String"
    value="/opt/solr-4.6.1/example/solr"
    override="true" />
  <Loader
    className="org.apache.catalina.loader.VirtualWebappLoader"
    virtualClasspath="/opt/solr-4.6.1/example/lib/ext/*.jar;/opt/solr-4.6.1/example/resources/" />
</Context>
```

Save the file, wait for Tomcat to read it and deploy the war. That's it! Solr is up and running.


---
layout: post
title:  "Adding Custom Apache Log Fields to filebeat "
date:   2019-04-28 18:00:00 -0500
categories: tech
tags: [server management, linux]
comments: true
---
## Introduction

The [Apache filebeat module](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-apache.html) (TODO: change this link to a specific version of ES) is very useful for getting Apache log monitoring up and running quickly.
By default, a typical setup would consist of filebeat reading the Apache logs, sending them to elasticsearch (or is it called elastic stack?), and then visualizing them via Kibana.
Lo

** Block diagram here**

Note that there is no need for logstash in this pipeline, since parsing Apache logs is fairly straightforward and can be done with a (ES ingest or filebeat ingest?).
The default Apache module is setup to parse default Apache logs and already provides a great Kibana dashboard for visulizing all the data.

** Picture of default Kibana dashboard **

But what if we want to edit our Apache logging to include more details?
- I wanted to be able to see which virtualhost the log belonged to.
- I wanted to log and visualize how long it was taking to deliver pages.
- I was experimenting with some optimizations for reducing page load times and needed 


# Apache config

Add this line to the bottom of /etc/apache2/apache2.conf

```
# Custom format to record response times
LogFormat "%v:%p %h %l %u %t %D \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined_time
```

# Filebeat config
The filebeat apache module doesn't parse in the virtualhost or the reponse time.

Create a new parser here:
/usr/share/filebeat/module/apache2/access/ingest

copy `default.json` to `vhost_combined_time.json`


https://www.elastic.co/guide/en/elasticsearch/reference/master/grok-processor.html










###### The correct filebeat template

`/etc/filebeat/fields.yml`

Edit that file to add following in the `apache2.access` fields:
```
- name: server_name
  type: keyword
  description: >
    ServerName of the server.
- name: server_port
  type: keyword
  description: >
    Server port.
- name: response_time
  type: long
  description: >
    The HTTP response time (us).
```

Then delete the existing template
```
curl -X DELETE "localhost:9201/_template/filebeat-6.7.1"
```
delete the index too?
```
curl -X DELETE "localhost:9201/filebeat-6.7.1-2019.04.29"
```


```
sudo filebeat setup --template
```
check with:
```
sudo filebeat export template | grep server_name

OR

curl -X GET "localhost:9201/_template/filebeat-6.7.1?pretty" | grep server_port -A 15
```

restart filebeat

you may need to refresh kibana to see field type updates



CREDIT: https://gryzli.info/2019/02/15/advanced-filebeat-configuration/
TOD): change all the 9201 to 9200
## Unknowns

If you have a filebeat instance on machine A, and are piping the data to an elasticsearch instance on machine B, do both of their filebeat configs (ingest/default.json) have to match?
They both specify the same ingest pipeline name, so I'm guessing when filebeat starts it just checks if that name exists already. If it does, it just uses that pipeline as it exists on the elasticsearch instance.
The only way to re-set the ingest pipeline config is to delete it from the elasticsearch database.
So if you deleted the config off of elasticsearch, you would have a race condition where the first instance of filebeat that connects to elasticsearch is the one that sets the new config for the ingest pipeline.

That is just my hypothesis though, I have not tested this, so I may be totally wrong.





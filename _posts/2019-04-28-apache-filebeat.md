---
layout: post
title:  "Updating Multiple Namecheap Domains With ddclient"
date:   2019-04-28 18:00:00 -0500
categories: tech
tags: [server management, linux]
comments: true
---
## Introduction


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








########################### FOUND IN WIKI folder

Title: filebeat apache module: different timestamps for access and error


I setup a Filebeat-Elasticsearch-Kibana stack for the first time the other day.
I wanted to be able to visualize log data from apache.
After installing the [apache filebeat module](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-apache2.html)
I noticed that the timestamps for the apache error logs were offset compared to the access logs.
To confirm this discrepancy, I compared the data in elasticsearch with the log file contents:

DEBUG:

Print Log File Contents:
```
tail -n 1 /var/log/apache2/access.log
tail -n 1 /var/log/apache2/error.log
```

Print latest elasticsearch doc of both access and error:
```
curl -H 'Content-Type: application/json' -XPOST 'http://localhost:9200/filebeat-*/_search?pretty' -d '
{
   "size": 1,
   "sort": { "@timestamp": { "order": "desc" }},
   "query": {
      "exists": { "field": "apache2.access"}
   }
}' | grep @time
```

```
curl -H 'Content-Type: application/json' -XPOST 'http://localhost:9200/filebeat-*/_search?pretty' -d '
{
   "size": 1,
   "sort": { "@timestamp": { "order": "desc" }},
   "query": {
      "exists": { "field": "apache2.error"}
   }
}' | grep @time
```

Note that if you try to inspect the timestamp through the kibana web UI, you may see different results. This is because elasticsearch records timestamps in UTC time, but kibana has a [setting](https://www.elastic.co/guide/en/kibana/current/advanced-options.html) to adjust timestamps to the viewers timezone.


CAUSE:

It appears that the access logs are getting stored into elasticsearch with the correct UTC time, but the error logs have an offset.
This is due to the way that the apache2 filebeat module parses the log data.
Here is a snippet from the access log ingest config (/usr/share/filebeat/module/apache2/access/ingest/pipeline.json)
```
    "date": {
      "field": "apache2.access.time",
      "target_field": "@timestamp",
      "formats": ["dd/MMM/YYYY:H:m:s Z"]
```
The ["Z" at the end](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)
indicates that the timezone offset is present in the log file and should be taken into consideration when creating the timestamp.

Now, when we look at the error logs config:
```
      "date": {
        "field": "apache2.error.timestamp",
        "target_field": "@timestamp",
        "formats": ["EEE MMM dd H:m:s YYYY", "EEE MMM dd H:m:s.SSSSSS YYYY"],
        "ignore_failure": true
      }
```
We see that there is no parsing of the timezone offset, this is because apache2 error logs don't print out the offset by default.
Since all filebeat/elasticsearch has is a timestamp and no knowledge of the timezone that the stamp was recorded in, it simply stores the value incorrectly as UTC.

FIX:

We could probably edit apache's logging config to print out the timezone offset in the log line, but I'd rather stick to default apache logging.
All we need to do is edit the ingest pipeline to tell it which timezone the timestamp from the error log is in.
Update the config (/usr/share/filebeat/module/apache2/error/ingest/pipeline.json) to add the `"timezone": "{{my_timezone}}"` option to the [date processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/date-processor.html):
```
      "date": {
        "field": "apache2.error.timestamp",
        "target_field": "@timestamp",
        "formats": ["EEE MMM dd H:m:s YYYY", "EEE MMM dd H:m:s.SSSSSS YYYY"],
        "timezone": "{{my_timezone}}",
        "ignore_failure": true
      }
```

Now we have to delete the ingest pipeline so that elasticsearch adds a new one with the new configs.
```
curl -X DELETE "localhost:9200/_ingest/pipeline/filebeat-6.6.0-apache2-error-pipeline"
```

You can confirm that elastic search loaded the new ingest pipeline config by inspecting the output of:
```
curl -X GET "localhost:9200/_ingest/pipeline?pretty"
```

### Finishing touches

Go to Kibana's Management page and select Kibana "Index Patterns".
Then click on "Refresh field list" (the refresh symbol) in the upper right corner.


You will notice now how the field types are wrong (string instead of number).
This is because elasticsearch doesn't know what to do with the data coming in labelled as "apache2.access.response_time".
All data gets passed to elasticsearch as a giant string via the http protocol, so the portions of that string that represent the field values need to be converted into their appropriate datatype.
Elasticsearch does this using an "index pattern".
You could manually create and send an index pattern into elasticsearch, but filebeat automatically creates it's own index pattern every 24 hours for a day's worth of logs.
We can edit the filebeat index template to include our new custom fields.

To view the existing index pattern template:
```
filebeat export template
```

EDIT: tthis template is completely wrong/unrelated, see below for correct file
Open the template found here: `/usr/share/filebeat/kibana/6/index-pattern/filebeat.json`
If you pretty print the fields, they will look like this:

```
[
    {
        \"aggregatable\": true,
        \"analyzed\": false,
        \"count\": 0,
        \"doc_values\": true,
        \"indexed\": true,
        \"name\": \"apache2.access.http_version\",
        \"scripted\": false,
        \"searchable\": true,
        \"type\": \"string\"
    },
    {
        \"aggregatable\": true,
        \"analyzed\": false,
        \"count\": 0,
        \"doc_values\": true,
        \"indexed\": true,
        \"name\": \"apache2.access.response_code\",
        \"scripted\": false,
        \"searchable\": true,
        \"type\": \"number\"
    },
    {
        \"aggregatable\": true,
        \"analyzed\": false,
        \"count\": 0,
        \"doc_values\": true,
        \"indexed\": true,
        \"name\": \"apache2.access.body_sent.bytes\",
        \"scripted\": false,
        \"searchable\": true,
        \"type\": \"number\"
    },
    { ... a whole bunch of other ones ... }
]
```
TODO: add link that explains all the fields.

All we need to do is copy the existing "response_code" field and change the name to "response_time".
We will create fields for "server_name" and "server_port" as well.
(Don't forget to backup the existing template)

Add the following to the very end of the "fields" array

```
,{\"aggregatable\":true,\"analyzed\":false,\"count\":0,\"doc_values\":true,\"indexed\":true,\"name\":\"apache2.access.response_time\",\"scripted\":false,\"searchable\":true,\"type\":\"number\"}


,{\"aggregatable\":true,\"analyzed\":false,\"count\":0,\"doc_values\":true,\"indexed\":true,\"name\":\"apache2.access.server_port\",\"scripted\":false,\"searchable\":true,\"type\":\"number\"}

,{\"aggregatable\":true,\"analyzed\":false,\"count\":0,\"doc_values\":true,\"indexed\":true,\"name\":\"apache2.access.server_name\",\"scripted\":false,\"searchable\":true,\"type\":\"string\"}
```

,{\"aggregatable\":true,\"analyzed\":false,\"count\":0,\"doc_values\":true,\"indexed\":true,\"name\":\"apache2.access.response_time\",\"scripted\":false,\"searchable\":true,\"type\":\"number\"},{\"aggregatable\":true,\"analyzed\":false,\"count\":0,\"doc_values\":true,\"indexed\":true,\"name\":\"apache2.access.server_port\",\"scripted\":false,\"searchable\":true,\"type\":\"number\"},{\"aggregatable\":true,\"analyzed\":false,\"count\":0,\"doc_values\":true,\"indexed\":true,\"name\":\"apache2.access.server_name\",\"scripted\":false,\"searchable\":true,\"type\":\"string\"}

Delete the index to make sure:
```
curl -X DELETE "localhost:9200/filebeat-6.7.1-2019.04.29"
```
Restart filebeat

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

you may need to refresg kibana to see field type updates



CREDIT: https://gryzli.info/2019/02/15/advanced-filebeat-configuration/
TOD): change all the 9201 to 9200
## Unknowns

If you have a filebeat instance on machine A, and are piping the data to an elasticsearch instance on machine B, do both of their filebeat configs (ingest/default.json) have to match?
They both specify the same ingest pipeline name, so I'm guessing when filebeat starts it just checks if that name exists already. If it does, it just uses that pipeline as it exists on the elasticsearch instance.
The only way to re-set the ingest pipeline config is to delete it from the elasticsearch database.
So if you deleted the config off of elasticsearch, you would have a race condition where the first instance of filebeat that connects to elasticsearch is the one that sets the new config for the ingest pipeline.

That is just my hypothesis though, I have not tested this, so I may be totally wrong.





Access log and error log are different

Access logs have timeOffset explicitly written out in the log. Error logs however just write out the current time in the system's time zone.

(BTW it's recommended to perform these queries via the "Dev Tools" tab on Kibana)

use the following to list all ingest pipelines:
```
curl -X GET "localhost:9200/_ingest/pipeline"
```
Note how ours is not yet updated.

Delete the pipeline and filebeat will create a new one with the updated settings

```
curl -X DELETE "localhost:9200/_ingest/pipeline/filebeat-6.6.0-apache2-error-pipeline"
```


TODO: write script to "reload" pipeline (delete then restart filebeat)

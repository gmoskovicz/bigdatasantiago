# Big Data Week Santiago

Santiago Big Data workshop

![image](http://santiago.bigdataweek.com/wp-content/uploads/sites/36/2018/03/logo-web-2017.png)

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>


## First lets talk about Architecture

https://docs.google.com/presentation/d/18yq1IWDAUTiRm8mdd9Vydv1jWrJ7X4k7HPeYCQ38M4Q/edit?usp=sharing

![image](https://github.com/gmoskovicz/bigdatasantiago/blob/master/Architecture.png)

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

## The hands on

<br/>
<br/>

### [1] - Create an Elastic Cloud account

1. Login to https://cloud.elastic.co/login
2. Click in Sign up now
3. Lets create a Deployment

![image](https://github.com/gmoskovicz/bigdatasantiago/blob/master/Screen%20Shot%202018-10-16%20at%201.43.38%20PM.png)

<br/>
<br/>

### [2] - Inspect the Elastic Cloud UI and open Kibana

1. Check what you have in your Elastic Cloud cluster
2. Find the Elasticsearch link.
3. Find the Kibana link and login into Kibana.

<br/>
<br/>

### [3] - Load the data logs

1. Go to https://data.ny.gov/Transportation/NYC-Transit-Subway-Entrance-And-Exit-Data/i9wp-a4ja
2. Export > CSV
3. Inspect the dataset
4. Download filebeat: https://www.elastic.co/downloads/beats/filebeat
5. Download the following log: https://github.com/gmoskovicz/bigdatasantiago/blob/master/logs.gz
6. Modify `filebeat.yml` to read from the logs and enable it (comment out `enabled: false`)
7. Modify `output.elasticsearch.hosts` to send to Elastic Cloud, and setup username and password
8. Run filebeat
9. Open kibana and configure index pattern

<br/>
<br/>

### [4] Creating the pipeline to parse the logs

1. Go to Kibana and Devtools
2. Start with the following and create the pipeline, checkoug https://github.com/elastic/elasticsearch/blob/master/libs/grok/src/main/resources/patterns/grok-patterns#L94

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "Parsing the logs",
    "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": [
          ""
        ]
      }
    }
  ]
  },
  "docs": [
    {
      "_source": {
      "message": """     
        83.149.9.216 - - [26/Aug/2014:21:13:42 +0000] "GET /presentations/logstash-monitorama-2013/images/sad-medic.png HTTP/1.1" 200 430406 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"
        """
      }
    }
  ]
}
```

You should end up with something like the following:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "Parsing the logs",
    "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": [
          "%{COMMONAPACHELOG}"
        ]
      }
    },
    {
      "remove": {
        "field": "message"
      }
    },
    {
      "convert": {
        "field": "response",
        "type": "integer"
      }
    },
    {
      "convert": {
        "field": "bytes",
        "type": "long"
      }
    },    
    {
      "date" : {
        "field" : "timestamp",
        "target_field" : "@timestamp",
        "formats" : ["dd/MMM/yyyy:HH:mm:ss Z"]
      }
    }
  ]
  },
  "docs": [
    {
      "_source": {
        "message": """     
        83.149.9.216 - - [26/Aug/2014:21:13:42 +0000] "GET /presentations/logstash-monitorama-2013/images/sad-medic.png HTTP/1.1" 200 430406 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"
        """
      }
    }
  ]
}
```

<br/>
<br/>

### [5] Add the pipeline and configure it

1. Add the pipeline and configure it in `filebeat.yml`

```
PUT _ingest/pipeline/parse_logs
{
  "description": "Parsing the logs",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": [
          "%{COMMONAPACHELOG}"
        ]
      }
    },
    {
      "remove": {
        "field": "message"
      }
    },
    {
      "convert": {
        "field": "response",
        "type": "integer"
      }
    },
    {
      "convert": {
        "field": "bytes",
        "type": "long"
      }
    },    
    {
      "date" : {
        "field" : "timestamp",
        "target_field" : "@timestamp",
        "formats" : ["dd/MMM/yyyy:HH:mm:ss Z"]
      }
    }
  ]
}
```

Then add `pipeline: parse_logs` under `output.elasticsearch` in `filebeat.yml`

<br/>
<br/>

### [6] Delete data and start from scratch

1. `DELETE filebeat-*` in Devtools
2. Delete the `registry` file under `data` int he filebeat folder
3. Start filebeat again
4. Go to Management>Index Templates and refresh the index template in Kibana

<br/>
<br/>

### [7] Create dashboard to visualize the logs

Create 4 visualizations and a dashboard

![image](https://github.com/gmoskovicz/bigdatasantiago/blob/master/dashboard.png)

# ElasticCloud-Intro

Talk for the Elastic Triangle User Group in Durham NC.  Lightning fast intro to getting data to the Elastic Cloud via Logstash.

**Agenda** (20 minutes!)

- Intro (1 min)
- Demo (5 min)
- Design considerations (5 min)
- Architecture (2 min)
- Setup account and cluster (2 min)
- Philosophy (2 min)
- Getting started with moving data (2 min)
- Help with parsing (2 min)

**Intro** (1 min)

- Me: I am an SRE at IBM working on the performance and incident services we provide in IBM's Cloud.  My introduction to Elastic was building Logstash integrations for 31 device types for a bank.  This was a great gig, and we learned a whole bunch :)

**Demo** (5 min)

![Kibana Dashboard](https://user-images.githubusercontent.com/25182304/32256184-e9272b5c-be83-11e7-873f-5dfb011c8180.png)

**Design considerations** (5 min)

- Organizing your Data: Think about how your company works.  If your Ops team is broken into silos (DBAs, Sys Admins, App Admins, Network People, Firewall People) that don't work together, then you need to have a way to group "stuff" from various sources.

- A common format: You might look at <a href="https://ibm.co/2A3qIiz" target="_blank"> Netcool Fields</a> or <a href="https://support.pagerduty.com/v1/docs/pd-cef" target="_blank">PD-CEF</a> for inspiration.  While I think this is absoluteley necessary at a higher level, I also see the need for log specific formats.  Here is why: one of the cool things about the Elastic Stack is the ability to chart data from logs.  If you want to chart the response time of a web server by using the response time field in the access logs, then you need a field for that.  So, what I propose is that you start out by deciding which fields you need based on the above **Organizing your Data** bullet and add those to the fields for your source.  For example, if you are a DevOps shop, then you might want to have fields for **Service**, **Env**, **Customer**.  But if you are old school Ops, maybe you want **DataCenter**, **Team**, **OS**. 

**Architecture** (2 min)

![Events -> Netcool -> Logstash -> Elasticsearch -> Kibana](https://user-images.githubusercontent.com/25182304/32248340-c0aeeade-be5b-11e7-8789-a86c9e18c277.png)

**Setup account** and cluster (2 min)

**Philosophy** (2 min)

This comes from Jordan Sissel, who is the main contributor to Logstash

![If a new user has a hard time, IT'S A BUG!](https://user-images.githubusercontent.com/25182304/32244045-b4ef4364-be4d-11e7-8726-c79d62af2946.png)

@jordansissel

**Getting started with moving data** (2 min)

- Minimal logstash config listening on port 1235 and writing to STDOUT
```
input {
  tcp {
    type => "netcool"
    codec => "plain"
    port => 1235
  } # end tcp
} # end input

output {

  stdout { codec => rubydebug } # end stdout 

} # end output
```
And the Ruby Debug output.  The line I look at is the **message**
```
{
      "@version" => "1",
          "host" => "10.106.48.6",
    "@timestamp" => 2017-11-01T14:04:28.480Z,
       "message" => "UPDATE\"sidr30eoisnco01.noc.envops.ibmserviceengage.com\";\"sidr30eoisnco01.noc.envops.ibmserviceengage.com\";\"ConnectionStatus\";\"\";2;\"GATEWAY: Gateway Reader/Writer connected from host sidr30eoisnco01.noc.envops.ibm (ID: 3).\";2017-11-01T09:04:01-0500;2017-10-29T13:42:01-0500;2017-11-01T09:04:01-0500;13;\"\";\"\";\"\";2;\"DEMO\"",
          "type" => "netcool",
          "port" => 55298
}
```
- What is grok?
Think of grok as a collection of regular expression patterns.  Here are some default grok patterns, and you will end up with your own library if you get into this. https://grokdebug.herokuapp.com/patterns#
```
USERNAME [a-zA-Z0-9._-]+
USER %{USERNAME}
INT (?:[+-]?(?:[0-9]+))
BASE10NUM (?<![0-9.+-])(?>[+-]?(?:(?:[0-9]+(?:\.[0-9]+)?)|(?:\.[0-9]+)))
NUMBER (?:%{BASE10NUM})
BASE16NUM (?<![0-9A-Fa-f])(?:[+-]?(?:0x)?(?:[0-9A-Fa-f]+))
BASE16FLOAT \b(?<![0-9A-Fa-f.])(?:[+-]?(?:0x)?(?:(?:[0-9A-Fa-f]+(?:\.[0-9A-Fa-f]*)?)|(?:\.[0-9A-Fa-f]+)))\b

POSINT \b(?:[1-9][0-9]*)\b
NONNEGINT \b(?:[0-9]+)\b
WORD \b\w+\b
NOTSPACE \S+
SPACE \s*
DATA .*?
GREEDYDATA .*
QUOTEDSTRING (?>(?<!\\)(?>"(?>\\.|[^\\"]+)+"|""|(?>'(?>\\.|[^\\']+)+')|''|(?>`(?>\\.|[^\\`]+)+`)|``))
UUID [A-Fa-f0-9]{8}-(?:[A-Fa-f0-9]{4}-){3}[A-Fa-f0-9]{12}

# Networking
MAC (?:%{CISCOMAC}|%{WINDOWSMAC}|%{COMMONMAC})
CISCOMAC (?:(?:[A-Fa-f0-9]{4}\.){2}[A-Fa-f0-9]{4})
WINDOWSMAC (?:(?:[A-Fa-f0-9]{2}-){5}[A-Fa-f0-9]{2})
COMMONMAC (?:(?:[A-Fa-f0-9]{2}:){5}[A-Fa-f0-9]{2})
.
.
.
```

Let's pull out the message line and pop that in the grok debugger https://grokdebug.herokuapp.com/
![Grok Debugger](https://user-images.githubusercontent.com/25182304/32279443-2337e014-beef-11e7-99f7-c2030917a4a7.png)

Notice that the message format looks like a semicolon delimited format:
![Lots of semicolons](https://user-images.githubusercontent.com/25182304/32280229-89c27234-bef1-11e7-8c9d-99e77497c382.png)

let's work on a pattern for that:
![DATA:a, DATA:b, etc.](https://user-images.githubusercontent.com/25182304/32280820-72a08ae4-bef3-11e7-8cd1-077dd83a6b55.png)

At this point I realize that since this is truly just a semicolon delimited string I would be better off using a plugin designed for CSVs.  But one more grok pattern is necessary, and you will very often see some leading or trailing text that needs to be stripped, so here is what I did:

```
filter {
  
  mutate {
    # The message begins with either UPDATE or INSERT depending on whether it
    # is a new event or an update.  In order for the CSV parser to succeed this
    # initial verb needs to be removed.
    gsub => ["message", "^\w*", ""]
  }

} # end filter
```
This removes the first **word** from the message, note the change in the Ruby Debug.  Now the **message** is conforming semicolon delimited text with double quotes around strings:
```
{
      "@version" => "1",
          "host" => "10.106.48.6",
    "@timestamp" => 2017-11-01T15:08:53.471Z,
       "message" => "\"sidr30eoisnco01.noc.envops.ibmserviceengage.com\";\"sidr30eoisnco01.noc.envops.ibmserviceengage.com\";\"ConnectionStatus\";\"\";2;\"GATEWAY: Gateway Reader/Writer connected from host sidr30eoisnco01.noc.envops.ibm (ID: 3).\";2017-11-01T10:08:01-0500;2017-10-29T13:42:01-0500;2017-11-01T10:08:01-0500;13;\"\";\"\";\"\";2;\"DEMO\"",
          "type" => "netcool",
          "port" => 56670
}
```
Before we switch to the CSV plugin let's use the grok pattern (just showing the filter section):
```
filter {

  mutate {
    # The message begins with either UPDATE or INSERT depending on whether it
    # is a new event or an update.  In order for the CSV parser to succeed this
    # initial verb needs to be removed.
    gsub => ["message", "^\w*", ""]
  }

  grok {
    match => { "message" => "%{DATA:a};%{DATA:b};%{DATA:c};%{DATA:d};%{DATA:e};%{DATA:f};%{DATA:g};%{DATA:h};%{DATA:i};%{DATA:j};%{DATA:k};%{DATA:l};%{DATA:m};%{DATA:n};%{GREEDYDATA:o}"}
  } #end grok

} # end filter
```

Here is the Ruby Debug output:
```
{
             "a" => "\"link2\"",
             "b" => "\"link2\"",
             "c" => "\"Link\"",
             "d" => "\"\"",
             "e" => "0",
             "f" => "\"Link Up on port\"",
             "g" => "2017-11-01T10:16:16-0500",
             "h" => "2017-11-01T09:39:08-0500",
             "i" => "2017-11-01T10:16:15-0500",
             "j" => "2",
             "k" => "\"\"",
       "message" => "\"link2\";\"link2\";\"Link\";\"\";0;\"Link Up on port\";2017-11-01T10:16:16-0500;2017-11-01T09:39:08-0500;2017-11-01T10:16:15-0500;2;\"\";\"\";\"Mortgage Services\";0;\"DEMO\"",
          "type" => "netcool",
             "l" => "\"\"",
             "m" => "\"Mortgage Services\"",
             "n" => "0",
             "o" => "\"DEMO\"",
    "@timestamp" => 2017-11-01T15:16:47.806Z,
          "port" => 56754,
      "@version" => "1",
          "host" => "10.106.48.6"
}
```

- then grok debugger
- then csv plug-in
- then elasticsearch output

- Beginner Logstash config
```
input {
  tcp {
    type => "netcool"
    codec => "plain"
    port => 1235
  } # end tcp
} # end input

filter {
  mutate {
    # The message begins with either UPDATE or INSERT depending on whether it 
    # is a new event or an update.  In order for the CSV parser to succeed this 
    # initial verb needs to be removed.
    gsub => ["message", "^\w*", ""]
  }

  csv {
    # This filter splits the message field into the indicated fields (columns).  The 
    # name and order of the fields comes from the socket gateway mapping file.
      skip_empty_columns => true
      separator => ";"
      columns => ["Node", "NodeAlias", "AlertGroup", "AlertKey", "Severity", "Summary", "StateChange", "FirstOccurrence", "LastOccurrence", "Type", "Location", "Customer", "Service", "OriginalSeverity", "ServerName" ] 
  } #end csv

} # end filters

output {
  stdout { codec => rubydebug } # end stdout 
} # end output
```
- Use the Ruby Debug output
```
{
            "AlertKey" => "Disk 85% full",
           "NodeAlias" => "Tokyo",
                "Node" => "Tokyo",
             "Service" => "Online Banking",
            "Severity" => "3",
     "FirstOccurrence" => "2017-10-29T14:00:57-0500",
             "message" => "\"Tokyo\";\"Tokyo\";\"Stats\";\"Disk 85% full\";3;\"Diskspace alert\";2017-10-29T14:06:38-0500;2017-10-29T14:00:57-0500;2017-10-29T14:06:38-0500;0;\"\";\"\";\"Online Banking\";3;\"DEMO\"",
                "type" => "netcool",
          "AlertGroup" => "Stats",
                "Type" => "0",
          "@timestamp" => 2017-10-29T19:06:39.164Z,
                "port" => 41404,
         "StateChange" => "2017-10-29T14:06:38-0500",
          "ServerName" => "DEMO",
            "@version" => "1",
                "host" => "10.106.48.6",
             "Summary" => "Diskspace alert",
    "OriginalSeverity" => "3",
      "LastOccurrence" => "2017-10-29T14:06:38-0500"
}
```
- Connect to Elastic Cloud
Once the above Ruby Debug is showing data flowing into Logstash and being parsed from the CSV into fileds (for example, in the above we see that the field Summary is created and populated with Diskspace alert) it is time to send to Elastic Cloud.  Here is the output stanza for my cluster in Elastic Cloud:
```
  elasticsearch {
    hosts => "https://46524239483934789ded08315e5d215b.us-east-1.aws.found.io:9243/"
    user => "logstash_agent"
    password => "C@tF00d"
    index => "logstash-netcool"
  }
  ```

**Help with parsing** (2 min)

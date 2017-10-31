# ElasticCloud-Intro
Talk for the Elastic Triangle User Group in Durham NC.  Lightning fast intro to getting data to the Elastic Cloud via Logstash.

**Agenda** (20 minutes!)
- Demo (5 min)
- Design considerations (5 min)
- Architecture (2 min)
- Setup account and cluster (2 min)
- Philosophy (2 min)
- Getting started with moving data (2 min)
- Help with parsing (2 min)

**Demo** (5 min)

**Design considerations** (5 min)
- Me: I am an SRE at IBM working on the performance and incident services we provide in IBM's Cloud.  My introduction to Elastic was building Logstash integrations for 31 device types for a bank.  This was a great gig, and we learned a whole bunch :)

- Organizing your Data: Think about how your company works.  If your Ops team is broken into silos (DBAs, Sys Admins, App Admins, Network People, Firewall People) that don't work together, then you need to have a way to group "stuff" from various sources.

- A common format: You might use <a href="https://support.pagerduty.com/v1/docs/pd-cef" target="_blank">PD-CEF</a>, or something similar.  While I think this is absoluteley necessary at a higher level, I also see the need to think about specific log formats.  One of the cool things about Elastic is the ability to chart data from logs.  If you want to chart the response time of a web server by using the response time field in the access logs, then you need a field for that.  So, what I propose is that you start out by deciding which fields you need based on the above **Organizing your Data** bullet and add those to the fields for your source.  I will show you this in a few minutes.

**Architecture** (2 min)

![Events -> Netcool -> Logstash -> Elasticsearch -> Kibana](https://user-images.githubusercontent.com/25182304/32245126-23b52252-be51-11e7-8736-4259b0a7e749.png)

**Setup account** and cluster (2 min)

**Philosophy** (2 min)

This comes from Jordan Sissel, who is the main contributor to Logstash

![If a new user has a hard time, IT'S A BUG!](https://user-images.githubusercontent.com/25182304/32244045-b4ef4364-be4d-11e7-8726-c79d62af2946.png)

@jordansissel

**Getting started with moving data** (2 min)
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

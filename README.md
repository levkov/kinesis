## AWS Kinesis


Coralogix provides integration with AWS Kinesis using Logstash, so you can send your logs from anywhere and parse them according to your needs.  

### Table of contents

1. Prerequisites
2. Usage
3. Installation
4. Configuration

### Prerequisites
Have Logstash installed, for more information on how to install: https://www.elastic.co/guide/en/logstash/current/installing-logstash.html

### Usage

**Private Key** – A unique ID which represents your company, this Id will be sent to your mail once you register to Coralogix.

**Application Name** – Used to separate your environment, e.g. SuperApp-test/SuperApp-prod.

**SubSystem Name** – Your application probably has multiple subsystems, for example: Backend servers, Middleware, Frontend servers etc. in order to help you examine the data you need, inserting the subsystem parameter is vital.

**Kinesis Stream Name** - Stream represents an ordered and immutable list of messages. Each stream has a unique name. 

**Region** - The AWS region for Kinesis. 

### Installation

```bash
logstash-plugin install logstash-output-coralogix_logger
logstash-plugin install logstash-input-kinesis
```

If you are not sure where logstash-plugin is located, you can check here:  
https://www.elastic.co/guide/en/logstash/current/dir-layout.html

### Configuration

Open your Logstash configuration file and add AWS Kinesis input and Coralogix output. (More information about Logstash Input Kinesis plugin: https://www.elastic.co/guide/en/logstash/current/plugins-inputs-tcp.html )

```javascript
input {

  tcp {
    port => 6044
    codec => "json_lines"
  }

}


output {
    coralogix_logger { 
        config_params => {
            "PRIVATE_KEY" => "11111111-1111-1111-1111-111111111111"
            "APP_NAME" => "Logstash Tester"
            "SUB_SYSTEM" => "Logstash subsystem"
        } 
        log_key_name => "message"
        timestamp_key_name => "YOUR_TIMESTAMP_FIELD"


        is_json => true
    }
}  
```
**Input**  
TCP port can be changed to other port number.

**Output**  
The first key (config_params) is mandatory while the other two are optional. 

**Timestamp:**  Coralogix automatically generates the timestamp based on the log arrival time.  If you rather use your own timestamp, use the “timestamp_key_name” to specify your timestamp field, and it will be read from your log. 

```json
{ 
    "@timestamp":"2018-10-28T13:37:52.159Z",
    "host":"localhost",
    "source":"jenkins",
    "data":{   
              "buildDuration":19,
              "buildHost":"Jenkins",
              "projectName":"test",
              "rootProjectName":"test",
              "fullDisplayName":"test #16",
              "rootProjectDisplayName":"#16",
              "buildNum":16,
              "id":"16",
              "displayName":"#16",
              "buildVariables":{
                                  "BUILD_NUMBER":"16",
                                  "_":"/usr/bin/daemon",
                                  "NODE_LABELS":"master",
                                  "SHELL":"/bin/bash",
                                  "JOB_URL":"http://XXXXXXXX:8080/job/test/",
                                  "JOB_NAME":"test",
                                  "USER":"jenkins",
                                  "JENKINS_URL":"http://XXXXXXXX:8080/",
                                  "HOME":"/var/lib/jenkins",
                                  "XDG_RUNTIME_DIR":"/run/user/109",
                                  "CLASSPATH":"",
                                  "WORKSPACE":"/var/lib/jenkins/workspace/test",
                                  "NODE_NAME":"master",
                                  "JOB_DISPLAY_URL":"http://XXXXXXXX:8080/job/test/display/redirect",
                                  "MAIL":"/var/mail/jenkins",
                                  "JENKINS_HOME":"/var/lib/jenkins",
                                  "XDG_SESSION_ID":"c1",
                                  "RUN_DISPLAY_URL":"http://XXXXXXXX:8080/job/test/16/display/redirect",
                                  "HUDSON_HOME":"/var/lib/jenkins",
                                  "BUILD_ID":"16",
                                  "BUILD_TAG":"jenkins-test-16",
                                  "PWD":"/var/lib/jenkins",
                                  "JOB_BASE_NAME":"test",
                                  "PATH":"/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games",
                                  "BUILD_URL":"http://XXXXXXXX:8080/job/test/16/",
                                  "RUN_CHANGES_DISPLAY_URL":"http://XXXXXXXXX:8080/job/test/16/display/redirect?page=changes",
                                  "EXECUTOR_NUMBER":"0","LANG":"en_US.UTF-8",
                                  "LOGNAME":"jenkins",
                                  "BUILD_DISPLAY_NAME":"#16",
                                  "JENKINS_SERVER_COOKIE":"XXXXXXXXX",
                                  "HUDSON_URL":"http://XXXXXXXX:8080/",
                                  "HUDSON_SERVER_COOKIE":"XXXXXXXX",
                                  "SHLVL":"1"
                                             },
              "rootFullProjectName":"test",
              "sensitiveBuildVariables":[],
              "url":"job/test/16/",
              "rootBuildNum":16,
              "result":"SUCCESS",
              "fullProjectName":"test",
              "buildLabel":"master"},
              "port":54848,
              "message":
                  [
                     "Started by user XXXXXXXX","Building in workspace /var/lib/jenkins/workspace/test",
                     "[test] $ /bin/sh -xe /tmp/jenkins1446349865733621641.sh","+ echo test","test"],
                     "tags":[""],
                     "source_host":"http://XXXXXXXX:8080/",
                     "@buildTimestamp":"2018-10-28T14:37:52.047+0100","@version":1
          }
}
```
Because your Jenkins job log is a JSON object, in the case you don’t want to send the entire JSON, rather just a portion of it, you can write the value of the key you want to send in the log_key_name.

For instance, in the above example, if you write log_key_name message then only the value of message key will be sent to Coralogix. If you do want to send the entire message then you can just delete this key.

Restart Logstash.


### Jenkins Configuration

Using Jenkins "Plugin Manager" please install Logstash plugin. (More information about Jenkins Logstash Plugin: https://wiki.jenkins.io/display/JENKINS/Logstash+Plugin )

In configuration section of Jenkins server, go to Logstash section and enable "Enable sending logs to an Indexer" option.  
Set Indexer Type  -> "Logstash TCP".  
Set your Logstash	Host name.
Set same port as in Logstash TCP lugin configuration. (In our example it's 6044).

Inside Freestyle job, please go to "Post-build Actions" section and choose "Send console logs to Logstash" option.
In the case of Pipeline jobs and Pipeline Script usage -  please reffer to examples in Logstash plugin documentation: https://wiki.jenkins.io/display/JENKINS/Logstash+Plugin


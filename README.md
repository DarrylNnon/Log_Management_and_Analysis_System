# LOG MANAGEMENT AND ANALYSIS SYSTEM
This project focuses on gathering logs from multiple systems, ensuring logs are properly rotated and archived, analyzing logs for insights, and vizualizing key data. Each step below is essential for building a comprehensive logging infrastructure.

Step 1: I Set up a Centralized logging system with syslog/rsyslog

Centralized logging with rsyslog (and advanced syslog) lets me gather logs from multiple server into a single host, making it easier to monitor and manage.

1.1 I Configure the Central Log Server

1- I install rsyslog on the central server if it's not already installed:

-> sudo apt update
-> sudo apt install rsyslog

2- I edit the rsyslog configuration:
OPen  /etc/rsyslog.conf and enaable the UDP or Tcp listener.

-> sudo nano /etc/rsyslog.conf

Uncomment these lines to allow log reception:

# Provides UDP syslog reception
module(load="imudp")
input(type="imudp" port="514")

# Provides TCP syslog reception
module(load="imtcp")
input(type="imtcp" port="514"

    3- I create log file Rules:
 To store logs from each remote host separately, i edit or add to /etc/rsyslog.d/remote.conf
 
   Add:
   # save logs from remote hosts to their own files
   template(name="RemoteLogs" type="string" string="/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log")
   *.* ?Remotelogs
   
   4- Restart rsyslog:
   -> sudo systemctl restart rsyslog
   
   5- Configure Firewall to allow rsylog traffic:
   
   -> sudo ufw allow 514/tcp
   -> sudo ufw allow 514/udp
   
   1.2 I configure Client Servers
 On each client server that will send logs to the central server:
 
   1. I install rsyslog if it's not already installed:
   
   -> sudo apt update
   -> sudo apt install rsyslog
   
   2- Edit rsyslog configuration: 
 Open  /etc/rsyslog.conf and add the central server's IP address:
 
 -> sudo nano /etc/rsyslog.conf
 
   Add:
   *.* @<central-server-ip>:514  # For UDP
   *.* @<central-server-ip>:514  # For TCP
   
   3- Restart rsyslog:
   -> sudo systemctl restart rsyslog
   
   
   Step 2: I implement Log Rotation and Archival
 Log rotation prevent disk space from being consumed by logs and keeps files manageable.
 
   1- Edit logrotate configuration: 
 Custom log rotation rules for 
 /var/log/remote can be added in /etc/logrotate.d/remote-logs.
 
 -> sudo nano /etc/logrotate.d/remote-logs
 
  Add the following:
 -> /var/log/remote/*/*.log {
      daily
      missingok
      rotate 14
      compress
      delaycompress
      notifempty
      create 0640 syslog adm
      sharedscripts
      postrotate
           systemctl reload rsyslog > /dev/null
      enddscript
  }
  
  
  2- I test the configuration:
 I run logrotate in debug mode to confirm it will process files as expected:
 
 ->sudo logrotate -d /etc/logrotate.d/remote-logs
 
 
   Step 3: Parse Logs and Generate ALerts using logwatch
 Logwatch is a powerful tool for parsing logs and sending regular summaries of system activity.
 
    1. I install logwatch:
    
   -> sudo apt install logwatch
   
   2- I Configure Logwatch:logwatch configuration is in
   /etc/logwatch/conf/logwatch.conf
   
   -> sudo nano /etc/logwatch/conf/logwatch.conf
   
   Modify settings like:
   
   MailTo = root
   Details = low
   Range = Yesterday
   Service = All
   
   
   3 - I test Logwatch:
I run a test of logwatch to ensure it parses logs and emails the results:

-> sudo logwatch --detail Low --mailto myemail@example.com --range today

  
   4- I automate logwatch Reports:
 To receive regular reports, i create a cron job to run Logwatch daily:
 
 -> sudo crontab -e
 
   Add:
 -> 0 7 * * *  /usr/sbin/logwatch --output mail
 
 
 Step 4: Create Vizualizations of Log Data
 
 Visualizing logs helps me see trends and identify anomalies. Here i will use the ELK stak (Elasticsearch, Logstash, Kibana) for visualization.
 
  4.1 I install and configure Elasticsearch
  1. I install Elasticsearch on the central logging server:
  
  -> sudo apt install elasticsearch
  
  2- I configure Elasticsearch: 
 I edit /etc/elasticseach/elasticsearch.yml to set network binding and data paths.
 
   -> sudo nano /etc/elasticsearch/elasticsearch.yml
   
   Update:
   -> network.host: localhost
   
   3- Start and Enable Elasticsearch:
   
   -> sudo systemctl start elasticsearch
   -> sudo systemctl enable elasticsearch
   
   
   4.2 I install and COnfigure Logstash
Logstash processes logs sending them to elasticsearch

    1. I install logstash
    -> sudo apt install logstash
    
    2. I create a Logstash configuration:
 I Configure logstash to accept rsyslog data output it to elasticsearch.
 
 -> sudo nano /etc/logstash/conf.d/syslog.conf
 
   Add:
   
   Input {
       tcp {
              port => 5044
        }
        udp {
             port => 5044
        }
    }
    
    filter {
         grok {
              match => { "message" => "%{SYSLOGBASE}"} 
         }
    }
    
    output  {
        elasticsearch {
             hosts => ["localhost:9200"]
         }
   }
   
   
   3- I start and enable logstash:
   
   -> sudo systemctl start logstash
   -> sudo systemctl enable logstash
   
   
   4.3 Install and Configure Kibana
   
 Kibana is the visualization interface for data stored in Elasticsearch.
 
 1. I install kibana:
 -> sudo apt install kibana
 
 2- I configure Kibana:

Edit /etc/kibana/kibana.yml to set up network binding and elasticsearch connection

-> sudo nano /etc/kibana/kiban.yml

  Update:
 -> server.host: "localhost"
 -> elasticsearch.hosts: ["http://localhost:9200"]
 
 3 - I start and Enable Kibana:
 -> sudo systemctl start kibana
 -> sudo systemctl enable kibana
 
 4- Access kibana: Open my browser and go to http://<my-server-ip>:5601.
 
 5- I create visualizations:
    * I use kibana to create visualizations of key metrics, such as failed login attempts, SSH access, and system performance.
    
    
    
    Review and Maintain the system
  1- Check system Health: Regularly review system logs, logstash pipelines, and elasticsearch indices.
  
  2- Update configurations: Adjust log parsing and visualization rules as logging needs evolve.
  
  3. Scale as Needed: For larger infrastructures, consider moving to a managed log solution or scaling elasticsearch and Logstash for distributed deployments.
  
  
  
  This approach provides me with a secure, centralized log management system, complete with log rotation, analysis, alerting, and visualization. This setup can be tailored further based on my infrastructure's specific logging requirements.
            
   


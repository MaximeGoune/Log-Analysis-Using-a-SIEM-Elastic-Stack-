# Log-Analysis-Using-a-SIEM-Elastic-Stack-
Log Analysis: Using a SIEM with Elastic Stack

OBJECTIVE 1
Ingest Log Data with a SIEM

This lab environment uses four tools from the Elastic Stack. 
Three docker containers,
 named
 ir-elasticsearch, 
ir-kibana, and ir-logstash,
 run Elasticsearch, Kibana, and Logstash respectively. You can find these docker containers running on the Elastic Stack Console and Logstash Console in the lab environment. The fourth tool, Winlogbeat, is installed on the Windows Desktop in the lab environment.


What is Winlogbeat?

👉 Winlogbeat is a lightweight log shipper from the Elastic Stack (ELK).
It is used to:
📤 Collect Windows Event Logs and send them to a central system (like Elasticsearch or Logstash)
Telemetry data is information that’s automatically collected and transmitted from a remote system to another location for monitoring, analysis, and control.

Logstash : processes the telemetry
Elasticsearch  : stores and indexes it
Kibana : lets analysts investigate it

**First, check the configuration files for Logstash. Logstash is a tool that ingests data from multiple sources, transforms it, and then sends it to stash in an index or other centralized storage.

Type: cd lab_security-log-analysis-using-a-siem/

Use ll to show the files in the directory.

Note the files logstash_sample.yml, logstash.yml, and logstash.conf.


<img src="https://imgur.com/DXFDMrg.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

<img src="https://imgur.com/1AJoFh8.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

<img src="https://imgur.com/dFe3z8l.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

Identify the various sections for configuration, noting the
 Node Identity, Pipeline Settings, and X-Pack Settings. 
 These are the most commonly changed settings when configuring a Logstash instance.

 
<img src="https://imgur.com/U12M4IR.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

1-logstash.yml — Logstash Instance Configuration 
This file controls the behavior of the Logstash process itself.
It defines things like:
Node Identity
node.name
path.data
Pipeline Settings
pipeline.workers
pipeline.batch.size
pipeline.id (default pipeline)
X-Pack Settings
Monitoring
Security
Management
Logging settings
Log level
Log file paths
logstash.conf — Pipeline Configuration



nano logstash.conf

A pipeline in Logstash is an event processing operation that's defined to tell Logstash how to input, filter, parse, and output data. In this lab, you're starting with a configuration for a single pipeline that ingests the dpkg.log and sample auth.log, and outputs it to the Elasticsearch host for indexing. The data is being sent to the index by-date for ir-logstash

<img src="https://imgur.com/sWgjI9O.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

Explanation : 
  /var/log/dpkg.log
This is a log file about package installs/updates.

start_position => "beginning" means:
“Start reading from the very first page, not the end.”

/usr/share/logstash/auth.log
This is an authentication log.

Same rule:
“Start reading from the beginning.”


validate that the ir-logstash docker container is up and running
sudo docker ps -a

<img src="https://imgur.com/xqGfWD6.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />


sudo docker exec -it ir-logstash /bin/bash

Enter the logstash container by typing sudo docker sudo docker exec -it ir-logstash /bin/bash
<img src="https://imgur.com/eX9BphH.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<img src="https://imgur.com/VBe0vAU.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

Modify the pipeline configuration to change the first input from only /var/log/dpkg.log to /var/log/*.log so any log in the /var/log/ directory will be sent to the Elastic Stack. Enter the command

sed 's/dpkg\.log/*.log/' pipeline/logstash.conf  > b && mv b pipeline/logstash.conf
You can then run cat pipeline/logstash.conf to see the change.

<img src="https://imgur.com/VBe0vAU.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />


Now we  connect to the Windows Desktop.
pen Firefox, navigate to Kibana using the following address and look at the data being ingested.

http://172.31.24.100:5601
Now click the menu (☰) icon, scroll down to the Management section, and click Stack Management.
nside of the Stack Management dashboard, use the left panel to find the Kibana section, then click Index Patterns.
Click the Create index pattern button in the top-right of the dashboard.

An index pattern selects the data and allows you to define the properties of the fields within that data. They're helpful for ingesting logs and searching data. The index pattern can point to a specific log for a day or to a set of indices. It's common to use index patterns that match an index or log source by name, with a wildcard to pull logs from all dates within the index.

You should see four index patterns for the data prebuilt into this Kibana instance, along with one from the new Logstash data labeled ir-logstash-<date>.
Click the Create index pattern button in the top-right of the dashboard.
<img src="https://imgur.com/Vsh8PSr.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

<img src="https://imgur.com/VBe0vAU.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

<img src="https://imgur.com/ZZphJOJ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />


<img src="https://imgur.com/2x16Y9V.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />


<img src="https://imgur.com/1XYNH20.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />



You should see four index patterns for the data prebuilt into this Kibana instance, along with one from the new Logstash data labeled ir-logstash-<date>.

Note: This data isn't normalized and uses some generic log fields.

Click the menu (☰) icon, then click Discover.


<img src="https://imgur.com/PHpc5Xw.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
Use the time picker in the top-right of the dashboard to change the search range to Last 24 hours, then click Apply.

Note the logs that appear in the search. They're from the dpkg.log inside of the Logstash container, as well as the sample auth.log file.
<img src="https://imgur.com/PHpc5Xw.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

In the search field, type in sudo to view some examples of the auth.log events that have been ingested.

Notice that normalized fields still aren't extracted at this point. You can use custom Grok filters in Logstash to parse and enrich the data with structured fields for normalization.
<img src="https://imgur.com/z1iGe0G.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

Configure Winlogbeat to Send Event Data to Elasticsearch

To explore event data with extracted and normalized fields, you'll add a new log source to the deployment using another tool in the Elastic Stack called Winlogbeat. Winlogbeat is a lightweight shipper for Windows event logs that installs as a Windows service. It can send logs to either Elasticsearch or Logstash within the Elastic Stack.

Minimize the Firefox browser in the Windows Desktop.

Open File Explorer by clicking the folder icon on the taskbar

In the File Explorer window, go to the top menu bar and click the View tab.

Check the box to enable viewing of Hidden items.
<img src="https://imgur.com/Uzzxy35.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
Navigate to the following path: C:\ProgramData\chocolatey\lib\winlogbeat\tools
<img src="https://imgur.com/51Nuadg.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

<img src="https://imgur.com/51Nuadg.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

Double-click the winlogbeat.yml (Yaml Source File) fille to open it in Visual Studio Code.

Locate the section labeled winlogbeat.event_logs.

Note the various Windows events that are included here. These are the events that Winlogbeat will send to the Elastic Stack.

<img src="https://imgur.com/Le3TaoG.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />


Scroll down to Elasticsearch Output section. Change the value of the hosts to the Elastic Stack instance address: 172.31.24.100:9200

<img src="https://imgur.com/tyQPbcK.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

<img src="https://imgur.com/tyQPbcK.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />



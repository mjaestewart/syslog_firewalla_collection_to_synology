################
**Prerequisite**
################
Did you setup the syslog inputs on the synology in log center? 

**If not** go to `Log Center` > `Log Receiving` > `Create` > Give your  connection a `name`, then specify whether you want to use `TCP or UDP` on `port 514`. `BSD` format is fine as well. 


**###############################**
**Firewalla Syslog Config**
**###############################**
```
# deifne global workDirectory for saving the state file of log messages.
global(workDirectory="/var/spool/rsyslog")

# enable the Rsyslog imfile module processing text files or logs.
module(load="imfile" PollingInterval="10")


# define template for StandardSyslogFormat for processing log messages.
# that will be forwarded to rsyslog server
template(
    name="StandardSyslogFormat"
    type="string"
    string="<%PRI%>%TIMESTAMP:::date-rfc3339% %HOSTNAME% %syslogtag:1:32%%msg:::sp-if-no-1st-sp%%msg%"
    )


# define ruleset "forwardSysLogs" with action object to send logs to rsyslog server
# define the queue
ruleset(name="forwardSysLogs") {
    action(
        type="omfwd"
        target="172.16.2.20"  # set your Synology Syslog Server NAS IP
        port="514"  # Specify port number
        protocol="tcp"  # specify protocol UDP or TCP
        template="StandardSyslogFormat"  # specifies the template to use above

        queue.SpoolDirectory="/var/spool/rsyslog"
        queue.FileName="remote"
        queue.MaxDiskSpace="1g"
        queue.SaveOnShutdown="on"
        queue.Type="LinkedList"
        ResendLastMSGOnReconnect="on"
        )
        stop
}

# define input files forwardSysLogs logs to send to the rsyslog server
# and apply ruleset "forwardSysLogs" 
# in /bspool/manager

input(type="imfile" ruleset="forwardSysLogs" Tag="ConnLog" File="/bspool/manager/conn.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="ConnLongLog" File="/bspool/manager/conn_long.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="DNS" File="/bspool/manager/dns.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="Files" File="/bspool/manager/files.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="HeartBeat" File="/bspool/manager/heartbeat.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="NTP" File="/bspool/manager/ntp.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="OSCP" File="/bspool/manager/oscp.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="SSL" File="/bspool/manager/ssl.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="StdErr" File="/bspool/manager/stderr.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="StdOut" File="/bspool/manager/stdout.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="HTTP" File="/bspool/manager/http.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="Notice" File="/bspool/manager/notice.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="Weird" File="/bspool/manager/weird.log")

# define input files forwardSysLogs logs to send to the rsyslog server
# and apply ruleset "forwardSysLogs" 
# in /alog

input(type="imfile" ruleset="forwardSysLogs" Tag="ACL-Alarm" File="/alog/acl-alarm.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="ACL-Audit" File="/alog/acl-audit.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="DNS-Masq" File="/alog/dnsmasq-acl.log")

# define input files forwardSysLogs logs to send to the rsyslog server
# and apply ruleset "forwardSysLogs" 
# in /alog/firewalla

input(type="imfile" ruleset="forwardSysLogs" Tag="FireApi" File="/alog/firewalla/FireApi.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="FireKick" File="/alog/firewalla/FireKick.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="FireMain" File="/alog/firewalla/FireMain.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="FireMon" File="/alog/firewalla/FireMon.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="FireRouter" File="/alog/firewalla/FireRouter.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="Trace" File="/alog/firewalla/Trace.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="CleanLog" File="/alog/firewalla/clean_log.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="Firelog" File="/alog/firewalla/firelog.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="Node" File="/alog/firewalla/node.log")
input(type="imfile" ruleset="forwardSysLogs" Tag="SyncTime" File="/alog/firewalla/sync_time.log")

# Sending all other Syslog logs to Server (Synology) 
# @@IP is for TCP
# @IP is for UDP
*.* @@172.16.2.20:514
```


#########################
**Modifying the config**
########################

**Be sure to change the following attributes in the new config file before pasting via `VI` into the syslog conf file:**
```
        target="172.16.2.20"  # set your Synology Syslog Server NAS IP
        port="514"  # Specify port number
        protocol="tcp"  # specify protocol UDP or TCP
```
**AND**
```
# Sending all other Syslog logs to Server (Synology) 
# @@IP is for TCP
# @IP is for UDP
*.* @@172.16.2.20:514
```


###############################
**Setting up Syslog on Firewalla to send to Synology**
###############################

**To collect the all of the important Firewalla connection logs, etc., doing the following:**

- Go to the following directory by *running the following command*:
`cd /etc/rsyslog.d`

- Create the the Syslog conf file :
`sudo touch /etc/rsyslog.d/09-externalserver.conf`
`sudo vi /etc/rsyslog.d/09-externalserver.conf`

- Next we need to open the file in order to paste our new configs by running the following command:
**```sudo vi 09-externalserver.conf```**

- *Press* the letter `i` on your keyboard for insert

- *Copy the configs* and paste them into the file by `right clicking` (this is how you paste using VIM)

- Once the configs are copied then `press escape` then type `:wq!` on your kyboard and hit `enter`

- Now *run the following command* to restart the syslog engine:
**```sudo systemctl restart rsyslog```**


**That's it! You should now be grabbing all of the important Firewalla logs!**


**######################**
**Synology Log Center Results**
**######################**

Here is a screenshot of my `Synology Log Center`:

![image](https://gist.github.com/assets/14807507/2d91aa25-0aa5-4447-954b-49e9a1515aeb)


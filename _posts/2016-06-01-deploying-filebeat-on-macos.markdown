---
layout: post
title: Deploying Filebeat on macOS
date: 2016-06-01 17:11 -0700
categories: mac
tags: apple,mac,macos,filebeat,osquery
---

Got a few questions about the way I've deployed Filebeat to transport OSQuery logs over the past few days, so I thought I'd do a quick writeup about it.
<!-- more -->
There are a few components to this.
- Filebeat executable (the Darwin version)
- `filebeat.yml` (config file to tell Filebeat where to deliver the logs to)
- Certificates (for TLS transport, placed in your location of choice)
- `com.elastic.filebeat.plist` (Launchd task to daemonize Filebeat)

Easy enough, really. 

The certs are generated with a basic openssl command. 

`openssl req -subj '/CN=<your.domain.name>/' -x509 -batch -nodes -newkey rsa:2048 -keyout filebeat.key -out filebeat.crt -days 3650`

The key sits with the cert on the Logstash server, while the cert alone sits on the endpoints. 

Here's what my `filebeat.yml` file looks like:

```yaml
################### Filebeat Configuration Example #########################

############################# Filebeat ######################################
filebeat:
  # List of prospectors to fetch data.
  prospectors:
    # Each - is a prospector. Below are the prospector specific configurations
    -
      # Paths that should be crawled and fetched. Glob based paths.
      # To fetch all ".log" files from a specific level of subdirectories
      # /var/log/*/*.log can be used.
      # For each file found under this path, a harvester is started.
      # Make sure not file is defined twice as this can lead to unexpected behaviour.
      paths:
        - /var/log/osquery/osqueryd.results.log
      document_type: json
      fields:
          type: osquery_json
          codec: json
      # - c:\programdata\elasticsearch\logs\*

      # Configure the file encoding for reading files with international characters
      # following the W3C recommendation for HTML5 (http://www.w3.org/TR/encoding).
      # Some sample encodings:
      #   plain, utf-8, utf-16be-bom, utf-16be, utf-16le, big5, gb18030, gbk,
      #    hz-gb-2312, euc-kr, euc-jp, iso-2022-jp, shift-jis, ...
      #encoding: plain

      # Type of the files. Based on this the way the file is read is decided.
      # The different types cannot be mixed in one prospector
      #
      # Possible options are:
      # * log: Reads every line of the log file (default)
      # * stdin: Reads the standard in
      input_type: log

      # Optional additional fields. These field can be freely picked
      # to add additional information to the crawled log files for filtering
      #fields:
      #  level: debug
      #  review: 1

      # Set to true to store the additional fields as top level fields instead
      # of under the "fields" sub-dictionary. In case of name conflicts with the
      # fields added by Filebeat itself, the custom fields overwrite the default
      # fields.
      #fields_under_root: false

      # Ignore files which were modified more then the defined timespan in the past
      # Time strings like 2h (2 hours), 5m (5 minutes) can be used.
      #ignore_older: 24h

      # Type to be published in the 'type' field. For Elasticsearch output,
      # the type defines the document type these entries should be stored
      # in. Default: log
      #document_type: log

      # Scan frequency in seconds.
      # How often these files should be checked for changes. In case it is set
      # to 0s, it is done as often as possible. Default: 10s
      #scan_frequency: 10s

      # Defines the buffer size every harvester uses when fetching the file
      #harvester_buffer_size: 16384

      # Setting tail_files to true means filebeat starts readding new files at the end
      # instead of the beginning. If this is used in combination with log rotation
      # this can mean that the first entries of a new file are skipped.
      #tail_files: false

      # Backoff values define how agressively filebeat crawls new files for updates
      # The default values can be used in most cases. Backoff defines how long it is waited
      # to check a file again after EOF is reached. Default is 1s which means the file
      # is checked every second if new lines were added. This leads to a near real time crawling.
      # Every time a new line appears, backoff is reset to the initial value.
      #backoff: 1s

      # Max backoff defines what the maximum backoff time is. After having backed off multiple times
      # from checking the files, the waiting time will never exceed max_backoff idenependent of the
      # backoff factor. Having it set to 10s means in the worst case a new line can be added to a log
      # file after having backed off multiple times, it takes a maximum of 10s to read the new line
      #max_backoff: 10s

      # The backoff factor defines how fast the algorithm backs off. The bigger the backoff factor,
      # the faster the max_backoff value is reached. If this value is set to 1, no backoff will happen.
      # The backoff value will be multiplied each time with the backoff_factor until max_backoff is reached
      #backoff_factor: 2

      # Defines the time on how long the harvester will wait for a line to be completed.
      # Sometimes a lines it not completely written when checked by filebeat. Filebeat
      # will wait for the time defined below so the system can complete the line.
      # In case the line is not completed in this time, the line will be skipped.
      #partial_line_waiting: 5s

      # This option closes a file, as soon as the file name changes.
      # This config option is recommended on windows only. Filebeat keeps the files it's reading open. This can cause
      # issues when the file is removed, as the file will not be fully removed until also Filebeat closes
      # the reading. Filebeat closes the file handler after ignore_older. During this time no new file with the
      # same name can be created. Turning this feature on the other hand can lead to loss of data
      # on rotate files. It can happen that after file rotation the beginning of the new
      # file is skipped, as the reading starts at the end. We recommend to leave this option on false
      # but lower the ignore_older value to release files faster.
      #force_close_files: false

    #-
    #  paths:
    #    - /var/log/apache/*.log
    #  type: log
    #
    #  # Ignore files which are older then 24 hours
    #  ignore_older: 24h
    #
    #  # Additional fields which can be freely defined
    #  fields:
    #    type: apache
    #    server: localhost
    #-
    #  type: stdin
    #  paths:
    #    - "-"

  # General filebeat configuration options
  #
  # Event count spool threshold - forces network flush if exceeded
  #spool_size: 1024

  # Defines how often the spooler is flushed. After idle_timeout the spooler is
  # Flush even though spool_size is not reached.
  #idle_timeout: 5s

  # Name of the registry file. Per default it is put in the current working
  # directory. In case the working directory is changed after when running
  # filebeat again, indexing starts from the beginning again.
  #registry_file: .filebeat

  # Full Path to directory with additional prospector configuration files. Each file must end with .yml
  # These config files must have the full filebeat config part inside, but only
  # the prospector part is processed. All global options like spool_size are ignored.
  # The config_dir MUST point to a different directory then where the main filebeat config file is in.
  #config_dir:


###############################################################################
############################# Libbeat Config ##################################
# Base config file used by all other beats for using libbeat features

############################# Output ##########################################

# Configure what outputs to use when sending the data collected by the beat.
# Multiple outputs may be used.
output:

  ### Elasticsearch as output
  #elasticsearch:
    #enabled: false
    # Array of hosts to connect to.
    # Scheme and port can be left out and will be set to the default (http and 9200)
    # In case you specify and additional path, the scheme is required: http://localhost:9200/path
    # IPv6 addresses should always be defined as: https://[2001:db8::1]:9200
    #hosts: ["localhost:9200"]

    # Optional protocol and basic auth credentials. These are deprecated.
    #protocol: "https" s
    #username: "admin"
    #password: "s3cr3t"

    # Number of workers per Elasticsearch host.
    #worker: 1

    # Optional index name. The default is "filebeat" and generates
    # [filebeat-]YYYY.MM.DD keys.
    #index: "filebeat"

    # Optional HTTP Path
    #path: "/elasticsearch"

    # The number of times a particular Elasticsearch index operation is attempted. If
    # the indexing operation doesn't succeed after this many retries, the events are
    # dropped. The default is 3.
    #max_retries: 3

    # The maximum number of events to bulk in a single Elasticsearch bulk API index request.
    # The default is 50.
    #bulk_max_size: 50

    # Configure http request timeout before failing an request to Elasticsearch.
    #timeout: 90

    # The number of seconds to wait for new events between two bulk API index requests.
    # If `bulk_max_size` is reached before this interval expires, addition bulk index
    # requests are made.
    #flush_interval: 1

    # Boolean that sets if the topology is kept in Elasticsearch. The default is
    # false. This option makes sense only for Packetbeat.
    #save_topology: false

    # The time to live in seconds for the topology information that is stored in
    # Elasticsearch. The default is 15 seconds.
    #topology_expire: 15

    # tls configuration. By default is off.
    #tls:
      # List of root certificates for HTTPS server verifications
      #certificate_authorities: ["/etc/pki/root/ca.pem"]

      # Certificate for TLS client authentication
      #certificate: "/etc/pki/client/cert.pem"

      # Client Certificate Key
      #certificate_key: "/etc/pki/client/cert.key"

      # Controls whether the client verifies server certificates and host name.
      # If insecure is set to true, all server host names and certificates will be
      # accepted. In this mode TLS based connections are susceptible to
      # man-in-the-middle attacks. Use only for testing.
      #insecure: true

      # Configure cipher suites to be used for TLS connections
      #cipher_suites: []

      # Configure curve types for ECDHE based cipher suites
      #curve_types: []

      # Configure minimum TLS version allowed for connection to logstash
      #min_version: 1.0

      # Configure maximum TLS version allowed for connection to logstash
      #max_version: 1.2


  ### Logstash as output
  logstash:
    # The Logstash hosts
    hosts: ["<your-host-name:port>"]

    # Number of workers per Logstash host.
    #worker: 1

    # Optional load balance the events between the Logstash hosts
    #loadbalance: true

    # Optional index name. The default index name depends on the each beat.
    # For Packetbeat, the default is set to packetbeat, for Topbeat
    # top topbeat and for Filebeat to filebeat.
    #index: filebeat

    # Optional TLS. By default is off.
    tls:
      # List of root certificates for HTTPS server verifications
      certificate_authorities: ["/etc/pki/tls/certs/filebeat.crt"]

      # Certificate for TLS client authentication
      #certificate: "/etc/pki/client/cert.pem"

      # Client Certificate Key
      #certificate_key: "/etc/pki/client/cert.key"

      # Controls whether the client verifies server certificates and host name.
      # If insecure is set to true, all server host names and certificates will be
      # accepted. In this mode TLS based connections are susceptible to
      # man-in-the-middle attacks. Use only for testing.
      #insecure: true

      # Configure cipher suites to be used for TLS connections
      #cipher_suites: []

      # Configure curve types for ECDHE based cipher suites
      #curve_types: []


  ### File as output
  #file:
    # Path to the directory where to save the generated files. The option is mandatory.
    #path: "/tmp/filebeat"

    # Name of the generated files. The default is `filebeat` and it generates files: `filebeat`, `filebeat.1`, `filebeat.2`, etc.
    #filename: filebeat

    # Maximum size in kilobytes of each file. When this size is reached, the files are
    # rotated. The default value is 10 MB.
    #rotate_every_kb: 10000

    # Maximum number of files under path. When this number of files is reached, the
    # oldest file is deleted and the rest are shifted from last to first. The default
    # is 7 files.
    #number_of_files: 7


  ### Console output
  # console:
    # Pretty print json event
    #pretty: false


############################# Shipper #########################################

shipper:
  # The name of the shipper that publishes the network data. It can be used to group
  # all the transactions sent by a single shipper in the web interface.
  # If this options is not defined, the hostname is used.
  #name:

  # The tags of the shipper are included in their own field with each
  # transaction published. Tags make it easy to group servers by different
  # logical properties.
  #tags: ["service-X", "web-tier"]

  # Uncomment the following if you want to ignore transactions created
  # by the server on which the shipper is installed. This option is useful
  # to remove duplicates if shippers are installed on multiple servers.
  #ignore_outgoing: true

  # How often (in seconds) shippers are publishing their IPs to the topology map.
  # The default is 10 seconds.
  #refresh_topology_freq: 10

  # Expiration time (in seconds) of the IPs published by a shipper to the topology map.
  # All the IPs will be deleted afterwards. Note, that the value must be higher than
  # refresh_topology_freq. The default is 15 seconds.
  #topology_expire: 15

  # Configure local GeoIP database support.
  # If no paths are not configured geoip is disabled.
  #geoip:
    #paths:
    #  - "/usr/share/GeoIP/GeoLiteCity.dat"
    #  - "/usr/local/var/GeoIP/GeoLiteCity.dat"


############################# Logging #########################################

# There are three options for the log ouput: syslog, file, stderr.
# Under Windos systems, the log files are per default sent to the file output,
# under all other system per default to syslog.
logging:

  # Send all logging output to syslog. On Windows default is false, otherwise
  # default is true.
  #to_syslog: true

  # Write all logging output to files. Beats automatically rotate files if rotateeverybytes
  # limit is reached.
  to_files: true

  # To enable logging to files, to_files option has to be set to true
  files:
    # The directory where the log files will written to.
    path: /var/log/filebeat

    # The name of the files where the logs are written to.
    name: filebeat.log

    # Configure log file size limit. If limit is reached, log file will be
    # automatically rotated
    rotateeverybytes: 10485760 # = 10MB

    # Number of rotated log files to keep. Oldest files will be deleted first.
    keepfiles: 7

  # Enable debug output for selected components. To enable all selectors use ["*"]
  # Other available selectors are beat, publish, service
  # Multiple selectors can be chained.
  #selectors: [ ]

  # Sets log level. The default log level is error.
  # Available log levels are: critical, error, warning, info, debug
  level: info
```

There's a lot in that file above; I used the sample config file included with the daemon and edited what I needed, leaving the rest commented out.

*A note about the `document_type` and `codec` fields; these were educated guesses based on the way the documentation for Logstash (and some Google searching) recommended for the input type.*

Finally, there's the launchd plist. I can't recall how I did this; I strongly suspect it's a build on someone else's work as I don't recall writing this from scratch, but it's small enough that it's not at all hard.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Disabled</key>
    <false/>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/sbin</string>
    </dict>
    <key>KeepAlive</key>
    <dict>
        <key>SuccessfulExit</key>
        <true/>
    </dict>
    <key>Label</key>
    <string>com.elastic.filebeat</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/filebeat</string>
        <string>-c</string>
        <string>/etc/filebeat/filebeat.yml</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

The important parts for me are the `ProgramArguments` portion. 

```xml
<key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/filebeat</string>
        <string>-c</string>
        <string>/etc/filebeat/filebeat.yml</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
```
That's the part that tells Filebeat where to look for the config file and to run at load time. 

I deploy this with a .pkg created in Packages and a pre- and post-install script to do a few things.

```bash
filebeatRunning=`sudo launchctl list | grep com.elastic.filebeat`

if [ ! -z "$filebeatRunning" ]; then
  LogScript "Filebeat is running. Stopping in order to install new or updated version."
    sudo launchctl unload /Library/LaunchDaemons/com.elastic.filebeat.plist
  else
        LogScript "Filebeat does not appear to be running. Proceeding with install."
fi
```

The `LogScript` part is from a functions library I use and is inspired by [Rich Trouton's][1] CasperCheck script, which gave me the idea for it.

My post-install script also looks for and unloads the old logstash-forwarder launchdaemon; I'd created this in order to pull logstash-forwarder from some of the early pilot Macs, before logstash-forwarder was replaced by Filebeat.

```bash
launchdRunning=`sudo launchctl list | grep com.elastic.logstash-forwarder`
filebeatRunning=`sudo launchctl list | grep com.elastic.filebeat`

if [ ! -z "$launchdRunning" ]; then
  LogScript "Logstash-Forwarder existed on this Mac, unloading extension, deleting."
  /bin/launchctl unload -w /Library/LaunchDaemons/com.elastic.logstash-forwarder.plist
    else
        LogScript "No Logstash-Forwarder LaunchDaemon running on this Mac."
fi

if [[ -f /Library/LaunchDaemons/com.elastic.logstash-forwarder.plist ]]; then
  LogScript "Logstash-Forwader found in /Library/LaunchDaemons, erasing."
  rm -rf /Library/LaunchDaemons/com.elastic.logstash-forwarder.plist
else
  LogScript "No Logstash-Forwader found in /Library/LaunchDaemons."
fi


/bin/launchctl load /Library/LaunchDaemons/com.elastic.filebeat.plist
```

Again, the LogScript stuff won't work for you if you don't have a functions library to call it from.

On the server-side, I have the following input file set up in `/etc/logstash/conf.d/01-beats-input.conf`.

```
input {
  beats {
    port => <your-port-here>
    ssl => true
    type => "JSON"
    ssl_certificate => "<path-to-cert>"
    ssl_key => "<path-to-key>"
    congestion_threshold => 1000
  }
}
```

I set a higher congestion threshold to allow for hundreds of endpoints. As I cluster Elasticsearch and allow for better ingestion, I'll up that congestion threshold.

Also, SSL is disabled by default, so make sure you have the SSL arguments in there. Security will give you a nasty look (and maybe fire your ass?) if you deploy with in-the-clear transport. Plus, it's just rude not to encrypt.

That's about it. I have \~600+ clients now reporting into my ELK stack, with the final number being around 1400 when we move to production. 

Happy to field any questions; @sohail on Twitter, the MacAdmins, OSQuery, or ITThinkTank Slack teams. 

[1]:	http://derflounder.wordpress.com
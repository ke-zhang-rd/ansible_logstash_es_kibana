# Ansible -> logstash -> elasticsearch -> kibana

To make this pipeline possible, each node need properly config.
### logstash
Edit `logstash.config` in `/usr/local/etc/logstash`
```
input {
    tcp {
        port => 5000
        codec => json
    }
}
output {
    elasticsearch {
        hosts => ["localhost:9200"]
        index => "idk"
    }
    stdout {
        codec => json
    }
    file {
        path => '~/ansible_logstash_es_kibana/test.log'
        codec => line { format => "custom format: %{message}"}
    }
}
```
If you are tring to use `brew services start logstash`, the services 
will be start by a `homebrew.mxcl.logstash.plist` file of homebrew. 
You need make sure config argument setting has been added to 
`/usr/local/Cellar/logstash/6.6.1/homebrew.mxcl.logstash.plist`. 
The default file dosen't add it. **IT WON'T WORK** if you edit 
`/Users/kz2249/Library/LaunchAgents/homebrew.mxcl.logstash.plist` 
which is ```brew services list``` show you. The former one will overwrite 
latter one

Add lines below to your `homebrew.mxcl.logstash.plist`
```
      <string>--config</string>
      <string>/usr/local/etc/logstash/logstash.config</string>
```
Your file should end up something like:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>KeepAlive</key>
    <false/>
    <key>Label</key>
    <string>homebrew.mxcl.logstash</string>
    <key>ProgramArguments</key>
    <array>
      <string>/usr/local/opt/logstash/bin/logstash</string>
      <string>--config</string>
      <string>/usr/local/etc/logstash/logstash.config</string>
    </array>
    <key>EnvironmentVariables</key>
    <dict>
    </dict>
    <key>RunAtLoad</key>
    <true/>
    <key>WorkingDirectory</key>
    <string>/usr/local/var</string>
    <key>StandardErrorPath</key>
    <string>/usr/local/var/log/logstash.log</string>
    <key>StandardOutPath</key>
    <string>/usr/local/var/log/logstash.log</string>
  </dict>
</plist>
```

Finally start `logstash` service by:
```bash
$ brew services start logstash
```
### elasticsearch
Edit `elasticsearch.yml` in `/usr/local/etc/elasticsearch`.
Default is good to go in this case.
Then start elasticsearch service by:
```bash
$ brew services start elasticsearch
```
### kibana
Edit `kibana.yml` in `/usr/local/etc/kibana`
```
server.port: 5601
elasticsearch.hosts: ["http://localhost:9200"]
```
Then start kibana service by:
```bash
$ brew services start kibana
```

### Ansible callback plugin
You need create a `.py` file which include class `CallbackModule` whose method will be
called at the end of anisble steps.

### Ansible
Add line below to your `ansible.cfg `
```
callback_whitelist = logstash
callback_plugins   = ~/ansible_logstash_es_kibana/callback
[callback_logstash]
server = localhost
port = 5000
pre_command = git rev-parse HEAD
type = ansible
```
Run your ansible
```bash
$ ansible-play helloworld.yml
```
Enjoy log at `http://localhost:5601`

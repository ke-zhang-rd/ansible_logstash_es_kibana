This test need to be configured by several .config file



ansible-play helloworld.yml

configuration file: ansible.cfg
location: whereever your ansible.cfg is



brew services start logstash

configeration file: logstash.config
default location: /usr/local/etc/logstash

If you use brew services start logstash. The services will be
start by a .plist file of homebrew. Make sure config file setting
is set correctly in 
/usr/local/Cellar/logstash/6.6.1/homebrew.mxcl.logstash.plist
The default file dosen't start severces with config file. Yor need
mannully add it.

IT WON'T WORK if you edit
/Users/kz2249/Library/LaunchAgents/homebrew.mxcl.logstash.plist
which is brew services list show you. The former one will overwrite
latter one



brew services start elasticsearch

configeration file: elasticsearch.yml
default location: /usr/local/etc/elasticsearch



brew services start kibana

configeration file: kibana.yml
default location: /usr/local/etc/kibana

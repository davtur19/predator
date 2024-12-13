# predator
Predator IDS and Proxy solution

## Main concepts
* NIDS = Network Intrusion Detection System, it is a tool used to identify a threat inside a network basing on a set of rules user-defined
* Proxy = It is a tool used to provide a single Internet access point with SSL termination in order to check encrypted traffic

## Overview
Predator has been designed to reach following needs:
* Easy rules integration
* Modularity
* Multithreaded, usable and fast
* Syslog log redirection to be analyzed by third parties (SIEM)
Configuration is reported in config.py

## Rules integration
As default, Predator is able to ingest rules generated by my [Anubi Signatures project](https://github.com/kavat/anubi-signatures)
It allows rules based on:
* IP (file with suffix _ip.json)
* TOR nodes (tor_nodes.list)
* domain (file with suffix _fqdn.json)
* payload (patterns_tcp_udp.json)
Format is similar for IP and domains, such below:
* Simple file JSON with IP as key and description CTI as value, such
```
{"1.10.146.175":"misp","1.117.62.81":"misp","1.14.194.206":"misp","1.14.206.72":"misp", … }
```
name will be similar to anubi_YYYY_MM_ip.json
* Simple file JSON with domain as key and description CTI as value, such
name will be similar to anubi_YYYY_MM_fqdn.json
```
{"0-lx.2037673.xyz":"misp","0-lx.49746487.xyz":"misp","00lx.13265926.xyz":“misp", …}
```
* Simple file with Array with patterns as values, such
```
["PATTERN_1", "PATTERN_2", "PATTERN_3", …] 
```
name is static and it is patterns_tcp_udp.json

## Modules
### Layer 4 source/destination control and Layer 7 pattern match
* Based on set of rules previously described
* Check for incoming/outcoming connection from/to malicious IP
* Check for incoming/outcoming connection with pattern monitored
Default in config.py is True
```
IDS = True
```
### API
API handles interaction among users and system (such updating rules without restart tool)
Features available are:
* help, listing features available
```
curl -XPOST http://127.0.0.1:10000/api -H "content-type: application/json" -d "{\"func\":\"help\"}"
```
* createca, creating Certification Authority certificate usable to intercept SSL traffic in proxy module
```
curl -XPOST http://127.0.0.1:10000/api -H "content-type: application/json" -d "{\"func\":\"createca\"}"
```
* loadjson, loading of new rules saved in rule config path without restarting tool; example with JSON file named test.json based in /opt/predator/conf/json as default
```
curl -XPOST http://127.0.0.1:10000/api -H "content-type: application/json" -d "{\"func\":\"loadjson\",\"file_json\":\"test.json\"}"
```
Default in config.py is True
```
API = True
```
and set dummy address to 127.0.0.1 on TCP/10000
```
MANAGEMENT_HOST = "127.0.0.1"
MANAGEMENT_PORT = 10000
```
User is helped by a simply webui in order to interact with Predator (as default console is reacheable at http://127.0.0.1:10000)

![Predator WebUI](screenshots/api.png)
### Proxy
MITM for SSL termination and possibility to pattern checking based on rules previously described; default in config.py is False
```
PROXY = False
```
To generate CA in order to perform MITM following commands are required:
```
curl -XPOST http://127.0.0.1:10000/api -H "content-type: application/json" -d "{\"func\":\"createca\"}"
curl -x http://127.0.0.1:7777 http://predator.fuck
```
Default configuration in config.py set proxy address to 127.0.0.1 on TCP/7777
```
PROXY_HOST = "127.0.0.1"
PROXY_PORT = 7777
```
After CA has been generated, proxy can be set in order to perform traffic inspection
### Dummy
Traffic decryption through Proxy module replication to internal network for third parties analysis (such Suricata); default in config.py is False
```
DUMMY = False
```
and set dummy address to 127.0.0.1 on TCP/9999
```
DUMMY_HOST = "127.0.0.1"
DUMMY_PORT = 9999
```
## Custom tags
* On HTTP connection, Host field in header is isolated if available and propagated in log visibility in order to whitelist if host is safe
* On HTTPS connection with no SSL termination, SNI field in TLS extensions is isolated if available and propagated in log visibility in order to whitelist if host is safe

## config.py
Most imporant settings in configuration file are reported and explained below:
* general paths
```
PATH_LOGGER_PREDATOR_MAIN = '/var/log/predator.log'
PATH_LOGGER_PREDATOR_DNS = '/var/log/predator_dns.log'
PATH_LOGGER_PREDATOR_INTELLIGENCE = '/var/log/predator_intelligence.log'
PATH_LOGGER_PREDATOR_THREATS = '/var/log/predator_threats.log'
PATH_LOGGER_PREDATOR_MANAGEMENT = '/var/log/predator_management.log'
PATH_LOGGER_PREDATOR_SNIFFERS = '/var/log/predator_sniffers.log'
PATH_LOGGER_PREDATOR_L7 = '/var/log/predator_l7.log'
PATH_LOGGER_PREDATOR_PROXY = '/var/log/predator_proxy.log'
PATH_LOGGER_PREDATOR_DUMMY = '/var/log/predator_dummy.log'
PATH_LOGGER_PREDATOR_MASTER_EXCEPTIONS = '/var/log/predator_boom.log'
PATH_JSON = "/opt/predator/conf/json/"
```
* IP or CIDR list to control as traffic source or destination
```
CIDRS = ['___XXX.XXX.XXX.XXX/YY___']
```
* Proxy CA
```
CA_KEY = "/opt/predator/certs/ca.key"
CA_CRT = "/opt/predator/certs/ca.crt"
CERT_KEY = "/opt/predator/certs/cert.key"
CERT_DIR = "/opt/predator/certs"
CA_KEY_SIZE = 2048
CERT_KEY_SIZE = 2048
LINK_DOWNLOAD_CA = "http://predator.fuck/"
```
* Proxy IP interface
```
PROXY_HOST = "___LOCAL_IP___"
PROXY_PORT = 7777
```
* IDS interface to monitor
```
NICS_TO_SNIFF = ["___LOCAL_ETH___"]
```
* Dummy interface
```
DUMMY_HOST = "___LOCAL_IP___"
DUMMY_PORT = 9999
```
* Modules enabling/disabling control
```
IDS = True
PROXY = False
SEND_TO_DUMMY = False
SEND_TO_SYSLOG = False
```

## Start
After installing dependencies
```
pip3 install scapy
pip3 install flask
```
you can proceed launching tool; Predator is easy to start and you can do it with
```
python3 ./predator.py
```
or
```
./predator.sh start
```
Root privileges are required.

## Screenshots
* Status
![Status](screenshots/status.png)
* Threats identified
![Threats](screenshots/threats.png)

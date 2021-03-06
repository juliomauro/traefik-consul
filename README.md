# Træfik with a consul using key/value

With these settings you can start a *load balancer* using **traefik** and **consul** as the *configuration server*.

Copy this repository into the "/ opt /" directory to be necessary to change the directories in the configuration scripts. (if you feel free to do so without any problem). You need to change some configuration files for the server to come up in your environment. 

## Consul files

#### config.json
The first file to change is the *config.json*, inside the **consul** folder.

    {
        "bootstrap": true,
        "server": true,
        "datacenter": "CONSUL_NAME",
        "data_dir": "/opt/traefik-consul/consul/data",
        "encrypt": "n8UUwpbNspNmVo3uAH01uq==",
        "log_level": "INFO",
        "enable_syslog": true,
        "node_name": "NODE_NAME01",
        "addresses": {"http": "IP_CONSUL_SERVER" },
        "bind_addr": "IP_CONSUL_SERVER",
        "ui": true
    }

 - Change the content of the ***datacenter*** item to the name you want to
   give to the consul server.
 - Change the contents of the ***encrypt*** item to the generated key by the
   command "`consul keygen`".
 - Change the contents of the ***node_name*** item if you want to use the
   clustered consul in the future.
 - Change the contents of the ***addresses*** and ***bind_addr*** items to the
   ip of your network interface.

#### master.json
The master.json file is also in the consul folder.

    {
    "acl_datacenter":"CONSUL_NAME",
    "acl_default_policy":"allow",
    "acl_down_policy":"allow",
    "acl_master_token":"398073a8-5091-4d9c-871a-bbbeb030d1f5"
    }

Change only the contents of the ***acl_datacenter*** item with the same value entered in  ***datacenter*** item of the ***config.json*** file

## Traefik files

#### traefik_config.toml
In this file, you only need to change the contents of the ***endpoint*** key. Change to the address of the consul server.

    [web]
    	address = ":8080"
    	
    [api]
    	entryPoint = "traefik"
    	dashboard = true
    	debug = true
    	
    [ping]
    	entryPoint = "traefik"
    	
    [metrics]
    	[metrics.statistics]
    		recentErrors = 10
    		
    [consul]
    	endpoint = "IP_CONSUL_SERVER:8500"
    	watch = true
    	prefix = "traefik"
    	
    [traefikLog]
    	filePath = "/opt/traefik-consul/traefik/log/traefik.log"
    	
    [accessLog]
    	filePath = "/opt/traefik-consul/traefik/log/access.log"

## Starting servers
#### Consul

    /opt/traefik-consul/bin/consul agent -config-dir /opt/traefik-consul/consul/

#### Traefik

    /opt/traefik-consul/bin/traefik -c /opt/traefik-consul/traefik_config.toml

## Balancing Information

We need to define what the sites and their respective backend servers will be to configure the balancer.


##### Host: site01.oruam.cloud
|IP      |TCP Port  |Health check Path |Health check Interval 
|--------|----------|------------------|---------------------|
|1.1.1.1 |80        |/_cat/health      |10s                  |
|1.1.1.2 |80        |/_cat/health      |10s                  |
|1.1.1.3 |80        |/_cat/health      |10s                  |
|1.1.1.4 |80        |/_cat/health      |10s                  |

##### Host: site02.oruam.cloud
|IP      |TCP Port  |Health check Path |Health check Interval 
|--------|----------|------------------|---------------------|
|1.1.1.5 |80        |/_cat/health      |10s                  |
|1.1.1.6 |80        |/_cat/health      |10s                  |
|1.1.1.7 |80        |/_cat/health      |10s                  |
|1.1.1.8 |80        |/_cat/health      |10s                  |

### Adding informations to consul server

**Host site01**

*Backend servers*

    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site01/servers/01/url http://1.1.1.1:80
    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site01/servers/02/url http://1.1.1.2:80
    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site01/servers/03/url http://1.1.1.3:80
    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site01/servers/04/url http://1.1.1.4:80

*Backend heath check*

    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site01/healthcheck/interval 10s
    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site01/healthcheck/path /_cat/health

*Frontend*

    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/frontends/site01/backend site01
    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/frontends/site01/routes/url/rule Host:site01.oruam.cloud

**Host site02**

*Backend servers*

    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site02/servers/01/url http://1.1.1.5:80
    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site02/servers/02/url http://1.1.1.6:80
    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site02/servers/03/url http://1.1.1.7:80
    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site02/servers/04/url http://1.1.1.8:80

*Backend heath check*

    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site02/healthcheck/interval 10s
    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/backends/site02/healthcheck/path /_cat/health

*Frontend*

    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/frontends/site02/backend site02
    consul kv put -http-addr=IP_CONSUL_SERVER:8500 traefik/frontends/site02/routes/url/rule Host:site02.oruam.cloud


*Check infos into consul*

    consul kv get -keys -http-addr=IP_CONSUL_SERVER:8500

    consul kv get -recurse -http-addr=IP_CONSUL_SERVER:8500

    consul kv get -recurse -detailed -http-addr=IP_CONSUL_SERVER:8500


*Check infos Web interface*

**Consul**
http://IP_CONSUL_SERVER:8500
![Screenshot](traefik-SS.png)

**Traefik**
http://IP_CONSUL_SERVER:8080]
![Screenshot](consul-SS.png)

**For more information about the products, visit:**

https://www.consul.io/docs/

https://docs.traefik.io/

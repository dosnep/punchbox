# Punchplatform validation

Punchplatform validation could be use to daily check your platform health or to check a 
specific critic point

Punchplatform team provides and uses this tool to validate Punchplatform releases
and to maintain stabilily of Punch repository 

Files in `templates` folder combined with Punchplatform standalone examples
are the resources necessary for Punchplatform team validation

However, you can use your own channels and create your own validation templates to respond
to your specific use case 

## Punchplatform integration 

### Manual integration 

Once your deployment is successful you can check your platform health. 
Connect to your operator node, you will find a shell in `pp-conf/check_platform.sh`
Global execution takes around 15 minutes. 

To execute it: 
```sh
ssh vagrant@server_operator
./pp-conf/check_platform.sh
```

This automatic test checks if aggregation channel works and send result in a specific slack channel.

**Watchout** : You must set 'SLACK_WEBHOOK' environment variable with a valid slack webhook to be able to publish results  

### Automatic integration 

You can also configure an automatic integration on your laptop or in remote servers by calling shell
in `bin` directory in a crontab

Be sure to complete these steps and to adapt shell to your use case : 

With your current user, copy and paste full result of 'env' command at the beggining of the user crontab file :

```sh
crontab -e  
```

Then, under these lines add :

```sh
PUNCHBOX_DIR=/home/punch/workspace/punchbox
PUNCH_DIR=/home/punch/workspace/pp-punch
00 1  * *  * $PUNCHBOX_DIR/punch/validation/bin/local_integration.sh 6.0 > /tmp/punchbox-6.0 2>&1
```

## Punchplatform tests 

This section describes tests done during Punchplatform team validation :

### Aggregation test 

Launch aggregation and apache channels from Punchplatform standalone and then use log injectors 

**Scope covered** : 
  - Put configuration in zookeeper
  - Push some templates to Elasticsearch
  - Import Kibana dashboards 
  - Storm execution 
  - Plan and spark in foreground mode  
  - Log injectors 
  - Elasticsearch 

### Elastalert test : 

Launch elastalert channel to get and publish results from previous tests 

**Scope covered** : 
  - Shiva execution
  - Elastalert 
  - Elasticsearch


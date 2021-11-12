ELK Logging stack for K8s
=========================
These manifests allow for installing [EFK logging stack - Elasticsearch Fluentd Kibana](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes) in a K8s cluster.


Installation
------------
```shell
.
├── elasticsearch.yml
├── elasticsearch_ingress.yml
├── elasticsearch_pv_local_storage.yml
├── fluentd.yml
├── kibana.yml
└── kibana_ingress.yml
```

First set the variables in the K8s manifests;
```shell
for yml_file in `ls -lX *yml |awk '{print $9}'` ; do sed -i 's/$(ns)/efk-logging-ns/g' $yml_file; done
for yml_file in `ls -lX *yml |awk '{print $9}'` ; do sed -i 's/$(es_svc)/es-logging-svc/g' $yml_file; done
for yml_file in `ls -lX *yml |awk '{print $9}'` ; do sed -i 's/$(es_storage_class)/es-local-storage/g' $yml_file; done
for yml_file in `ls -lX *yml |awk '{print $9}'` ; do sed -i 's/$(base_url)/boburciu.privatecloud.com/g' $yml_file; done
```

Then ensure there are no leftover PV and Storage Class artifacts from a previous try;
```shell
 kubectl get pv
 kubectl get storageclasses.storage.k8s.io
 for es_pv in `kubectl get pv | grep es-pv-local-storage | awk {'print $1'}`; do kubectl delete pv $es_pv; done
 for es_storage_class in `kubectl get storageclasses.storage.k8s.io | grep es-local-storage | awk {'print $1'}`; do kubectl delete storageclasses.storage.k8s.io $es_storage_class ; done
```

 `kubectl create ns efk-logging-ns` <br/>
 `kubectl -n efk-logging-ns apply -f .`


Configuring Kibana dashboard
----------------------------
> 1. Click on `Discover` in the left-hand navigation menu of Kibana webUI. This allows you to define the Elasticsearch indices you’d like to explore in Kibana. To learn more, consult [Defining your index patterns by Kibana doc](https://www.elastic.co/guide/en/kibana/current/tutorial-define-index.html);
> 2. Enter `logstash-*` in the text box and click on Next step. In the dropdown, select the `@timestamp` field, and hit `Create index pattern`;
> 3. Now, hit `Discover` in the left hand navigation menu. You should see a histogram graph and some recent log entries


Troubleshooting
---------------

Once installed, check the  Elasticsearch `$(es_svc): es-logging-svc` status from the `kibana` pod for example;
```shell
kubectl -n efk-logging-ns exec -it `kubectl -n efk-logging-ns get pods | grep kibana | awk {'print $1'}` -- curl http://es-logging-svc:9200/_cluster/state?pretty
```
```shell
root@master-node:~/efk-test-final# kubectl -n efk-logging-ns exec -it `kubectl -n efk-logging-ns get pods | grep kibana | awk {'print $1'}` -- curl http://es-logging-svc:9200/_cluster/state?pretty | head -30
{
  "cluster_name" : "k8s-logs",
  "cluster_uuid" : "yBxEP96kTuK8X17RTV2xbg",
  "version" : 284,
  "state_uuid" : "8ciXrhxHQFu5APEVROKSpg",
  "master_node" : "ZzVn5--uQPiyTajDfNp_ag",
  "blocks" : { },
  "nodes" : {
    "ZzVn5--uQPiyTajDfNp_ag" : {
      "name" : "es-cluster-0",
      "ephemeral_id" : "nSTKVno8QuG2C7B51hDHnw",
      "transport_address" : "10.233.106.27:9300",
      "attributes" : {
        "ml.machine_memory" : "16790241280",
        "ml.max_open_jobs" : "20",
        "xpack.installed" : "true"
      }
    },
    "McNGIY8dStu1_ZRW19GBnA" : {
      "name" : "es-cluster-2",
      "ephemeral_id" : "av43hD0wTFGzX-wqDNZWxw",
      "transport_address" : "10.233.106.29:9300",
      "attributes" : {
        "ml.machine_memory" : "16790241280",
        "ml.max_open_jobs" : "20",
        "xpack.installed" : "true"
      }
    },
    "YfVSORN1SPelpSr-RC_M1g" : {
      "name" : "es-cluster-1",
root@master-node:~/efk-test-final#
```


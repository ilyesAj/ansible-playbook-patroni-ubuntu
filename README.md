# Ansible Playbook for bootstraping postgresql with patroni using zookeeper as dcs and HAproxy

======================================


## Requirements

Platform: ubuntu 18.04-lts
## Role Variables
````yml
---
dcs: "zookeeper"
dcs_port: "2181"
postgresql_port: "5432"
patroni_api: 8008
postgresql_cluster_name: "test"
postgresql_major_version: "10"
postgresql_data_dir_base: "/var/lib/postgresql"
patroni_replication_user: "replicator"
patroni_replication_pass: "replicator"
superuser_password: "secret"
superuser_username: "dba"
patroni_postgres_pass: ""
log_directory: "/var/log/postgresql"
#-----------------------
haproxy_port: 5000
haproxy_stats: 7000
#Haproxy sends health-check probes to all patroni nodes it knows
#8008 is the patroni rest api port.
#If node is running as a master, patroni will respond on request ‘GET /’ with http status code 200, in other case it will return 503.
#haproxy sends traffic only to the nodes which responding with 200.
#--------------------------
node_exporter_version: 1.0.1
````
## Deploying ansible-playbook

````
ansible-playbook -i inventory main.yml -vvv
````
to check patroni cluster
````
ansible-playbook -i inventory main.yml -vvv --tags patroni_cluster

````

## License

ilyes Ajroud
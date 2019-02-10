Ansible Playbook for Patroni on Debian
======================================

This Ansible playbook allows to deploy a Patroni cluster using the `patroni`
packages provided by Debian and/or PostgreSQL's `apt.postgresql.org`
repository. Those packages have been integrated into Debian's
`postgresql-common` framework and will look very similar to a regular
stand-alone PostgreSQL install on Debian. For production use it should be
hardened (e.g. using Ansible Vault and encrypted communication between the DCS
server and the Patroni clients).

The default topology consists of one node (`master`) acting as the DCS
(Distributed Consensus Store) server, and three PostgreSQL/Patroni nodes
(`pg1`, `pg2` and `pg3`), see `inventory`.

DCS Server
----------

Patroni requires a DCS for leader election and as configuration store.
Supported DCS server are Etcd, Consul and Zookeeper. If e.g. an Etcd cluster is
already available, then it can be configured either by editing
`templates/dcs.yml` or (if that suffices) by setting the `dcs_server` Ansible
variable to its IP address.

Usage
-----

Assuming password-less SSH access to the four nodes is configured, the playbook
can be run as follows:

```
ansible-playbook -i inventory patroni.yml
```

Supported Versions
------------------

The playbooks have been tested on Debian 9 (stretch), testing (buster) and
unstable (sid), as well as Ubuntu LTS 18.04 (bionic). As the
`apt.postgresql.org` PostgreSQL (and Patroni) packages are used, all supported
PostgreSQL versions can be installed in principle.

Note that Consul is unsupported as DCS on Debian 9 (stretch).
 
Variables
---------

The following useful variables can be set: 

 * `dcs` (`etcd` (default), `consul` or `zookeeper`)
 * `dcs_server` (default: IP of `master` node)
 * `postgresql_cluster_name` (default: test)
 * `postgresql_major_version` (default: 11)
 * `postgresql_data_dir_base` (default: `/var/lib/postgresql`)
 * `patroni_replication_user` (default: `replicator`)
 * `patroni_replication_pass`
 * `patroni_postgres_pass`

Example: `ansible-playbook -i inventory -e dcs=consul patroni.yml`

Running multiple instances of Patroni/PostgreSQL
------------------------------------------------

By default, Patroni uses 8008 as the API REST port. The automatic Patroni
configuration generation will increment this for every additional cluster, so
running

```
ansible-playbook -i inventory -e postgresql_cluster_name=test2 --tags=config pgsql-server.yml
```

will result in a second cluster, `11/test2` using PostgreSQL port 5433 and
Patroni API port 8009.

Rewinding/Recloning outdated former primaries
---------------------------------------------

If a failover has occured and the old leader has additional transactions that
do not allow for a clean change of timeline, `pg_rewind` can be  used in order
to rewind the old leader so that it can reinitiate streaming replication to the
new leader.  For this to happen, the variable `patroni_postgres_pass` needs to
be set in `vars.yml`.  If this is not the case, a full reclone will be done.

In both cases, the former primary's data directory will be gone, so if you
prefer to have the primary stay down for manual inspection instead, you should
comment out (or set to `false`) the parameters `use_pg_rewind`,
`remove_data_directory_on_rewind_failure` and
`remove_data_directory_on_diverged_timelines`.

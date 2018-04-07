# Ansible-Postgresql-Replication-Monitoring

Ansible Monitoring For Postgesql Replication. This monitor script can be used to add monitoring to any PostgreSQL replicated environment and is not limited to just monitor Ansible Tower. However for the purposes of this README.md I will mention Ansible specifics and can elaborate further in the future as use-cases arises.

Monitoring can be hooked in several sources including [Ansible Tower logging aggregators externally](http://docs.ansible.com/ansible-tower/latest/html/administration/logging.html) or [Ansible Tower notification template setup](http://docs.ansible.com/ansible-tower/latest/html/userguide/notifications.html).

# Pre-requisites

* [Ansible Tower Installation](http://docs.ansible.com/ansible-tower/latest/html/quickinstall/prepare.html)
* [samdoran/ansible-role-pgsql-replication](https://github.com/samdoran/ansible-role-pgsql-replication)
* [Ansible Tower Inventory](http://docs.ansible.com/ansible-tower/latest/html/quickinstall/install_script.html#setting-up-the-inventory-file)

## Inventory Example

```ini
[tower]
tower-server1

[database]
tower-pg-master pgsqlrep_role=master

[database_replica]
tower-pg-slave pgsqlrep_role=replica

[all:vars]
admin_password='admin'

pg_host='tower-pg-master'
pg_port='5432'

pg_database='awx'
pg_username='awx'
pg_password='password'

pgsqlrep_password='password'

rabbitmq_port=5672
rabbitmq_vhost=tower
rabbitmq_username=tower
rabbitmq_password='password'
rabbitmq_cookie=cookiemonster

# Needs to be true for fqdns and ip addresses
# rabbitmq_use_long_name=true

# Isolated Tower nodes automatically generate an RSA key for authentication;
# To disable this behavior, set this value to false
# isolated_key_generation=true
```

## Job Execution Example

`ansible-playbook -i inventory pgsql-replicate-monitor.yml`

# Required Variables

Variable Name | Value | Description
--- | --- | ---
pgpass_loc | `~/.pgpass` | Location to store .pgpass file for [automated authentication to PostgreSQL](https://www.postgresql.org/docs/current/static/libpq-pgpass.html).
psql_opts | `-h localhost -d {{ pg_database }} -U {{ pg_username }} -w -t` | Default options passed to [psql command](https://www.postgresql.org/docs/current/static/app-psql.html).
psql_query | `SELECT CASE WHEN pg_last_xlog_receive_location() = pg_last_xlog_replay_location() THEN 0 ELSE EXTRACT (EPOCH FROM now() - pg_last_xact_replay_timestamp()) END AS log_delay;` | Default query to send to psql command to search for time difference.
threshold | `10` | Default to 10 seconds. This can be configurable in a high-latency or high-load environment.
pg_database | `awx` | Ansible Tower defaults it's database name to `awx` in the installer. Configurable via inventory.
pg_username | `awx` | Ansible Tower defaults it's database user name to `awx` in the installer. Configurable via inventory.
pg_password | `password` | This is configureable via inventory for Ansible Tower installation.
pg_port | `5432` | Default port for PosgreSQL database.

# Notes

User assumes all responsibility of PostgreSQL database data.


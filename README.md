# postgres-ha-ansible

## Vagrant setup

### on Mac

```sh
vagrant plugin install vagrant-hostmanager
vagrant up
vagrant reload
vagrant ssh-config >> ~/.ssh/config

ssh-keygen
ssh-copy-id vagrant@192.168.1.201
ssh-copy-id vagrant@192.168.1.202

ssh 192.168.1.201 hostname
ssh 192.168.1.202 hostname

ansible all -i inventories/vagrant/hosts.yml -m ping

vagrant halt
vagrant snapshot save init
vagrant up
```

### on Windows

```sh
vagrant plugin install vagrant-hostmanager
vagrant up
vagrant reload
vagrant ssh-config >> %userprofile%\.ssh\config

ssh pg0

ssh-keygen
ssh-copy-id vagrant@192.168.1.201
ssh-copy-id vagrant@192.168.1.202
ssh 192.168.1.201 hostname
ssh 192.168.1.202 hostname

cd /vagrant
ansible all -i inventories/vagrant/hosts.yml -m ping

exit

vagrant halt
vagrant snapshot save init
vagrant up
```

## install `ansible`

### on Mac

```sh
brew install ansible
```

### on Windows

```sh
ssh pg0
su - root
yum install -y ansible
exit
cd /vagrant
```

## `ansible-playbook`

```sh
ansible-playbook site.yml -i inventories/vagrant/hosts.yml
```

## startup PostgreSQL and pgpool-II

### 1. ssh setup

ssh `pg1` and `pg2`

```sh
su - root
passwd postgres

su - postgres
ssh-keygen
ssh-copy-id backend-pg1
ssh-copy-id backend-pg2
```

### 2. startup `primary` PostgreSQL database at `pg1`

```sh
systemctl start postgresql-10.service
systemctl start pgpool.service
```

### 3. startup `standby` PostgreSQL database

```sh
pcp_recovery_node -h pg -U postgres -n 1
```

### 4. startup pgpool-II at `pg2`

```sh
systemctl start pgpool.service
```

## check

```sh
pcp_watchdog_info -h pg -U postgres -v
Password:
Watchdog Cluster Information
Total Nodes          : 2
Remote Nodes         : 1
Quorum state         : QUORUM EXIST
Alive Remote Nodes   : 1
VIP up on local node : YES
Master Node Name     : backend-pg1:9999 Linux pg1
Master Host Name     : backend-pg1

Watchdog Node Information
Node Name      : backend-pg1:9999 Linux pg1
Host Name      : backend-pg1
Delegate IP    : 192.168.1.203
Pgpool port    : 9999
Watchdog port  : 9000
Node priority  : 1
Status         : 4
Status Name    : MASTER

Node Name      : backend-pg2:9999 Linux pg2
Host Name      : backend-pg2
Delegate IP    : 192.168.1.203
Pgpool port    : 9999
Watchdog port  : 9000
Node priority  : 1
Status         : 7
Status Name    : STANDBY
```

```sh
psql -h pg -p 9999 -U postgres -c "show pool_nodes"
 node_id |  hostname   | port | status | lb_weight |  role   | select_cnt | load_balance_node | replication_delay
---------+-------------+------+--------+-----------+---------+------------+-------------------+-------------------
 0       | backend-pg1 | 5432 | up     | 0.500000  | primary | 0          | true              | 0
 1       | backend-pg2 | 5432 | up     | 0.500000  | standby | 0          | false             | 0
(2 rows)
```

## memo

ansible check

```sh
ansible all -i inventories/vagrant/hosts.yml -m ping
ansible all -i inventories/vagrant/hosts.yml -m setup
```

start PostgreSQL and pgpool-II

```sh
# start: postgresql-10.service => pgpool.service
ls -1r /usr/lib/systemd/system | egrep "(postgres|pgpool)" | xargs -I{} systemctl start {}

# stop: pgpool.service => postgresql-10.service
ls -1 /usr/lib/systemd/system | egrep "(postgres|pgpool)" | xargs -I{} systemctl stop {}
```

manual install for pg0

```sh
yum install https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm -y
yum install postgresql10 -y

yum install http://www.pgpool.net/yum/rpms/3.7/redhat/rhel-7-x86_64/pgpool-II-release-3.7-1.noarch.rpm -y
yum install pgpool-II-pg10 -y

echo "192.168.1.203 pg" >> /etc/hosts

cat << EOF > .pgpass
pg:*:*:postgres:postgres
pg1:*:*:postgres:postgres
pg2:*:*:postgres:postgres
EOF
chmod 600 .pgpass
psql -h pg -p 9999 -U postgres -c "\l"

cat << EOF > .pcppass
pg:*:postgres:postgres
pg1:*:postgres:postgres
pg2:*:postgres:postgres
EOF
chmod 600 .pcppass
pcp_watchdog_info -h pg -U postgres -w
```

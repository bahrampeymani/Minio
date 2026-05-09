# prerequre
1. 4 server with 2 network interface 1 for replication 1 for service
```
  172.22.34.0/24 > serviee
  10.11.34.0/24 > replication 
```
2. recommend 4 server with 4 same drive
# installation ( all servers)
```

groupadd -r minio-user
useradd -M -r -g minio-user minio-user
wget https://dl.min.io/server/minio/release/linuxamd64/archive/minio.RELEASE.2025-04-22T22-12-26Z
chmod +x minio
mv minio /usr/local/bin/
mkdir /usr/local/share/minio
mkdir /usr/local/share/minio/data
chown minio-user:minio-user /usr/local/share/minio/data

```

# format and Create XFS disk ( all servers)
1. Create XFS disks - dont craete parttion else format
```
mkfs.xfs /dev/sdb
mkfs.xfs /dev/sdc
mkfs.xfs /dev/sdd
mkfs.xfs /dev/sde

```
2- create path
```
rm -rf /mnt/minio/
mkdir -p /minio/disk1
mkdir -p /minio/disk2
mkdir -p /minio/disk3
mkdir -p /minio/disk4
```
3- add to fstab 
```
UUID="b7aafcf0-817a-400c-a000-8130b25ff2cd" /minio/disk1 xfs defaults  0 0
UUID="319df06c-fa32-4a15-b96a-a84226d34c27" /minio/disk2 xfs defaults  0 0
UUID="9a0333d2-e480-478b-a363-15f21864a613" /minio/disk3 xfs defaults  0 0
UUID="9bae2db4-2efe-4b05-b5d3-1bd686b90285" /minio/disk4 xfs defaults  0 0

```
4- change permission
```
chown -R minio-user:minio-user /minio
chmod -R 750 /minio
chown -R minio-user:minio-user /minio/disk1 /minio/disk2 /minio/disk3 /minio/disk4 
```
5. create service env file
```
vi /etc/default/minio
MINIO_VOLUMES="http://srv-minio{1...4}.taksadc.local:9000/minio/disk{1...4}"
MINIO_OPTS="--console-address :9001"
MINIO_ROOT_USER=minioadmintsp
MINIO_ROOT_PASSWORD=sssssssssss
```
6. create service and edit  /etc/systemd/system/minio.service
```
vim /etc/systemd/system/minio.service
chown -R minio-user:minio-user minio.service

[Unit]
Description=MinIO
Documentation=https://min.io/docs/minio/linux/index.html
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio
[Service]
WorkingDirectory=/usr/local
User=minio-user
Group=minio-user
ProtectProc=invisible
EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
Restart=always
LimitNOFILE=65536
TasksMax=infinity
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
~


```




# Install single node


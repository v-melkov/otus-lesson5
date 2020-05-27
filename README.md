### Стенд для домашнего занятия "NFS"
Задание:

vagrant up должен поднимать 2 виртуалки: сервер и клиент
на сервер должна быть расшарена директория
на клиента она должна автоматически монтироваться при старте (fstab или autofs)
в шаре должна быть папка upload с правами на запись
- требования для NFS: NFSv3 по UDP, включенный firewall

#### Vagrant shell на SERVER
##### Отключаем SELINUX
    setenforce 0
    echo SELINUX=disabled > /etc/selinux/config
##### Установим необходимый софт
    yum install rpcbind nfs-utils -y -q
##### Включаем серверы NFS
    systemctl enable rpcbind nfs-server
    systemctl start rpcbind nfs-server
##### Создаем каталоги
    mkdir -p /var/nfs/upload
    chmod -R 777 /var/nfs/upload
##### Добавляем NFS каталог в /etc/exports
    echo "/var/nfs 192.168.11.0/24(rw,sync,no_root_squash,no_all_squash)" >> /etc/exports
    exportfs -r
##### Стартуем файрвол и разрешаем в нём NFS
    systemctl start firewalld.service
    firewall-cmd --permanent --zone=public --add-service=nfs
    firewall-cmd --permanent --zone=public --add-service=mountd
    firewall-cmd --permanent --zone=public --add-service=rpc-bind
    firewall-cmd --permanent --add-port=4001/udp --zone=public
    firewall-cmd --permanent --add-port=4001/tcp --zone=public
    firewall-cmd --permanent --add-port=2049/tcp --zone=public
    firewall-cmd --permanent --add-port=2049/udp --zone=public
    firewall-cmd --reload

#### Vagrant shell на CLIENT
##### Установим необходимый софт
    yum install nfs-utils -q -y
##### Стартуем нужные сервисы
    systemctl start rpcbind
    systemctl enable rpcbind
##### Создаем и монтируем NFS директорию по UDP
    mkdir /mnt/nfs-share
    mount -t nfs 192.168.11.101:/var/nfs/ /mnt/nfs-share/ -o udp
##### Создадим файл test.txt в директории /mnt/nfs-share/upload
    touch /mnt/nfs-share/upload/test.txt
##### Проверим наличие файла    
    ls /mnt/nfs-share/upload
##### Добавим монтирование NFS директории в /etc/fstab
    echo "192.168.11.101:/var/nfs /mnt/nfs-share/ nfs defaults,udp 0 0" >> /etc/fstab

На этом выполнение задания без звездочки завершено.

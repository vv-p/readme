1. OverlayFS для монтирования копии данных mysql:

    1.1 Запуск mysql на отдельном разделе

    Mysql (5.7.21-1.el7) из коробки поддерживает запуск нескольких версий на одном сервере. Нужно только прописать их настройки в /etc/my.cnf

    [mysqld@1]
    datadir=/data/1
    socket=/data/1/mysql.sock
    port=3307
    log-error=/var/log/mysqld-1.log

    [mysqld@2]
    datadir=/data/2
    socket=/data/2/mysql.sock
    port=3308
    log-error=/var/log/mysqld-replica02.log

    1.2 Подготовка файловой системы

    Для каждой копии нужно создать 3 папки + папка lowerdir (/data/mysql) с данными
    upperdir - папка, данные из которой перекрывают то, что лежит в lowerdir
    workdir - папка, необходимая для работы overlayfs
    mountpoint - папка, в которой будет доступна копия lowerdir

    mkdir /data/upperdir1 /data/workdir1 /data/1
    mkdir /data/upperdir2 /data/workdir2 /data/2
    chown mysql:mysql /data/upperdir* /data/workdir/* /data/1 /data/2

    1.3 Монтирование OverlayFS

    mount -t overlay overlay -olowerdir=/data/mysql,upperdir=/data/upperdir1,workdir=/data/workdir1 /data/1
    mount -t overlay overlay -olowerdir=/data/mysql,upperdir=/data/upperdir2,workdir=/data/workdir2 /data/2

    и результат:

    [root@st91 ~]# mount | grep overlay
    overlay on /data/1 type overlay (rw,relatime,lowerdir=/data/mysql,upperdir=/data/upperdir1,workdir=/data/workdir1)
    overlay on /data/2 type overlay (rw,relatime,lowerdir=/data/mysql,upperdir=/data/upperdir1,workdir=/data/workdir1)

    [root@st91 ~]# df -hT | grep data
    /dev/sdb1           ext4       7,8G          66M  7,3G            1% /data
    overlay             overlay    7,8G          66M  7,3G            1% /data/1
    overlay             overlay    7,8G          66M  7,3G            1% /data/2

2. Запуск MySQL на OverlayFS

    [root@st91 ~]# systemctl start mysqld@1
    [root@st91 ~]# systemctl start mysqld@2


3. Проверка работы

    [root@st91 ~]# netstat -ntpl | grep mysqld
    tcp6       0      0 :::3307                 :::*                    LISTEN      3566/mysqld
    tcp6       0      0 :::3308                 :::*                    LISTEN      3843/mysqld

4. Полезные ссылки

    4.1 https://www.fromdual.com/multi-instance-set-up-with-mysql-enterprise-server-5-7-on-rhel-7-with-systemd
    4.2 https://dev.mysql.com/doc/refman/5.7/en/using-systemd.html
    4.3 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/filesystems/overlayfs.txt
    4.4 https://askubuntu.com/questions/109413/how-do-i-use-overlayfs


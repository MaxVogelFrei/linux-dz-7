# Домашнее задание 7

## Размещаем свой RPM в своем репозитории

* создать свой RPM (можно взять свое приложение, либо собрать к примеру апач с определенными опциями)
* создать свой репо и разместить там свой RPM
* реализовать это все либо в вагранте, либо развернуть у себя через nginx и дать ссылку на репо

## Процесс решения

[vagrantfile со скриптом](Vagrantfile)

### Устаналиваем зависимости для сборки пакетов и создания репозитория

	yum install -y redhat-lsb-core wget rpmdevtools rpm-build createrepo yum-utils

### Скачиваю пакет с исходниками nginx и openssl, устаналиваю nginx и распаковываю архив openssl

	wget http://nginx.org/packages/centos/7/SRPMS/nginx-1.16.1-1.el7.ngx.src.rpm
	wget https://www.openssl.org/source/latest.tar.gz
	rpm -i nginx-1.16.1-1.el7.ngx.src.rpm
	tar -xvf latest.tar.gz

### Устанавливаю зависимости для сборки nginx

	yum-builddep /root/rpmbuild/SPECS/nginx.spec -y

### в nginx.spec меняю опцию --with-debug на --with-openssl=/root/openssl-1.1.1d

	sed -i 's/with-debug/with-openssl=\/root\/openssl-1.1.1d/' /root/rpmbuild/SPECS/nginx.spec

### сборка

	rpmbuild -bb /root/rpmbuild/SPECS/nginx.spec

### устаналиваю собранный пакет 

	yum localinstall -y rpmbuild/RPMS/x86_64/nginx-1.16.1-1.el7.ngx.x86_64.rpm

### запускаю, проверяю

	systemctl start nginx
	systemctl status nginx
	● nginx.service - nginx - high performance web server
	   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
	   Active: active (running) since Чт 2019-12-05 11:16:47 UTC; 7min ago
	     Docs: http://nginx.org/en/docs/
	  Process: 12947 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
	 Main PID: 12948 (nginx)
	   CGroup: /system.slice/nginx.service
	           ├─12948 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
	           └─12959 nginx: worker process




### создаю папку для репозитория в дефолт папке nginx

	mkdir /usr/share/nginx/html/repo

### копирую в repo собранный nginx и допольнительно качаю туда percona-release

	cp /root/rpmbuild/RPMS/x86_64/nginx-1.16.1-1.el7.ngx.x86_64.rpm /usr/share/nginx/html/repo/
	wget http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm -O /usr/share/nginx/html/repo/percona-release-0.1-6.noarch.rpm

### утилитой createrepo создаю там репозиторий

	createrepo /usr/share/nginx/html/repo/

### добавляю в конфиг nginx опцию autoindex чтобы увидеть свой файлы в подпапке repo

	sed -i '/index.htm;/ a\ autoindex on;'  /etc/nginx/conf.d/default.conf

### перечитаю конфиг nginx

	nginx -t
	nginx -s reload

### с помощью curl вижу что файлы доступны через браузер

	curl -a http://localhost/repo
	<html>
	<head><title>Index of /repo/</title></head>
	<body>
	<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
	<a href="repodata/">repodata/</a>                                          05-Dec-2019 11:16                   -
	<a href="nginx-1.16.1-1.el7.ngx.x86_64.rpm">nginx-1.16.1-1.el7.ngx.x86_64.rpm</a>                  05-Dec-2019 11:16              783620
	<a href="percona-release-0.1-6.noarch.rpm">percona-release-0.1-6.noarch.rpm</a>                   13-Jun-2018 06:34               14520
	</pre><hr></body>
	</html>	



### добавляю репозиторий в yum

	cat >> /etc/yum.repos.d/otus.repo << EOF
	[otus]
	name=otus-linux
	baseurl=http://localhost/repo
	gpgcheck=0
	enabled=1
	EOF

### проверяю yum

	yum repolist enabled | grep otus
	otus                            otus-linux                                     2

	yum list | grep otus
	percona-release.noarch                  0.1-6                          @otus

### устанавливаю percona-release (nginx я не могу установить, т.к. он уже установлен, и не могу его удалить т.к. тогда не увижу свой репозиторий)

	yum install percona-release -y


	    centos: Loaded plugins: fastestmirror
	    centos: Loading mirror speeds from cached hostfile
	    centos:  * base: mirror.docker.ru
	    centos:  * epel: fedora-mirror02.rbc.ru
	    centos:  * extras: mirror.docker.ru
	    centos:  * updates: dedic.sh
	    centos: Resolving Dependencies
	    centos: --> Running transaction check
	    centos: ---> Package percona-release.noarch 0:0.1-6 will be installed
	    centos: --> Finished Dependency Resolution
	    centos:
	    centos: Dependencies Resolved
	    centos:
	    centos: ================================================================================
	    centos:  Package                  Arch            Version           Repository     Size
	    centos: ================================================================================
	    centos: Installing:
	    centos:  percona-release          noarch          0.1-6             otus           14 k
	    centos:
	    centos: Transaction Summary
	    centos: ================================================================================
	    centos: Install  1 Package
	    centos:
	    centos: Total download size: 14 k
	    centos: Installed size: 16 k
	    centos: Downloading packages:
	    centos: Running transaction check
	    centos: Running transaction test
	    centos: Transaction test succeeded
	    centos: Running transaction
	    centos:   Installing : percona-release-0.1-6.noarch                                 1/1
	    centos:
	    centos:   Verifying  : percona-release-0.1-6.noarch                                 1/1
	    centos:
	    centos:
	    centos: Installed:
	    centos:   percona-release.noarch 0:0.1-6
	    centos:
	    centos: Complete!


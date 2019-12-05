# Домашнее задание 7

## Размещаем свой RPM в своем репозитории

* создать свой RPM (можно взять свое приложение, либо собрать к примеру апач с определенными опциями)
* создать свой репо и разместить там свой RPM
* реализовать это все либо в вагранте, либо развернуть у себя через nginx и дать ссылку на репо

## Процесс решения

[vagrantfile со скриптом](Vagrantfile)

### Устаналиваем зависимости 

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
yum list | grep otus

### устанавливаю percona-release (nginx я не могу установить, т.к. он уже установлен, и не могу его удалить т.к. тогда не увижу свой репозиторий)

yum install percona-release -y


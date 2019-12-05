# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :centos => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.101'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "256"]
          end

          box.vm.provision "shell", inline: <<-SHELL
#          mkdir -p ~root/.ssh
#          cp ~vagrant/.ssh/auth* ~root/.ssh
#           yum install epel-release -y -q
#           yum install fish wget -y -q
# Install tools for building rpm
#           yum install rpmdevtools rpm-build -y -q
#           yum install tree yum-utils mc wget gcc vim git -y -q
# Install tools for building woth mock and make prepares    
#           yum install mock -y -q
#           usermod -a -G mock root
# Install tools for creating your own REPO
#           yum install nginx -y -q
#           yum install createrepo -y -q
# Install docker-ce
#           sudo yum install -y -q yum-utils links \
#           device-mapper-persistent-data \
#           lvm2
#           sudo yum-config-manager \
#           --add-repo \
#           https://download.docker.com/linux/centos/docker-ce.repo
#           yum install docker-ce docker-compose -y -q
#           systemctl start docker
#           docker run hello-world

yum install epel-release -y
yum install -y gcc redhat-lsb-core wget rpmdevtools rpm-build createrepo yum-utils
cd /root
wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm
wget http://nginx.org/packages/centos/7/SRPMS/nginx-1.16.1-1.el7.ngx.src.rpm
wget https://www.openssl.org/source/latest.tar.gz
rpm -i nginx-1.16.1-1.el7.ngx.src.rpm
tar -xvf latest.tar.gz
yum-builddep /root/rpmbuild/SPECS/nginx.spec -y
sed -i 's/with-debug/with-openssl=\/root\/openssl-1.1.1d/' /root/rpmbuild/SPECS/nginx.spec
rpmbuild -bb /root/rpmbuild/SPECS/nginx.spec
yum localinstall -y rpmbuild/RPMS/x86_64/nginx-1.16.1-1.el7.ngx.x86_64.rpm
systemctl start nginx
systemctl status nginx
mkdir /usr/share/nginx/html/repo
cp /root/rpmbuild/RPMS/x86_64/nginx-1.16.1-1.el7.ngx.x86_64.rpm /usr/share/nginx/html/repo/
wget http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm -O /usr/share/nginx/html/repo/percona-release-0.1-6.noarch.rpm
createrepo /usr/share/nginx/html/repo/
sed -i '/index.htm;/ a\ autoindex on;'  /etc/nginx/conf.d/default.conf
nginx -t
nginx -s reload
curl -a http://localhost/repo/
cat >> /etc/yum.repos.d/otus.repo << EOF
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF
yum repolist enabled | grep otus
yum list | grep otus
yum install percona-release -y


      SHELL

      end
  end
end

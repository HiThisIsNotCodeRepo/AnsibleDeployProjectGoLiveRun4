# AnsibleDeployProjectGoLiveRun4

# Prerequisite
1. Master install ansible

# Deploy App
## ad-hoc version
### Prerequisite
Install Git
```shell
ansible node -m yum -a 'name=git state=installed'
```
Install Go
```shell
ansible node -m get_url -a "url=https://golang.org/dl/go1.16.5.linux-amd64.tar.gz dest=~"
ansible node -m unarchive -a "src=~/go1.16.5.linux-amd64.tar.gz dest=/usr/local remote_src=yes"
ansible node -m lineinfile -a "line='export GOROOT=/usr/local/go' dest=/etc/profile"
ansible node -m lineinfile -a "line='export GOROOT=/usr/local/go' dest=/root/.bashrc"
ansible node -m lineinfile -a "line='export GOPATH=~/gocode' dest=/etc/profile"
ansible node -m lineinfile -a "line='export GOPATH=~/gocode' dest=/root/.bashrc"
ansible node -m lineinfile -a "line='export PATH=\$GOPATH/bin:\$GOROOT/bin:\$PATH' dest=/etc/profile"
ansible node -m lineinfile -a "line='export PATH=\$GOPATH/bin:\$GOROOT/bin:\$PATH' dest=/root/.bashrc"
```
Install Docker
```shell
ansible node -m yum -a 'name=yum-utils state=installed'
ansible node -m shell -a 'sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo'
ansible node -m yum -a 'name=docker-ce state=installed'
ansible node -m yum -a 'name=docker-ce-cli state=installed'
ansible node -m yum -a 'name=containerd.io state=installed'
ansible node -m systemd -a 'name=docker enabled=yes'
ansible node -m systemd -a 'name=docker state=started'
```
Install Database populate app
```shell
ansible node -m shell "git clone https://github.com/qinchenfeng/DockerDeployProjectGoLiveRun4.git"
ansible node -m shell "mv DockerDeployProjectGoLiveRun4/ gocode/"
```
### Deploy
```shell
# 1 Run MySQL container
ansible node -m shell "docker run --name paotui_mysql -dp 3306:3306 magicpowerworld/paotui_mysql:20210706"
# 1-1 Prepare database
ansible node -m shell "docker exec -it paotui_mysql bash -c 'mysql -uroot -ppassword < /tmp/mysql.sql'"
# 1-2 Populate database
ansible node -m shell "cd gocode/DockerDeployProjectGoLiveRun4/ && go mod tidy && go run ."
# 2 Run Backend container
ansible node -m shell "docker run --name paotui_back_end --net=host -d magicpowerworld/paotui_back_end:20210706"
# 3 Run Frontend container
ansible node -m shell "docker run --name paotui_front_end -p 80:80 -d magicpowerworld/paotui_front_end:20210706"
```
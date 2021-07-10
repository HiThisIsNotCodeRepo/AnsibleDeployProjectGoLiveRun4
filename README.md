# AnsibleDeployProjectGoLiveRun4

# Prerequisite
Master install ansible

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
ansible node -m file -a "path=~/gocode state=directory"
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
ansible node -m shell -a 'yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo'
ansible node -m yum -a 'name=docker-ce state=installed'
ansible node -m yum -a 'name=docker-ce-cli state=installed'
ansible node -m yum -a 'name=containerd.io state=installed'
ansible node -m systemd -a 'name=docker enabled=yes'
ansible node -m systemd -a 'name=docker state=started'
```
Install Database populate app
```shell
ansible node -m shell -a "git clone https://github.com/qinchenfeng/DockerDeployProjectGoLiveRun4.git"
ansible node -m shell -a "mv DockerDeployProjectGoLiveRun4/ gocode/DockerDeployProjectGoLiveRun4"
```
### Deploy
```shell
# 1 Run MySQL container
ansible node -m shell -a "docker run --name paotui_mysql -dp 3306:3306 magicpowerworld/paotui_mysql:20210706"
# 1-1 Prepare database
ansible node -m shell -a "docker exec -it paotui_mysql bash -c 'mysql -uroot -ppassword < /tmp/mysql.sql'"
# 1-2 Populate database
ansible node -m shell -a "cd gocode/DockerDeployProjectGoLiveRun4/ && go mod tidy && go run ."
# 2 Run Backend container
ansible node -m shell -a "docker run --name paotui_back_end --net=host -d magicpowerworld/paotui_back_end:20210706"
# 3 Run Frontend container
ansible node -m shell -a "docker run --name paotui_front_end -p 80:80 -d magicpowerworld/paotui_front_end:20210706"
```
## playbook version
One script deploy
```shell
# on master
ansible-playbook paotui_deploy.yml
```
*paotui_deploy.yml*
```yaml
- hosts: node
  vars:
    - go_installer: go1.16.5.linux-amd64.tar.gz
    - paotui_mysql_image: magicpowerworld/paotui_mysql:20210706
    - paotui_back_end_image: magicpowerworld/paotui_back_end:20210708
    - paotui_front_end_image: magicpowerworld/paotui_front_end:20210710
  tasks:

    - name: Install Git
      yum: name=git state=installed

    - name: Download Go
      get_url: url="https://golang.org/dl/{{go_installer}}" dest=~

    - name: Extract Go download
      unarchive: src="~/{{go_installer}}" dest=/usr/local remote_src=yes

    - name: Make dir for go code
      file: path=~/gocode state=directory

    - name: Env variable
      lineinfile: line='{{item.content}}' dest={{item.dest}}
      with_items:
        - {content: 'export GOROOT=/usr/local/go', dest: '/etc/profile'}
        - {content: 'export GOROOT=/usr/local/go', dest: '/root/.bashrc'}
        - {content: 'export GOPATH=~/gocode', dest: '/etc/profile'}
        - {content: 'export GOPATH=~/gocode', dest: '/root/.bashrc'}
        - {content: 'export PATH=$GOPATH/bin:$GOROOT/bin:$PATH', dest: '/etc/profile'}
        - {content: 'export PATH=$GOPATH/bin:$GOROOT/bin:$PATH', dest: '/root/.bashrc'}

    - name: Pre install docker
      yum: name=yum-utils state=installed

    - name: Configure yum manager
      shell: yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

    - name: Install docker
      yum: name="{{packages}}" state=present
      vars:
        packages:
          - docker-ce
          - docker-ce-cli
          - containerd.io

    - name: Enable docker
      systemd: name=docker enabled=yes state=started

    - name: Clone database populate app
      shell: git clone https://github.com/qinchenfeng/DockerDeployProjectGoLiveRun4.git

    - name: Move downloaded repo
      shell: mv DockerDeployProjectGoLiveRun4/ gocode/DockerDeployProjectGoLiveRun4

    - name: Run MySQL container
      shell: "docker run --name paotui_mysql -dp 3306:3306 {{paotui_mysql_image}}"

    - name: Wait 20 sec
      shell: sleep 20

    - name: Prepare database
      shell: docker exec -it paotui_mysql bash -c 'mysql -uroot -ppassword < /tmp/mysql.sql'

    - name: Populate database
      shell: cd gocode/DockerDeployProjectGoLiveRun4/ && go mod tidy && go run .

    - name: Run backend container
      shell: "docker run --name paotui_back_end --net=host -d {{paotui_back_end_image}}"

    - name: Run frontend container
      shell: "docker run --name paotui_front_end -p 443:443 -d {{paotui_front_end_image}}"

```

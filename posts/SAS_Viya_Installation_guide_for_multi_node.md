# SAS Viya Installation Guide for multi node

[TOC]

## 0. Viya 삭제

삭제시 자동으로 인스톨 디렉토리를 '_'를 prefix 로 붙여 백업 하게 됩니다.

~~~{bash}
ansible-playbook deploy-cleanup.yml
~~~



## 1. 설치 사전 작업

### 1-1. 설치 환경

SAS Viya 설치 관련 내용은 아래와 같습니다.

| 항목               | 내용                         |
| ------------------ | ---------------------------- |
| 설치 유저          | root                         |
| 설치 파일 위치     | /opt/sas/install             |
| SAS Viya 설치 위치 | /opt/sas/viya, /opt/sas/spre |



### 1-2. 유저 생성 및 디렉토리 생성

~~~bash
# 사용자 추가
useradd sas
useradd cas -g sas

# 인스톨 디렉토리 생성
mkdir -p /opt/install
chown -R sas:sas /opt
~~~



### 1-3. 설치시 필요 파일 리스트

| 파일명                       | 설명                                           | 다운로드 링크                                                |
| ---------------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| sas-orchestration-linux.tgz  | ansible playbook build                         | [다운로드](https://support.sas.com/en/documentation/install-center/viya/deployment-tools/34/command-line-interface.html) |
| mirrormgr-linux.tgz          | 설치 파일 다운로드 및 yum 로컬 리포지토리 구성 | [다운로드](https://support.sas.com/en/documentation/install-center/viya/deployment-tools/34/mirror-manager.html) |
| SAS_Viya_deployment_data.zip | 라이센스 및 제품 목록                          | 메일첨부                                                     |
| OpenLDAP-master.tar.gz       | 계정관리용 LDAP                                | [다운로드](https://gitlab.sas.com/canepg/OpenLDAP) (SAS 네트워크) |





### 1-4. 파일 업로드

SOE (SAS Order Email) 에 첨부된 SAS_Viya_deployment_data.zip 파일을 다운받아 서버의 HOME  디렉토리에 업로드

![order_mail_1](../img/order_mail_1.png)



### 1-5. SAS CLI 유틸리티 다운로드

[SAS Viya Command line Interface Utility](https://support.sas.com/en/documentation/install-center/sas-viya/deployment-tools/command-line-interface.html)  웹사이트 에서 운영체제에 맞는 CLI 다운로드

다운로드 대상파일 : sas-orchestration.tgz

![image-20180727163046658](../img/image-20180727163046658.png)



### 1-6. Mirror Repo 생성(3.4 이상)

다운로드 : [Mirror Manager 다운로드](https://support.sas.com/en/documentation/install-center/viya/deployment-tools/34/command-line-interface.html)

####SAS 패키지 미러 만들기

```
./mirrormgr mirror --deployment-data /home/ec2-user/SAS_Viya_deployment_data.zip --path /opt/install/mirror --platform x64-redhat-linux-6 --latest
```

#### Ansible Playbook 생성

```
./sas-orchestration build --input ${path-to-SOE-zip-file} --repository-warehouse "file:///home/ec2-user/mirror" --platform redhat
```

> Platform TAG
>
> oracle linux , redhat linux : **redhat**
>
> suse linux : **suse**



### Ansible playbook 생성

~~~{bash}
# sas-orchestration.tgz 압축풀기 : sas-orchestration 실행파일 최종생성
gunzip sas-orchestration.tgz
tar -xvf sas-orchestration.tar

# 플레이북 생성 : SAS_Viya_playbook.tgz 파일 최종 생성
./sas-orchestration build --input SAS_Viya_deployment_data.zip

# 플레이북 압축 풀기
tar -xvf SAS_Viya_playbook.tgz

# 플레이북 파일 install 디렉토리로 이동
cp SAS_Viya_playbook.tgz /opt/sas/install/

# 플레이북 압축 풀기
cd /opt/sas/install
gunzip SAS_Viya_playbook.tgz
tar -xvf SAS_Viya_playbook.tar
~~~



### 1-8. SSH 키생성 및 복사

#### /etc/hosts 파일에 클러스터 노드 추가

~~~
172.31.30.9 casc.amore.com
172.31.20.129 casw1.amore.com
172.31.17.73 casw2.amore.com
172.31.27.86 casw3.amore.com
~~~

#### root 유저 키생성

~~~bash
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
~~~

#### root 유저 키복사

~~~bash
ssh-copy-id {hostname-or-ip}
~~~

####sas,cas 유저에 대한 키복사

~~~
vi /etc/gridhosts
===============================
casc.amore.com
casw1.amore.com
casw2.amore.com
casw3.amore.com
~~~

~~~
for i in $(cat /etc/gridhosts)
do
	scp -r ~/.ssh $i:/home/sas;
done
~~~

~~~
for i in $(cat /etc/gridhosts)
do
	scp -r ~/.ssh $i:/home/cas;
done
~~~



## 2. Pre-Installation

### YUM Repository 추가

#### Epel

~~~bash
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-$majversion.noarch.rpm
~~~

> $majversion 에 리눅스 버전 추가 
>
> redhat 7.4 일경우 $majversion 은 7



###확인사항

````
# 운영체제 확인
cat /etc/*-release

m
# sudo 권한 확인
sudo -v

# systemd 버전확인 (219 ~230버전)
rpm -qa systemd

# 필요하면 systemd 업데이트
yum update systemd

# HTTPD 확인
service httpd status

# 필요시 HTTPD 설치
yum install httpd

# java 확인
java -veriosn

# java 설치
yum install java-1.8.0-openjdk

# 필수 요구 패키지 설치 확인
rpm -qa at nfs-utils.x86_64 nfs-utils-lib.x86_64 gcc glibc firefox compat-libstdc++-33 compat-glibc GLIBC 2.12 libuuid libSM libXrender fontconfig libstdc++ zlib apr ksh numactl perl-Net-SSLeay libXext libXext.i686 libXp libXp.i686 libXts libXtst.i686 libgcc libgcc.i686 libpng12 libpng12.i686 python 2.7 xterm xauth libXmu uuid mod_ssl tcl

# 패키지 설치 스크립트
pkgs="                    \
 at                       \
 nfs-utils.x86_64         \
 nfs-utils-lib.x86_64     \
 gcc                      \
 glibc                    \
 firefox                  \
 compat-libstdc++-33      \
 compat-glibc             \
 GLIBC 2.12               \
 libuuid                  \
 libSM                    \
 libXrender               \
 fontconfig               \
 libstdc++                \
 zlib                     \
 apr                      \
 ksh                      \
 numactl                  \
 perl-Net-SSLeay          \
 libXext                  \
 libXext.i686             \
 libXp                    \
 libXp.i686               \
 libXtst                  \
 libXtst.i686             \
 libgcc                   \
 libgcc.i686              \
 libpng12                 \
 libpng12.i686            \
 python 2.7               \
 xterm                    \
 xauth                    \
 libXmu                   \
 uuid                     \
 mod_ssl                  \
 tcl                      \
"
yum install --nogpgkey $pkgs -y

# 방화벽 중단
service firewalld stop
sudo systemctl disable firewalld.service

# Selinux 중단
sudo sestatus 
만약 활성화 상태일 경우 모든 타겟 서버에 다음 명력을 통해 permissive mode 를 활성화 함

sudo setenforce 0    => 안먹을 때도 있음 재부팅 필요
sudo sed -i.bak -e 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config

# Ansible 인스톨을 위한 EPEL 리포지토리 추가 스크립트

## Attach EPEL
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

# Display the available repositories
sudo yum repolist
````

### Ansible 설치

+ Ansible 설치를 위한 패키지 설치

~~~bash
sudo yum install -y python python-setuptools python-devel openssl-devel
sudo yum install -y python-pip gcc wget automake libffi-devel python-six
~~~

+ Epel 삭제

~~~
sudo yum remove -y epel-release
~~~

+ PIP 업그레이드

~~~
sudo pip install --upgrade pip setuptools
~~~

+ Ansible 설치

~~~
sudo pip install ansible==2.4.1
~~~

+ 설치 확인

~~~
ansible --version
ansible localhost -m ping
~~~



### 커널변수 설정

+ /etc/ssh/sshd_config 파일 수정

~~~
MaxStartups 100
MaxSessions 100
~~~

+ /etc/security/limits.conf 파일 수정

~~~
*	soft	nproc	100000
*	hard	nproc	100000
*	soft	nofile	350000
*	hard	nofile	350000
*	soft	stack	10240
*	hard	stack	32768
sas	-	nofile	150000
*	-	nofile	150000
sas	-	stack	10240
~~~

+ /etc/security/limits.d/20-nproc.conf 파일 수정

~~~bash
*          soft    nproc     150000
root       soft    nproc     unlimited
*       -       nproc   100000
~~~

+ /etc/sysctl.conf 파일 수정

~~~
kernel.shmmni = 4096
kernel.sem = 512 32000 256 1024
net.core.somaxconn = 2048
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
~~~

+ 설정 변경 확인

~~~
sysctl -p
~~~

+ /etc/systemd/system.conf 파일 수정

~~~
DefaultTimeoutStartSec=1800s
DefaultTimeoutStopSec=1800s
~~~



##3. Installation

/opt/sas/install/sas_viya_playbook 경로에서 다음 명령어 수행

~~~
cp sample-inventories/inventory_local.ini ./inventory.ini
~~~

 vi /sas/install/sas_viya_playbook/inventory.ini  수행하여 ansible_connection 확인.  ansible을 local로 사용시 default 설정 유지. remote 사용시 ansible_connection을   ansible_host로 변경 후 target host name으로 값 변경 

~~~
 vi /sas/install/sas_viya_playbook/inventory.ini 
 
 [host-definitions]
 deployTarget ansible_connection=local
~~~

####cascache 생성 

~~~bash
# root 계정으로 /opt 하위에 sas/cascache 디렉토리 생성. 
# cascache 디렉토리에 chmod 777 cascache 명령어를 통해 권한 부여 

mkdir -p /opt/sas/cascache
chmod 777 /opt/sas/cascache
~~~



#### vars.yml  수정

~~~BASH
# vi /opt/sas/install/sas_viya_playbook/vars.yml 를 입력하여 파일을 연 후 적절한 DEPLOYMENT_LABEL 을 설정
DEPLOYMENT_LABEL : "{{ DEPLOYMENT_ID}}"

# sas_install_type 설정. default 설정은 all 이며 모든 소프트웨어를 설치. 
# programming 옵션 설정 시 CAS, SAS Foundation, SAS Studio를 포함한 programming interface만 설치
sas_install_type : all

# casenv_group 값을 cas가 속해 있는 그룹의 이름으로 설정
casenv_group : sas

# Full Deployment를 위하여 LDAP user를 cas admin user로 설정. casenv_admin_user의 값을 주석 해제 후 해당하는 user 이름으로 값 설정.
casenv_admin_user : cas

# CAS_DISK_CACHE를 주석 해제 후 /opt/sas/cacscache 로 경로 설정
CAS_DISK_CACHE : /opt/sas/cascache
~~~

#### Validation 수행

/opt/sas/install/sas_viya_playbook/  에서 아래 명령어 수행

~~~bash
ansible-playbook system-assessment.yml 
~~~

#### Deployment 수행

/opt/sas/install/sas_viya_playbook/  에서 아래 명령어 수행

~~~
# 일반적인 방법
ansible-playbook -vvv site.yml

# 백그라운드 수행
nohup ansible-playbook -vvv site.yml &
~~~

> 백그라운드로 수행할 경우 deployment.log 를 확인하면 인스톨 상황을 알 수 있음







###Yum Repository 등록

subscription-manager 미등록시 subscription-manager 등록 혹은 centOS repository 사용 

> vi /etc/yum.repository.d/centos.repo  

~~~
[base]
name=CentOS-$releasever - Base
baseurl=http://ftp.daum.net/centos/7/os/$basearch/
gpgcheck=1
gpgkey=http://ftp.daum.net/centos/RPM-GPG-KEY-CentOS-7

[updates]
name=CentOS-$releasever - Updates 
baseurl=http://ftp.daum.net/centos/7/updates/$basearch/
gpgcheck=1
gpgkey=http://ftp.daum.net/centos/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-$releasever - Extras 
baseurl=http://ftp.daum.net/centos/7/extras/$basearch/
gpgcheck=1
gpgkey=http://ftp.daum.net/centos/RPM-GPG-KEY-CentOS-7

[centosplus]
name=CentOS-$releasever - Plus 
baseurl=http://ftp.daum.net/centos/7/centosplus/$basearch/
gpgcheck=1
gpgkey=http://ftp.daum.net/centos/RPM-GPG-KEY-CentOS-7
~~~





~~~
rpm -qa at nfs-utils.x86_64 nfs-utils-lib.x86_64 gcc glibc firefox compat-libstdc++-33 compat-glibc GLIBC 2.12 libuuid libSM libXrender fontconfig libstdc++ zlib apr ksh numactl perl-Net-SSLeay libXext libXext.i686 libXp libXp.i686 libXts libXtst.i686 libgcc libgcc.i686 libpng12 libpng12.i686 python 2.7 xterm xauth libXmu uuid mod_ssl tcl

~~~

~~~
service firewalld stop
systemctl disable firewalld.service
~~~


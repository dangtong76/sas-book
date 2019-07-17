## SAS Viya 오프라인 설치를 위한 리포지토리 구성 가이드

[TOC]

본 가이드는 인터넷 연결이 전혀 허락되지 않는 금융권 고객사와 같은 서버환경에서 설치를 위한 각종 패키지를 usb 에 저장하여 설치 하는 경우를 위한 가이드 입니다. 오프라인 환경에서 설치시 3가지가 필요합니다.

1. CentOS 리포지토리(redhat subscription 이 없을경우)

2. epel 리포지토리(python 관련 패키지 설치를 위해)

3. Pip 리포지토리 (Ansible 설치를 위해 필요)

4. SAS Viya 리포지토리


### 1. CentOS 리포지토리 생성

아래와 같이 centos 용 리포지토리를 구성합니다.

```{bash}
[centos 6.X]
rsync -avz --exclude='repo*' rsync://ftp.kaist.ac.kr/CentOS/7/os/x86_64/ /repos/centos/7/os/x86_64/ 
rsync -avz --exclude='repo*' rsync://ftp.kaist.ac.kr/CentOS/7/updates/x86_64/ /repos/centos/7/updates/x86_64/ 
rsync -avz --exclude='repo*' rsync://ftp.kaist.ac.kr/CentOS/7/extras/x86_64/ /repos/centos/7/extras/x86_64/ 
rsync -avz --exclude='repo*' rsync://ftp.kaist.ac.kr/CentOS/7/centosplus/x86_64/ /repos/centos/7/centosplus/x86_64
```



~~~{bash}
[centos 7.X]
rsync -avz --exclude='repo*' rsync://ftp.kaist.ac.kr/CentOS/6/os/x86_64/ /repos/centos/6/os/x86_64/ 
rsync -avz --exclude='repo*' rsync://ftp.kaist.ac.kr/CentOS/6/updates/x86_64/ /repos/centos/6/updates/x86_64/ 
rsync -avz --exclude='repo*' rsync://ftp.kaist.ac.kr/CentOS/6/extras/x86_64/ /repos/centos/6/extras/x86_64/ 
rsync -avz --exclude='repo*' rsync://ftp.kaist.ac.kr/CentOS/6/centosplus/x86_64/ /repos/centos/7/centosplus/x86_64
~~~

### 2. epel 리포지토리 생성

epel 의 경우 rsync 및 ftp bulkload 를 막아 놓은 사이트가 많아 아래 사이트에서 다운로드 받습니다.

~~~{bash}
rsync -avz --exclude='repo*' rsync://ftp.iij.ad.jp/pub/linux/Fedora/epel/7/x86_64/  /Volumes/dangtong/repos/epel/7/x86_64/ 
~~~



### 3. ansible 설치 를 위한 pip repository 생성

#### 3-1. repository 디렉토리 생성

~~~{bash}
mkdir /home/ec2-user/pip_repo
cd /home/ec2-user/pip_repo
~~~



#### 3-2. ansible 관련 패키지만 다운로드

~~~bash
pip download ansible==2.4.1
~~~



#### 3-3. pip 설정 파일 만들기 또는 변경

~~~
cd /root/.config
mkdir pip
vi pip.conf
~~~

아래 내용은 pip.conf 파일에 넣어 줍니다.

~~~{bash}
index-url=file:///home/ec2-user/
~~~

커맨드 라인에서 아래 명령을 수행합니다.

~~~bash
export PIP_FIND_LINKS=file:///home/ec2-user/pip_repo
~~~



#### 3-4. 실제 SAS Viya ansible 인스톨 시에는 아래 명령어를 사용합니다.

~~~{bash}
pip install --no-index  ansible==2.4.1
~~~



### 4. SAS Viya 리포지토리  생성(3.4 이상)



#### 4-1. 미러메니저 다운로드

[다운로드 클릭](https://support.sas.com/en/documentation/install-center/viya/deployment-tools/34/mirror-manager.html)

#### 4-2. Mirror Manager 압축풀기
gunzip mirrormgr-linux.tgz
tar -xvf mirrormgr-linux.tar

#### 4-3. SAS 미러 리포지토리 만들기
명령어 : mirrormgr mirror --deployment-data {path-to-SOE-file} --path {mirror-path} --platform {platform-tag} --log-file {log-file-path} --latest

~~~bash
./mirrormgr mirror --deployment-data /home/ec2-user/SAS_Viya_deployment_data.zip --path /opt/install/mirror --platform x64-redhat-linux-6 --log-file /home/ec2-user/mirrormgr.log --latest
~~~


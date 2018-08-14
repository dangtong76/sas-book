# SAS Viya 플랫폼에 Jupyter Hub 설치

------

OS root 계정에 [Jupyter hub](https://github.com/jupyterhub/jupyterhub) 를 설치하는 것을 가정으로 작성 되었습니다.

### 요구사항

- 64bit Python 2.7 or 3.4-3.6 (리눅스)
- 바이너리 통신을 위한 libnuma.so.1 라이버리 (numactl 설치하면 됨)
- 이미 SAS Viya 가 서버에 인스톨 되어야 합니다.

### Anaconda 설치

- [Anaconda](http://www.anaconda.com/) 다운로드 및 설치

![img](https://dangtong.gitbooks.io/sas-viya-with-python/content/assets/anaconda_source_page_jupyter_hub.png)

- - 다운로드 : <https://www.anaconda.com/download/> (반듯이 64-bit 버전 다운로드)
  - 서버의 적당한 디렉토리에 업로드

```
# bash ./Anaconda3-5.1.0-Linux-x86_64.sh
```

### ![img](https://dangtong.gitbooks.io/sas-viya-with-python/content/assets/jupyter_hub_install_location.png)

설치중 위와 같이 설치 위치를 물으면 직접 입력 합니다. (예: /usr/local/anaconda3)

### 설치 위치 PATH 추가 하기

아나콘다 명령어를 바로 실행하기 위해 PATH 를 아래와 같이 .bash_profile 에 추가 합니다.

```
# vi ~/.bash_profile
```

```
/* .bash_profile */

PATH=$PATH:/usr/local/anaconda3/bin:$HOME/bin
```

### 가상환경 생성

```
# conda create --name jupyter-hub python=3.5 
# conda info --envs (생성된 가상환경 확인)
```

![img](https://dangtong.gitbooks.io/sas-viya-with-python/content/assets/conda_env.png)

### Jupyter Hub 설치

```
/* 의존성 패키지 설치(nodejs / configurable-http-proxy) */

# curl --silent --location https://rpm.nodesource.com/setup_8.x | sudo bash -

# yum -y install nodejs

# npm install -g configurable-http-proxy

/* 파이썬 가상 환경에서 수행 */

# source activate jupyter-hub

# pip install jupyterhub

# pip install --upgrade notebook
```

### 인증서 생성 (self signed Certificate)

```
# openssl req -newkey rsa:4096 -nodes -sha512 -x509 -days 3650 -nodes -out /root/cert/cert.pem -keyout /root/cert/key.pem
```

인증서 생성 시에 각각의 질문 항목에 적절한 내용을 입력 합니다

인증서 새성 후 cert 디렉토리로 이동하여 인증서가 생성 되었는지 확인 합니다.![img](https://dangtong.gitbooks.io/sas-viya-with-python/content/assets/cert_jupyterhub.png)

### Jupyterhub 실행

```
/* Jupyter Hub 실행 */
# jupyterhub --ip {your_ip_address} --port {ssl_port: eg 443} --ssl-cert /root/cert/cert.pem --ssl-key /root/cert/key.pem
```

![img](https://dangtong.gitbooks.io/sas-viya-with-python/content/assets/jupyter_exe.png)url 로 접속하여 정상 유무를 확인 하고 Ctrl + C 로 서버를 중지 합니다.

### SWAT Library 다운로드 및 설치

- SWAT 패키지 다운로드 및 설치
- - 다운로드 사이트 : <https://github.com/sassoftware/python-swat/releases>
  - 다운로드 후 서버에 적당한 위치에 업로드후 설치합니다.

```
# yum install python-swat-1.3.0-linux64.tar.gz
```

### Jupyterhub 재실행 및 접속

```
/* Jupyter Hub 실행 */

# jupyterhub --ip {your_ip_address} --port {ssl_port: eg 443} --ssl-cert /root/cert/cert.pem --ssl-key /root/cert/key.pem
```

[new] -> [python3] 선택하여 노트북 실행

### 샘플 코드로 테스트 하기

```
// OS 계정이 다르기 때문에 클라이언트 인증서에 대한 환경변수를 직접 입력 합니다. 
Import os
os.environ[‘CAS_CLIENT_SSL_CA_LIST’] = ‘/opt/sas/viya/config/etc/SASSecurityCertificateFramework/cacerts/trustedcerts.pem’

import swat
conn = swat.CAS('ip address', '5570', 'username','password')
conn.builtins.serverStatus()
```

![img](https://dangtong.gitbooks.io/sas-viya-with-python/content/assets/jypyter_hub_result.png)
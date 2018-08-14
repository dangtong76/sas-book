# SAS Viya with SWAT on jupyter notebook

SAS Viya서버에 Jupyter Notebook 을 설치 하고 실행 하는 방법에 대한 가이드 입니다.

Python 을 통해 Viya 플랫폼을 사용하기 위해서는 SWAT 라이브러리가 필요합니다.

**본 가이드는 이미 서버에 Viya 플랫폼이 설치 되었다고 가정합니다.**

------

- pip 설치

```
# yum install python-pip
```

- jupyter notebook 설치

```
# pip install jupyter
```

- numactl 패키지 설치

```
# yum install numactl
```

> numactl 은 SAS Viay의 CAS(Cloud Analytic Server) 서버와 바이너리 통신을 위해 필요한 라이브러리 입니다.

- SWAT 패키지 다운로드 및 설치
- - 다운로드 사이트 : <https://github.com/sassoftware/python-swat/releases>
  - 다운로드 후 서버에 적당한 위치에 업로드후 설치합니다.

```
# yum install python-swat-1.3.0-linux64.tar.gz
```

- 클라이언트 인증서 환경변수 설정

```
# export CAS_CLIENT_SSL_CA_LIST="/opt/sas/viya/config/etc/SASSecurityCertificateFramework/cacerts/trustedcerts.pem"
```

- jupyter notebook 실행하기

```
/* 모든네트워크 인터페이스 에서 포트를 열고, 작업 디렉토리를 설정 */
# jupyter notebook --ip=* --notebook-dir=/home/sas
```

- 간단한 소스를 통해 확인하기

```
import swat
conn = swat.CAS('ip address', '5570', 'username','password')
conn.builtins.serverStatus()
```

> 바이너리 통신 포트 : 5570(default) , REST 통신포트 : 8777(default)
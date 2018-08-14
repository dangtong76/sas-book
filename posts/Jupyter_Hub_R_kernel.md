# Jupyter Hub에 R 커널 적용하기

------

아나콘다 환경에 주피터 허브가 이미 설치 되어 있다고 가정합니다.

#### R 설치

```
# conda install -c r r-essentials
```

#### R 커널을 위한 SAS Viya SWAT 패키지 다운로드

- 다운로드 사이트 : <https://github.com/sassoftware/r-swat/releases>

#### SWAT 설치를 위한 의존성 라이브러리 설치

```
# R
> install.packages('httr')
> install.packages('jsonlite')
```

#### SWAT 라이브러리 설치

```
# R CMD INSTALL R-swat-1.2.0-linux64.tar.gz
```

#### 테스트

```
library(swat)
Sys.setenv(CAS_CLIENT_SSL_CA_LIST = "/opt/sas/viya/config/etc/SASSecurityCertificateFramework/cacerts/trustedcerts.pem")
conn <- swat::CAS('192.168.10.17', 5570, username='viya_sask01', password='viya')
res <- cas.builtins.serverStatus(conn)
res$nodestatus[1:5,]
```

#### 테스트 완료 화면

![img](https://dangtong.gitbooks.io/sas-viya-with-python/content/assets/sas_viya_jupyter_hub_R_test.png)
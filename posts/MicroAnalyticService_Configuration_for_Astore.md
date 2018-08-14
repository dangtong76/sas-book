# Micro Analytic Service -Astore 설정 가이드

## 개요

+ Astore 모델 저장 공간

~~~
/opt/sas/viya/config/data/modelsvr/astore
~~~

+ MAS 에서 모델을 참조하는 경로

~~~
/models/astores/viya
~~~

​      

모델저장 공간을 MAS 에서 잘 참조 할수 있도록 심볼릭 링크를 만들어 주되, 보안 정책에 위배 되지 않도록 만들어야 함



## 설정 내용

#### 추가 시스템 그룹 생성

analytic 이라는 그룹을 추가로 만들고 모델생성 유저를 해당 그룹에 추가 해주면됨

~~~
groupadd analytic
gpasswd -a sas analytic
gpasswd -a cas analytic
gpasswd -a viyademo01 analytic
gpasswd -a viyademo02 analytic
~~~

####setguid 설정(리눅스 특수권한)

/opt/sas/viya/config/data/modelsvr/astore 폴더를 analytic 그룹으로 변경하고, 특수권한 부여

~~~
chown sas:analytic astore
chmod 2775 astore /* owner, group 에 rwx 를 부여하고 setguid 설정(2)
~~~



#### Symbolic link 생성

~~~
mkdir /models/astores
cd /model/astores
ln -s /opt/sas/viya/config/data/modelsvr/astore viya
~~~



Model Manager 에서 Astore 를 publish 하게 되면 /opt/sas/viya/config/data/modelsvr/astore 에 Astore 파일이 생성되고 MAS 에서는 /models/astores/viya 링크를 통해 모델을 참조하게됨

 


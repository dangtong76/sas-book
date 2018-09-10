## SAS Viya3.4 인증서 교체 가이드

### 인증서 다운로드

교체할 새로운 인증서는 Base64 PEM 인코딩 되어 있어야 합니다.

### 인증서 복사 및 권한 설정

인즌서 복사

~~~bash
cp customer.crt /etc/pki/tls/certs/customer.crt
cp customer.key /etc/pki/tls/private/customer.key
~~~

권한 설정 변경

~~~{bash}
chmod 600 customer.key 
chmod 644 customer.crt
~~~



### ssl.conf 파일 수정

/etc/httpd/conf.d 디렉토리 내에 ssl.conf 파일의 아래 부분을 수정합니다.

~~~{bash}
SSLCertificateFile /etc/pki/tls/certs/customer.crt
SSLCertificateKeyFile /etc/pki/tls/private/customer.key
~~~

만약 체인드인증서를 사용 할 경우 SSLCertificateFile 항목을 주석 처리 하고 SSLCertificateChainFile 을 수정 합니다.

~~~{bash}
SSLCertificateChainFile /etc/pki/tls/certs/customer-chain.crt
~~~



### httpproxy 서비스 재가동

~~~{bash}
systemctl restart sas-viya-httpproxy-default
~~~



### https 서비스 재가동

~~~{bash}
systemctl restart httpd
~~~



### vars.yml 파일 수정

/opt/install/sas_viya_playbook 디렉토리로 이동하여 vars.yml 파일의 **HTTPD_CERT_PATH** 항목을 아래와 같이 수정 합니다.

~~~bash
HTTPD_CERT_PATH: '/etc/pki/tls/certs/customer.crt'
~~~



### ansible-playbook 실행

~~~{bash}
ansible-playbook -i inventory.ini utility/distribute-httpd-certs.yml
~~~



### 전체 서비스 재가동

~~~{bash}
systemctl stop sas-viya-all-services stop
systemctl start sas-viya-all-services start
~~~




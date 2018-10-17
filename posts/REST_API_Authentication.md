## REST API 클라이언트 등록 및 토큰 조회

[TOC]

### 1. 클라이언트 등록 하기

SAS Viya 서버의 아래 디렉토리로 이동합니다. 여기서 말하는 클라이언트란 REST API 를 호출하는 Application을 지칭 한다고 생각해도 무방합니다.

~~~{bash}
cd /opt/sas/viya/config/etc/SASSecurityCertificateFramework/tokens/consul/default
~~~

client.token 파일을 조회 합니다.

~~~{bash}
cat client.token 
~~~

아래 명령어로 Client 등록시에 사용할 Oauth 토근을 획득합니다.. Client 이름은 원하는 이름을 마음대로 정하셔도 무방합니다. 

~~~{bash}
curl -X POST "http://example.com/SASLogon/oauth/clients/consul?callback=false&serviceId=app" \
      -H "X-Consul-Token: <value-of-consul-token-goes-here>"
~~~

위와 같이 **serviceId** 에 Client 이름을 **app** 라고 등록 하였습니다.

**X-Consul-Token** 항목에는 client.token 에서 조회한 클라이언트 토큰을 넣어 줍니다.

그러면 아래와 같이 Oauth 토큰을 reponse 로 받게 됩니다.

아래 명령으로 Client를 등록 해줍니다.

~~~{bash}
curl -X POST "https://{your-host-name}/SASLogon/oauth/clients" \
        -H "Content-Type: application/json" \
        -H "Authorization: Bearer {Oauth-access-tokenz}" \ # 위에서 받은 Oauth 토큰을 넣어줍니다.
        -d '{
          "client_id": "app", #application 이름
          "client_secret": "<secret-goes-here>", # 비밀번호 설정
          "scope": ["openid"],
          "authorized_grant_types": ["password"],
          "access_token_validity": 43199
         }'
~~~

Client 등록에 성공하면 아래와 같음 메시지를 response 로 보내 줍니다.

~~~
{"scope":["openid"],"client_id":"app","resource_ids":["none"],"authorized_grant_types":["password","refresh_token"],
          "access_token_validity":43199,"authorities":["uaa.none"],"lastModified":1521124986406}
~~~



### 2. Access 토큰 얻기

아래와 같이 향후 Client 에서 사용할 Access 토큰을 얻기 위해 아래와 같이 요청 합니다.

~~~
curl -X POST "https://server.example.com/SASLogon/oauth/token" \
      -H "Content-Type: application/x-www-form-urlencoded" \
      -d "grant_type=password&username=<user-id>&password=<password>" \
      -u "app:mysecret"
~~~

수행 하게 되면  access 토큰을 리턴 하게 됩니다. 토근을 Client 에서 요청시  Authorization 항목에 넣어주면 됩니다.

> 사실 Access token 과 Refresh token 2개가 조회 됩니다. Access Token 의 경우 24시간 동안 사용하지 않게 되면 만료가 되지만, Refresh token의 경우 무기한 사용 할 수 있습니다.



### 3. 클라이언트 에서 Access 토큰 사용하기

아래와 같이 Access 토큰을 이용해 SAS Viya 서버에 요청을 보내 봅니다.

아래의 예는 폴더 서비스에 폴더 목록을 가져 오는 요청 입니다.

~~~
 curl -X GET "https://server.example.com/folders/folders/@myFolder" \
      -H "Accept: application/json" \
      -H "Authorization: Bearer <TOKEN-STRING>"
~~~



### 4. Refresh Token 을 이용한 토큰 갱신

새로 토큰을 얻는것 보다 Refresh 토큰을 이용해 Access Token 을 갱신하는 것이 서버나 클라이언트 입장에서 부하가 적고, 토큰을 빨리 발급 받습니다. 수행 하게 되면 새로운 access_token 을 발급받게 됩니다.

~~~
curl -X POST "https://server.example.com/SASLogon/oauth/token" \
      -H "Content-Type: application/x-www-form-urlencoded" \
      -d "grant_type=refresh_token&refresh_token=<refresh_token> \
      -u "app:mysecret"
~~~


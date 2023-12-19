---
layout: post
title: ModSecurity
subtitle: SeSAC 성동캠퍼스 1기
author: 박지수
categories: SeSAC
---

### 1. ModSecurity란?
- 오픈소스 웹 애플리케이션 방화벽(WAF)
- SQL Injection, Cross-Site Scripting(XSS) 등의 웹 공격 탐지 및 차단
- Apache 웹 서버를 위한 주요 공격을 차단

### 2. ModSecurity 환경 설정
dvwa로 취약한 웹 어플리케이션 서버를 돌리고, modsecurity를 사용해 취약점을 탐지하고 차단하는 실습을 진행하자. 이를 위해 칼리 리눅스를 생성하과 다음과 같은 단계를 거쳐 dvwa를 구축하고 modsecurity를 설치하여 우리가 사용할 환경에 맞게 설정 해 주어야 한다.
#### 2.1. dvwa 설치
1. 칼리 리눅스 랩 설치 후 dvwa 실행 확인   
```shell
# apt install update
# apt install -y kali-linux-labs
# dvwa-start
```
위 명령어를 입력한 후 localhost:42001 로 접속하여 dvwa의 동작을 확인하라.  
`http://localhost:42001`
2. php.ini 파일 수정  
	php란 웹사이트 제작에 특화된 백엔드 언어이다. php.ini 파일은 웹 사이트에서 실행될 때 사용하는 구성 설정을 지정한다. 각 디렉토리에 들어있는 php.ini 파일을 수정하자
	```shell
	# mousepad etc/php/8.2/apache2/php.ini 

	870번째 라인의 allow_url_include = Off 를 On으로 바꾸고 저장
	```
	 위 과정을 etc/php/8.2/cli/php.ini에서도 수행 해 준다. 
3. dvwa 멈추고 아파치 웹서버의 루트로 배치 및 실행
	```shell
	# cp -r /usr/share/dvwa /var/www/html
	# service apache2 start
	```
4. 웹 브라우저에 접속하여 dvwa 설정
	1. `http://localhost/dvwa` 로 접속
	2. Setup/Reset DB 클릭
	3. 하단의 Create/Reset Database 클릭

	1. 새 창에 about:config 입력 후 엔터
	2. 검색창에 proxy 입력
	3. network.proxy.allow_hijacking_localhost 더블클릭하여 true로 바꾸기

#### 2.2. ModSecurity 설치 및 설정
1. ModSecurity 설치
	```shell
	# apt install libapache2-mod-security2 -y
	```
2. modsecurity.conf 수정
    ```shell
    # cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
    # mousepad /etc/modsecurity/modsecurity.conf

    SecRuleEngine DetectionOnly On 
    ```
3. 아파치 재실행
    ```shell
    # systemctl restart apache2
    ```
만약 위 명령어를 입력했음에도 아파치가 실행되지 않는다면 다음 페이지를 참고하라
https://askubuntu.com/questions/629995/apache-not-able-to-restart

### 3. ModSecurity 설정
- /etc/apache2/sites-avaliable/000-default.conf
	- Apache 웹 서버에서 사이트의 구성을 정의하는 파일
	- 특정 웹사이트/도메인에 대한 설정을 정의

위 파일을 수정하여 통해 apache 웹사이트에 ModSecurity를 정의하자.
1. cd /etc/apache2/sites-avaliable
2. mosuepad 000-default.conf
	파일의 맨 아래에 다음 코드 추가
    ```shell
    SecRuleEngine On
    SecRule ARGS:testparam "@contains test" "id:9999999,deny,status:403,msg:'TEST RULE'"
    ```
3. systemctl restart apache2 입력

ModSecurity 환경 설정을 마쳤다. 
이후 
-  `curl -X GET http://127.0.0.1/?testparam=test`를 cli에 입력 
혹은 
- `http://127.0.0.1/?testparam=test`를 브라우저에 입력
하면 *403 Forbidden*(접근 거부) 페이지를 확인할 수 있다. 
![[Pasted image 20231212174644.png]]

이는 위에서 추가한 규칙 중 `testparam "@contains test" ..."id:9999999,deny,... ` 에 해당하는 것이다. 
testparam의 인자값으로 test가 입력되어 요청이 들어올 시 이를 deny하고, 403 상태값을 리턴하라는 규칙에 따라 testparam에 test를 넣으면 Forbidden된 결과값이 출력되는 것이다. 

만약 위 인자값을 'test'를 제외한 아무 값이나 넣으면 정상적인 페이지가 리턴된다. 
![[Pasted image 20231212174553.png]]


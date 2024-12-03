
.bashrc

alias


록키 리눅스 명령어
apt = dnf
nginx 설치: [nginx: Linux packages](https://nginx.org/en/linux_packages.html)
wget **https://nginx.org/download/nginx-1.26.0.tar.gz**
tar
```
Dependencies resolved.
=======================================================================================================
 Package           Architecture       Version                           Repository                Size
=======================================================================================================
Installing:
 nginx             x86_64             1:1.26.2-1.el8.ngx                nginx-stable             963 k

Transaction Summary
=======================================================================================================
Install  1 Package

Total download size: 963 k
Installed size: 3.2 M
Is this ok [y/N]: y
Downloading Packages:
nginx-1.26.2-1.el8.ngx.x86_64.rpm                                      419 kB/s | 963 kB     00:02
-------------------------------------------------------------------------------------------------------
Total                                                                  419 kB/s | 963 kB     00:02
nginx stable repo                                                       10 kB/s |  12 kB     00:01
Importing GPG key 0xB49F6B46:
 Userid     : "nginx signing key <signing-key-2@nginx.com>"
 Fingerprint: 8540 A6F1 8833 A80E 9C16 53A4 2FD2 1310 B49F 6B46
 From       : https://nginx.org/keys/nginx_signing.key
Is this ok [y/N]: y
Key imported successfully
Importing GPG key 0x7BD9BF62:
 Userid     : "nginx signing key <signing-key@nginx.com>"
 Fingerprint: 573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62
 From       : https://nginx.org/keys/nginx_signing.key
Is this ok [y/N]: y
Key imported successfully
Importing GPG key 0x8D88A2B3:
 Userid     : "nginx signing key <signing-key-3@nginx.com>"
 Fingerprint: 9E9B E90E ACBC DE69 FE9B 204C BCDC D8A3 8D88 A2B3
 From       : https://nginx.org/keys/nginx_signing.key
Is this ok [y/N]: y
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                               1/1
  Running scriptlet: nginx-1:1.26.2-1.el8.ngx.x86_64                                               1/1
  Installing       : nginx-1:1.26.2-1.el8.ngx.x86_64                                               1/1
  Running scriptlet: nginx-1:1.26.2-1.el8.ngx.x86_64         
```

nodejs : [How to Install Node.js and NPM in RHEL and Debian Systems](https://www.tecmint.com/install-nodejs-npm-in-centos-ubuntu/)
특정버전 설치 가능성 보려면 다음과 같이 입력
yum list available nodejs --showduplicates
yum install 버전에 맞느 ㄴ

npm 설치: 되어있음



### WAS

1. JDK 설치확인
	```
		java -v
	```
	[개발자생각 :: 아파치 톰캣 버전별 jdk 버전](https://programdev.tistory.com/49)

2. wget 설치
	yumlist wget
	yum install wget

3. tomcat 설치
	[How to Install Tomcat 8.5 on CentOS/RHEL 8 – TecAdmin](https://tecadmin.net/install-tomcat-8-on-centos-8/)
	[Index of /dist/tomcat/tomcat-8/v8.5.99/bin](https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.99/bin/)
	[[RHEL 9.2] 톰캣(Tomcat) 설치](https://engineeringcode.tistory.com/entry/RHEL-92-%ED%86%B0%EC%BA%A3Tomcat-%EC%84%A4%EC%B9%98)
	
1. 톰캣 스크립트(서비스 생성)


톰캣 유저로 바꿈


### DB 설치

dnf update 하고 dnf list 해서 설치 가능한 거 있는지 확인한 다음에 dnf 로 설치
그런데 10.3밖에 없으니까, 일단 로컬에 설치한  다음에 이거를 mobaxterm으로 옮겨보자(안되면말고)
[CentOS MariaDB 오프라인 수동 설치 방법 (tar.gz 파일)](https://wildeveloperetrain.tistory.com/341)
wget으로 설치(wget https://downloads.mariadb.org/f/mariadb-10.6.4/bintar-linux-systemd-x86_64/mariadb-10.6.4-linux-systemd-x86_64.tar.gz)여기서 버전만 수정

어쩔 수 없다! bastion host로 파일 전송 후 scp 명령어 씀
scp -P 포트번호 /path/to/local/file cloud-user@172.25.2.70:/home/cloud-user
[CentOS MariaDB 오프라인 수동 설치 방법 (tar.gz 파일)](https://wildeveloperetrain.tistory.com/341)

유저 처음 설정은 mysql로 함

mysql 명령엉가 안먹음 -> 환경설정 할 때 경로가 꼬였음. 링크 다시 설정 해 줌
그랬더니 libncurses 에러 발생
[mariadb 10.4.18 설치시 libncurses.so.5 에러문제 해결 - 쇠똥구리 DBA's Work and Life Balance](https://dungbeetle.co.kr/mariadb-10-4-18-%EC%84%A4%EC%B9%98%EC%8B%9C-libncurses-so-5-%EC%97%90%EB%9F%AC%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0/)



리눅스 종류좀 공부하자

netstat은 net-tools임.


### WEB-WAS연결
/etc/nginx/conf.d/default.conf : 개별 서버 블록(server block, 가상 호스트)을 정의하기 위한 설정 파일

/etc/nginx/nginx.conf : NGINX의 **전체 설정 파일**

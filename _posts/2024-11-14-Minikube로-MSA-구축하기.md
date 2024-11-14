---
layout: post
title: Minikube로 MSA 실습하기
author: 박지수
categories: K-PaaS
tags: 클라우드 K-PaaS
---

## 1. MiniKube란?
- kubernetes 클러스터를 관리하는 데 도움이 되는 오픈소스
- 로컬 환경에서 쿠버네티스 클러스터 환경을 단일 워커 노드로 구현하여 사용할 수 있는 도구


## 2. 실습 목표
![image](https://github.com/user-attachments/assets/1de0ae6d-e3e6-49c4-8b93-09c609d68563)
회원가입, 게시판, 댓글 기능이 있는 애플리케이션을 MSA 구조로 구성한다.  
이를 통해 MSA의 원리와 동작 방식에 대해 이해한다.


## 3. 실습
실습은 가상머신을 통해 진행한다. 가상머신을 설치하는 방법은 다음 페이지를 참고한다. [리눅스 시작하기]([https://jisoo449.github.io/%EB%A6%AC%EB%88%85%EC%8A%A4/2023/11/10/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0.html)

### 3.1. 환경설정
1. 가상머신에 vim, git, curl 설치
   ```
   $ sudo apt-get update
   $ sudo apt-get install -y curl vim git
   ```
2. Minikube 설치
   ```
   $ wget https://github.com/kubernetes/minikube/releases/download/v1.32.0/minikube-linux-amd64
   $ sudo cp minikube-linux-amd64 /usr/local/bin/minikube
   $ sudo install minikube-linux-amd64 /usr/local/bin/minikube

   # 버전확인
   $ minikube version
   ```
   ![image](https://github.com/user-attachments/assets/ff5d55c9-cb07-4182-940b-eeffb40b5348)
3. Kubectl 설치
   ```
   $ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.29.2/bin/linux/amd64/kubectl
   $ chmod +x ./kubectl
   $ sudo mv ./kubectl /usr/local/bin/kubectl

   # 버전 확인
   $ kubectl version -o json --client
   ```
   ![image](https://github.com/user-attachments/assets/33a09eac-64ef-4b38-8e0a-611c4d1dc249)
4. MSA 배포 파일 설치
   ```
   $ cd ~
   $ git clone https://github.com/K-PaaS/edu-msa-file.git
   ```
5. Podman 설치
   ```
   $ cd ~/edu-msa-file
   $ chmod +x podman-install.sh
   $ sudo ./podman-install.sh
   $ sudo rm -rf ~/.local/share/containers/
   ```
6. /etc/containers/ 경로의 registries.conf, policy.json 수정
   ```
   $ cd /etc/containers/
   $ sudo rm registries.conf
   $ sudo vim registries.conf
   
   ####### 아래 내용 입력
   [registries.search]
   registries = ["docker.io"]


   $ sudo rm policy.json
   $ sudo vim policy.json

   ######### 아래 내용 입력
   { "default": [{ "type": "insecureAcceptAnything" }] }

   $ sudo podman system reset
   ```

### 3.2. 클러스터 구축
1. Podman 드라이버를 사용해 Minikube 실행
   ```
   minikube start --driver=podman --container-runtime=cri-o
   ```
   ![image](https://github.com/user-attachments/assets/a5595edc-c687-4b27-a5b3-cdf8393f86d4)

2. Minikube 배포 확인
   ```
   kubectl get node
   kubectl get pods -n kube-system
   ```
   ![image](https://github.com/user-attachments/assets/f7145100-a01a-43e1-bc6f-fb651918b1ee)

3. CoreDNS 스크립트 적용  
   CoreDNS란? 쿠버네티스와 같은 분산 시스템 환경에서 DNS 서비스를 제공하는 DNS 서버  
   아래의 명령어를 입력하여 쿠버네티스 내부 통신을 위한 CoreDNS 스크립트를 적용하고, 적용 여부를 확인하라.  
   ```
   cd ~/edu-msa-file
   chmod +x coredns-apply.sh
   ./coredns-apply.sh
   kubectl get cm coredns -n kube-system -o yaml
   ```
   ![image](https://github.com/user-attachments/assets/f027ad66-7b76-41ab-9735-fa0a6073b674)

4. ~/edu-msa-file/Kubernetes 경로에 있는 redis-msa-ui.yaml, mysql-msa-board.yaml, mysql-msa-comment.yaml, mysql-msa-user.yaml 파일 수정 후 kubectl apply 명령어로 파드 실행  
   아래 포트번호를 참고하여 각 파일들을 수정하라.
   |애플리케이션|변수|포트번호|
   |----|-----|------|
   |edu-msa-ui|${EDU_MSA_UI}|30001|
   |edu-msa-zuul|${EDU_MSA_ZUUL}|30101|
   |edu-msa-board|${EDU_MSA_BOARD}|30201|
   |edu-msa-comment|${EDU_MSA_COMMENT}|30301|
   |edu-msa-user|${EDU_MSA_USER}|30401|
   |mysql-msa-board|${MYSQL_MSA_BOARD}|30501|
   |mysql-msa-comment|${MYSQL_MSA_COMMENT}|30601|
   |mysql-msa-user|${MYSQL_MSA_USER}|30701|
   |redis-msa-ui|${REDIS_MSA_UI]|30801|
   ```
   cd ~/edu-msa-file/Kubernetes
   vi 각 파일

   kubectl apply -f mysql-msa-board.yaml -f mysql-msa-comment.yaml -f mysql-msa-user.yaml -f redis-msa-ui.yaml
   ```
5. 파드 및 서비스 실행 확인
   ```
   kubectl get pod,svc
   ```
   ![image](https://github.com/user-attachments/assets/5f4f355a-5421-40c4-8bdb-704eaf3cc5d5)

### 3.3. DB 구축
아래 명령어를 입력하여 각 db 파드 내에 접속한다.
 ```
 kubectl exec -it 파드이름 -- mysql -u root --password=password
 ```
이후 다음 순서에 맞게 각 DB에 컬럼을 추가한다.

1. Board DB 접속 및 구축
   ```
   CREATE DATABASE msa_board default CHARACTER SET UTF8;
   USE msa_board;
   CREATE TABLE `TB_BOARD` (
     `board_seq` int(11) NOT NULL AUTO_INCREMENT,
     `board_title` varchar(50) COLLATE utf8_unicode_ci NOT NULL,
     `board_text` mediumtext 2COLLATE utf8_unicode_ci,
     `write_user_id` varchar(20) COLLATE utf8_unicode_ci NOT NULL,
     `write_user_name` varchar(20) COLLATE utf8_unicode_ci NOT NULL,
     `use_yn` varchar(1) COLLATE utf8_unicode_ci DEFAULT 'Y',
     `create_dt` datetime NOT NULL,
     `update_dt` datetime NOT NULL,
     PRIMARY KEY (`board_seq`)
   ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8
   COLLATE=utf8_unicode_ci;
   ```
2. Comment DB 구축
   ```
   CREATE DATABASE msa_comment default CHARACTER SET UTF8;
   USE msa_comment;
   CREATE TABLE `TB_COMMENT` (
     `comment_seq` int(11) NOT NULL AUTO_INCREMENT,
     `board_seq` int(11) NOT NULL,
     `comment` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
     `write_user_id` varchar(20) COLLATE utf8_unicode_ci NOT NULL,
     `write_user_name` varchar(20) COLLATE utf8_unicode_ci NOT NULL,
     `use_yn` varchar(1) COLLATE utf8_unicode_ci DEFAULT 'Y',
     `create_dt` datetime NOT NULL,
     `update_dt` datetime NOT NULL,
     PRIMARY KEY (`comment_seq`)
   ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 
   COLLATE=utf8_unicode_ci;
   ```
3. User DB 구축
   ```
   CREATE DATABASE msa_user default CHARACTER SET UTF8;
   USE msa_user;
   CREATE TABLE `TB_USER` (
    `user_seq` int(11) NOT NULL AUTO_INCREMENT,
    `user_id` varchar(20) COLLATE utf8_unicode_ci NOT NULL,
    `user_passwd` varchar(256) COLLATE utf8_unicode_ci NOT NULL,
    `user_name` varchar(20) COLLATE utf8_unicode_ci NOT NULL,
    `use_yn` varchar(1) COLLATE utf8_unicode_ci DEFAULT 'Y',
    `create_dt` datetime NOT NULL,
    `update_dt` datetime NOT NULL,
    PRIMARY KEY (`user_seq`),
    UNIQUE KEY `id_key` (`user_id`)
   ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 
   COLLATE=utf8_unicode_ci
   ```

### 3.4. WAR 파일 생성 및 배포
본 작업은 가상머신이 아닌 윈도우에서 실행한다.  
아래 링크에 접속하여 edu-msa-file, edu-msa-zuul, edu-msa-user, edu-msa-ui, edu-msa-comment, edu-msa-board를 클론한다.  
[K-PaaS 깃헙 링크](https://github.com/K-PaaS?q=edu&type=all&language=&sort=)
![image](https://github.com/user-attachments/assets/f2904d3d-c76a-40f3-b65c-cef3877a5a87)
아래 순서를 따라 각 프로젝트의 war 파일을 생성한다.  
1. 이클립스에서 프로젝트 열기
2. edu-msa-[board, comment, user] 설정 파일 수정(3개)  
   src/main/resources/properties/api.properties 의 경로로 들어가 db 연결 정보를 수정하라.
   ![image](https://github.com/user-attachments/assets/70bde4d8-1f80-419e-8b80-3b0d24e53d1b)
   dbms.url은 가상머신에 접속하여 `minikube ip`를 입력해 나온 결과를 입력하라.
   ![image](https://github.com/user-attachments/assets/6e8f0a23-216f-4e70-8f65-874970fc730b)
3. 모든 프로젝트에 대해 war 파일 생성(board, comment, ui, user, zuul)
   ![image](https://github.com/user-attachments/assets/7fd75a71-a300-4c17-8fc2-8ef7d65e00bb)
   ![image](https://github.com/user-attachments/assets/3bb52e29-cb28-4ea2-bb43-ced0ef54807f)
4. 생성된 war 파일을 mobaXterm을 통해 `~/edu-msa-file/Docker/`로 이동
5. 도커 허브에 레포지토리 생성
6. 레포지토리 이름과 일치하게 도커 이미지 생성 및 업로드
   ```
   podman build --tag edu-msa-board:latest .
   podman tag edu-msa-board ${도커허브 ID}/edu-msa-board
   podman push ${도커허브 ID}/edu-msa-board
   ```
7. `~/edu-msa-file/Kubernetes` 경로에 있는 edu-msa-ui.yaml, edu-msa-zuul.yaml, edu-msa-board.yaml, edu-msa-comment.yaml, edu-msa-user.yaml 파일을 수정 후 apply 명령어를 통해 실행하라.
   아래 포트번호를 참고하여 각 파일들을 수정하라.
   |애플리케이션|변수|포트번호|
   |----|-----|------|
   |edu-msa-ui|${EDU_MSA_UI}|30001|
   |edu-msa-zuul|${EDU_MSA_ZUUL}|30101|
   |edu-msa-board|${EDU_MSA_BOARD}|30201|
   |edu-msa-comment|${EDU_MSA_COMMENT}|30301|
   |edu-msa-user|${EDU_MSA_USER}|30401|
   |mysql-msa-board|${MYSQL_MSA_BOARD}|30501|
   |mysql-msa-comment|${MYSQL_MSA_COMMENT}|30601|
   |mysql-msa-user|${MYSQL_MSA_USER}|30701|
   |redis-msa-ui|${REDIS_MSA_UI]|30801|

<br/><br/>
위 과정을 완료했다면 아래와 같이 사이트에 접속할 수 있다
![image](https://github.com/user-attachments/assets/9abec97d-79cd-4a8b-8b38-61925ee3cc08)
![image](https://github.com/user-attachments/assets/c5659e53-68a9-4232-82b0-939d020b2a94)
![image](https://github.com/user-attachments/assets/e5906aec-65b1-4dd1-90bd-b944bb9e82a8)
![image](https://github.com/user-attachments/assets/3b5bcc07-4016-4553-be58-e1b9e0813ee8)
![image](https://github.com/user-attachments/assets/4530cf60-2d08-45ef-acc6-24f4725cd429)


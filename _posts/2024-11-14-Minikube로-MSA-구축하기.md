---
layout: post
title: Minikube로 MSA 실습하기
author: 박지수
categories: K-PaaS
tags: 클라우드, K-PaaS
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
   ```
3. Kubectl 설치
   ```
   $ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.29.2/bin/linux/amd64/kubectl
   $ chmod +x ./kubectl
   $ sudo mv ./kubectl /usr/local/bin/kubectl
   $ kubectl version -o json --client
   ```
4. Podman 설치
   ```
   $ cd ~/edu-msa-file
   $ chmod +x podman-install.sh
   $ sudo ./podman-install.sh
   $ sudo rm -rf ~/.local/share/containers/
   ```
5. Podman 환경설정
   ```
   $ cd /etc/containers/
   $ sudo rm registries.conf
   $ sudo vim registries.conf
   
   ####### 아래 내용 입력
   { "default": [{ "type": "insecureAcceptAnything" }] }

   $ sudo podman system reset
   ```

### 3.2. 클러스터 및 DB 구축
1. Podman 드라이버를 사용해 Minikube 실행
   ```
   minikube start --driver=podman--container-runtime=cri-o
   ```

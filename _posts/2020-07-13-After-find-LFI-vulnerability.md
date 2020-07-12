---
layout: post
title:  "After find LFI vulnerability"
date:   2020-07-13 01:02:00
---

### /etc/passwd

```
Leak 가능성 여부를 판별할 때 가장 많이 사용하며 리눅스 사용자 정보가 들어있는 시스템 기본 파일이다.
```



### /etc/hosts

```
리눅스 호스트 정보가 들어있는 시스템 기본 파일로서 도커 네트워크 구성 시 네이밍 된 서버들 또한 호스트 파일에 기재되어 있다. 주로 SSRF 등 내부 확산 시 유용하게 참고될 수 있다.
```



### /etc/\<service\>/\<service\>.conf

```
현재 운영되고 있는 서비스의 설정정보를 확인할 수 있다.
```



### /home/\<user\>/.\<shell\>_history

```
사용자의 명령어 히스토리가 저장된 기본 파일이다. 
```



### /home/\<user\>/.ssh/known_hosts

```
OpenSSH Stric key check를 한 호스트로 SSRF등 내부 확산 시 유용하게 참고될 수 있다.
```



### /home/\<user\>/.ssh/id_rsa

```
OpenSSH 서비스 운영 시 공개키를 통한 원격 접속을 사용하기 위해 키를 발행할 때 RSA형식의 디폴트 개인키 네임이다.
```



### /proc/cpuinfo

```
시스템 기본 파일로 시스템의 CPU 정보를 확인할 수 있다.
```



### /proc/self/cmdline

```
현재 로드된 프로세스의 실행 프로그램과 인자를 알 수 있다.
```



### /proc/self/environ

```
현재 프로세스가 실행될 때 로드된 시스템 환경변수를 확인할 수 있다.
```



### /proc/self/maps

```
프로세스의 메모리 맵을 확인할 수 있다.
```



### /proc/self/fd/\<seq\>

```
파일 디스크립터를 사용하는 명령어 또는 함수를 사용했을 때 로드된 파일을 읽을 수 있다.
```



### /proc/self/maps

```
프로세스가 사용하는 메모리 영역 내 메모리 주소와 맵핑된 라이브러리 등을 확인할 수 있다.
```



### /proc/self/mem

```
일반적으로 통으로 읽을 수는 없고 Range 헤더 컨트롤이 가능할 경우 범위를 지정하여 maps를 통해 확인된 주소값을 덤프할 수 있다.
```



### /var/run/secrets/kubernetes.io/serviceaccount/token

```
kubernetes 환경에서 Access token을 획득할 수 있다. 토큰 획득 후 Kubernetes API에 접근이 가능할 경우 API내 접근 가능한 Pods 또는 ConfigMap등을 확인할 수 있다.
```


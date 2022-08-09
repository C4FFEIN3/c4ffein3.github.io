---
layout: post
title: "multi-node HDFS 설치 및 실행"
subtitle: "ubuntu 20.04에서 Hadoop Distributed File System (HDFS) 3.2.3 버전 설치 및 실행하기"
categories: hadoop-ecosystem
tags: [linux, ubuntu, hadoop-ecosystem, hdfs]
---
## Overview

구성하고자 하는 HDFS 구조

![Untitled](https://user-images.githubusercontent.com/57282971/183603076-6fdfa7e0-5388-4db3-b1cb-c8be5b7cc1eb.png)

## java 설치

### openjdk-8-jdk 다운로드

```bash
sudo apt install openjdk-8-jdk
```

## /etc/hosts 수정

/etc/hosts는 IP와 hostname을 매핑하기 위한 파일

멀티 노드에 접속할 때의 편의를 위해 /etc/hosts 파일에 내용을 추가한다.

```bash
sudo vi /etc/hosts
```

![Untitled](https://user-images.githubusercontent.com/57282971/183603078-7c540526-734f-45b5-ba6c-c7ef88569379.png)

아래와 같이 다른 노드의 IP와 hostname을 지정하는 내용을 추가한다.

![Untitled](https://user-images.githubusercontent.com/57282971/183603084-1ae94810-b702-43a3-9cb5-b1dfd7284b59.png)

## hadoop 사용자 생성

멀티 노드로 사용하고자 하는 모든 노드들에서 하둡용 계정을 생성

하둡 설치 및 관련 설정은 모두 하둡용 계정에서 실행

```bash
sudo adduser hadoop
```

계정에 로그인하기 위해서 아래 명령어를 사용한다.

```bash
su - hadoop
```

## ssh key 설정

기본적으로 하둡은 ssh를 이용하여 노드간 통신을 하므로, ssh 통신에 사용할 키를 생성해줘야 한다.

그리고 네임노드에서 데이터노드의 생성된 key를 이용해 접속할 수 있도록 키를 등록해줘야 한다.

- **ssh key 생성**: 모든 노드에서 실행

```bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_key
```

실행시 ~/.ssh/ 폴더에 id_rsa.pub이라는 이름의 key가 생성된다.

- **각 노드에서 생성된 ssh key 등록**: 네임노드에서 실행

```bash
ssh hadoop@noslab-ssd2 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
ssh hadoop@noslab-gpu cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
ssh hadoop@noslab-gpu2 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

## 하둡 파일 다운로드

### 하둡 파일 다운로드 및 압축해제

멀티 노드로 사용하고자 하는 모든 노드들에 하둡 설치 

```bash
su - hadoop
wget https://downloads.apache.org/hadoop/common/hadoop-3.2.3/hadoop-3.2.3.tar.gz
tar -zxf hadoop-3.2.3.tar.gz
cd hadoop-3.2.3
```

## 하둡 설정 파일 설정

**{hadoop의 root directory}/etc/hadoop**에 존재

- **hadoop-env.sh**: JAVA_HOME 경로 설정

![Screenshot from 2022-08-04 09-09-20.png](https://user-images.githubusercontent.com/57282971/182780118-14038a90-0222-4bc2-8ee0-9c81a8bd71dc.png)

- **core-site.xml**: 클러스터의 네임노드에서 실행되는 하둡 데몬 설정
    
    하둡 파일 시스템 이름 설정 (URI 형식으로 입력)
    

```xml
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://10.100.54.177:9000</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>file:///home/hadoop/hadoop-3.2.3/tmp</value>
	</property>
</configuration>
```

*fs.defaultFS*의 값은 *hdfs://{namenode의 IP}:9000*으로 지정

tmp 디렉터리의 경우 지정한 위치에 디렉터리를 생성해줄 필요

- **hdfs-site.xml**: 네임노드와 데이터노드 저장 경로를 지정, 데이터 복제 개수를 설정

```xml
<configuration>
	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:///home/hadoop/hadoop-3.2.3/namenode</value>
	</property>
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///home/hadoop/hadoop-3.2.3/datanode</value>
	</property>
	<property>
		<name>dfs.http.address</name>
		<value>10.100.54.177:50070</value>
	</property>
</configuration>
```

*dfs.replication*: 데이터 복제 개수

*dfs.namenode.name.dir*: 네임노드의 디렉터리 경로

*dfs.datanode.data.dir*: 데이터노드의 디렉터리 경로

*dfs.http.address*: webUI를 통해서 dfs를 확인하고자 할 때의 address

- **mapred-site.xml**: 맵리듀스 설정
    
    기본 맵리듀스 프레임워크로 yarn을 설정
    

```xml
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
</configuration>
```

- **yarn-site.xml**: YARN 설정

```xml
<configuration>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>10.100.54.177</value>
	</property>
</configuration>
```

*yarn.resourcemanager.hostname*의 값으로는 namenode의 IP를 입력해야함

- **workers**: datanode 설정

```
noslab-ssd2
noslab-gpu
noslab-gpu2
```

각 라인에 datanode의 hostname을 입력

- **모든 파일 설정이 끝난 후에는 나머지 노드로 설정 파일을 복사**

```bash
#위의 파일 수정을 namenode에서 했다고 가정했을 때
scp ~/hadoop-3.2.3/etc/hadoop/* hadoop@noslab-ssd2:~/hadoop-3.2.3/etc/hadoop/
scp ~/hadoop-3.2.3/etc/hadoop/* hadoop@noslab-gpu:~/hadoop-3.2.3/etc/hadoop/
scp ~/hadoop-3.2.3/etc/hadoop/* hadoop@noslab-gpu2:~/hadoop-3.2.3/etc/hadoop/
```

## .bashrc 수정

```bash
vi ~/.bashrc
```

~/.bashrc를 열어 최하단에 아래 내용 추가

```bash
export HADOOP_HOME=/home/hadoop/hadoop-3.2.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
```

수정한 내용을 저장 후 적용

```bash
source ~/.bashrc
```

## HDFS 네임노드 포맷

```bash
{hadoop의 root directory}/bin/hdfs namenode -format
```

## HDFS 실행

### DFS 실행

```bash
{hadoop의 root directory}/sbin/start-dfs.sh
```

### YARN 실행

```bash
{hadoop의 root directory}/sbin/start-yarn.sh
```

### 전체 실행

- DFS와 YARN을 한 번에 실행하기 위한 스크립트

```bash
{hadoop의 root directory}/sbin/start-all.sh
```

![Untitled](https://user-images.githubusercontent.com/57282971/183603087-1d7b9617-a5b2-4314-9e65-0267a623a81f.png)

### 실행 확인

- **jps**: Java virtual machine Process Status tool. JVM에서 실행 중인 프로세스를 확인하기 위한 명령어

```bash
jps
```

jps 실행시 namenode에 표시되는 프로세스:

![Untitled](https://user-images.githubusercontent.com/57282971/183603093-e2367177-8592-4d88-a611-3134f88e74eb.png)

jps 실행시 datanode에 표시되는 프로세스:

![Untitled](https://user-images.githubusercontent.com/57282971/183603095-1b323732-9962-4af9-b70f-a5f978ef2d7f.png)

- **http://{namenode의 IP}:50070**으로 접속할 경우 hdfs의 상태를 web UI로 확인 가능

![Untitled](https://user-images.githubusercontent.com/57282971/183603100-24f85233-3cf8-46c5-8986-5b262215ef5f.png)

Live Nodes의 수가 3개로 뜨는 것을 확인할 수 있다.

Configured Capacity의 경우 node들의 capacity 총합을 보여준다.

- **http://{namenode의 IP}:8088**으로 접속할 경우 yarn의 resource manager를 web UI로 확인 가능

![Untitled](https://user-images.githubusercontent.com/57282971/183603101-3d6f0372-5df2-4661-bc80-4a5313fe727e.png)

Nodes를 누르면 아래 화면을 볼 수 있다.

![Untitled](https://user-images.githubusercontent.com/57282971/183603105-99a8b0f0-7f0b-4073-9ec1-ac804fbab0a0.png)

설정한 노드들이 모두 정상적으로 작동하고 있는 것을 볼 수 있다.

### 종료

```bash
{hadoop의 root directory}/sbin/stop-all.sh
```

## 발생했던 문제

- 하둡 설정 파일에서 네임노드의 IP로 설정하는 대신 네임노드의 호스트네임으로 설정하는 경우(*e.g.* 10.100.54.177 대신 mscho-ubuntu 사용) 에러가 발생해서 데이터 노드와의 연결이 되지 않았음

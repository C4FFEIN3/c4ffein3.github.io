---
layout: post
title: "리눅스 커널 소스코드 컴파일"
subtitle: "ubuntu 20.04에서 리눅스 커널 컴파일 및 설치 하기"
categories: linux-kernel
tags: [linux, ubuntu, kernel]
---
## 커널 소스코드 다운로드

### 현재 커널 버전 확인

```bash
uname -r
```

### 커널 소스코드 다운로드

리눅스 커널 아카이브: [https://www.kernel.org/](https://www.kernel.org/)

희망하는 버전의 커널 소스코드를 tarball을 눌러 다운로드

![Untitled](https://user-images.githubusercontent.com/57282971/183005698-342c20f1-536b-4bd8-b7d1-318cbbea5255.png)

### 소스코드 위치 이동

다운로드 받은 소스코드 압축 파일을 /usr/src 경로로 이동

ex) *~/Downloads*에 *linux-5.15.21.tar.xz* 파일을 다운로드 받은 경우

```bash
cd ~/Downloads
mv linux-5.15.21.tar.xz /usr/src
```

### 소스코드 압축 해제

/usr/src로 이동시킨 파일을 압축해제

```bash
cd /usr/src
# tar -xvf {압축된 커널 소스파일}
tar -xvf linux-5.15.21.tar.xz
```

## Dependencies 설치

커널 컴파일에 필요한 패키지들을 설치

```bash
# 추후 컴파일 중 dependency 문제로 에러 발생할 수 있음. 발생시 해당 dependency를 추가로 설치 필요
sudo apt-get install build-essential libncurses5 libncurses5-dev bin86 kernel-package libssl-dev bison flex
```

## kernel configuration 파일 설정

make config, make menuconfig 중 원하는 명령어를 이용하여 커널 설정을 할 수 있다.

make config의 경우 텍스트 기반이며, make menuconfig의 경우 GUI 기반이다.

- **make menuconfig**

![Untitled](https://user-images.githubusercontent.com/57282971/183005706-943ac3e6-51ab-4e2b-a07e-1f987b1fa33a.png)

설정이 완료되면 <Save>를 선택하여 설정 파일을 생성하고, <Exit>으로 빠져나오면 된다.

- **현재 설치된 리눅스의 설정 파일을 사용하고자 하는 경우**

복잡한 설정을 하는 대신 기존의 설정 파일을 재사용하고자 하는 경우에는 기존 리눅스의 설정 파일을 불러와서 설정으로 사용할 수 있다.

1. **현재 커널 버전 확인**

```bash
uname -r
```

2. **기존 커널 설정 파일을 설치하고자 하는 커널 소스파일 디렉토리에 .config 파일로 복사한다.**

ex) 현재 커널 버전이 5.13.0-28-generic이고 설치하고자 하는 커널 소스가 5.15.21인 경우

```bash
#sudo cp /boot/config-{현재 커널 버전} /usr/src/{설치하고자 하는 커널 소스 디렉토리명}/.config
sudo cp /boot/config-5.13.0-28-generic /usr/src/linux-5.15.21/.config
```

3. **menuconfig를 이용해 복사한 설정 파일의 설정을 적용한다.**

![Untitled](https://user-images.githubusercontent.com/57282971/183005706-943ac3e6-51ab-4e2b-a07e-1f987b1fa33a.png)

4. **menuconfig의 화면에서 <Load>를 선택한 후 불러올 설정 파일의 이름을 입력한다.**

![Untitled](https://user-images.githubusercontent.com/57282971/183005708-23026c6e-5c8b-41bb-9bf8-9421bc98b479.png)

5. **그 후 <Ok>를 누르고 나와서 <Save>를 선택해 설정을 저장하고 <Exit>으로 종료한다.**

## 커널 컴파일 및 설치

커널 컴파일에는 아래의 2가지 방법이 존재한다.

1. make-kpkg를 이용해서 커널 이미지 생성 및 dpkg를 이용한 설치
2. make modules(모듈 컴파일), make(커널 컴파일), make modules_install(컴파일된 모듈 설치), make install(컴파일된 커널 설치)를 통해 커널 컴파일 및 설치

- **make-kpkg를 이용해서 커널 이미지 생성 및 dpkg를 이용한 설치**
1. **커널 컴파일**

```bash
sudo make-kpkg -j 24 --initrd --revision=1.0 kernel_image
```

-j 옵션의 경우 멀티 스레드 옵션으로, 빠른 컴파일을 위하여 필요한 옵션이다. 일반적으로 cpu 코어 개수를 입력한다. cpu 코어 개수의 경우 아래의 명령어를 통해서 확인 가능하다.

```bash
grep -c processor /proc/cpuinfo
```

컴파일 완료시 상위 폴더인 /usr/src에서 .deb 파일로 생성된 커널 이미지를 확인할 수 있다.

![Untitled](https://user-images.githubusercontent.com/57282971/183005712-d8a25d1d-07ec-434d-b9a0-8aab2b89c8d0.png)

2. **커널 이미지 설치**

```bash
# sudo dpkg -i {커널 이미지 파일명}
sudo dpkg -i linux-image-5.15.21_1.0_amd64.deb
```

- **make를 이용해서 커널 컴파일 및 설치**

```bash
sudo make modules -j 24
sudo make -j 24
sudo make modules_install
sudo make install
```

make의 -j 옵션 또한 멀티 스레드 옵션으로 빠른 컴파일을 위해 필요하다. 

## Reboot

설치가 완료되면 재부팅 후 커널 버전을 확인하여 설치가 무사히 완료되었는지 확인할 수 있다.

```bash
uname -r
```

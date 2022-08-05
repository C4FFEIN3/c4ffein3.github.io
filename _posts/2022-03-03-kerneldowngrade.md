---
layout: post
title: "리눅스 커널 다운그레이드"
subtitle: "리눅스 20.04에서 리눅스 커널 다운그레이드 하기"
categories: linux-kernel
tags: [linux, ubuntu, kernel]
---

기본적으로 리눅스는 부팅 시에 가장 높은 버전의 리눅스 커널을 사용하도록 되어있다. 만약 사용하고자 하는 커널 버전보다 높은 버전의 커널이 설치되어있는 경우, 매번 부팅 시마다 grub으로 접근해서 커널 버전을 선택해야 하거나 grub 설정을 수정해서 부팅 순서를 바꿔야 할 필요가 있다.

grub 설정을 건드리고 싶지 않은 경우, 상위 버전의 리눅스 커널을 삭제하는 방법이 있다.

## dpkg, apt를 이용한 상위 버전 리눅스 커널 삭제

### 설치된 커널 이미지 확인

```bash
dpkg -l | grep linux-image
```

### 커널 이미지 삭제

```bash
sudo apt-get purge {삭제할 리눅스 이미지 파일명} 
```

### 설치된 커널 헤더 확인

```bash
dpkg -l | grep linux-headers
```

### 커널 헤더 삭제

```bash
sudo apt-get purge {삭제할 리눅스 헤더 파일명}
```

## /boot 파일 수정을 통한 커널 다운그레이드

일반적으로는 위에서 설명한 커널 이미지와 커널 헤더 파일 삭제만으로 다운그레이드가 가능하다. 아래는 위의 과정을 거쳤는데도 정상적으로 다운그레이드가 되지 않는 경우에 지정된 버전을 제외한 다른 버전 커널의 리눅스의 부팅 파일을 삭제하고 grub을 업데이트하여 다운그레이드를 하는 방법이다.

아래의 bash 쉘 스크립트**(리눅스 20.04, 커널 버전 5.15.21 기준)**를 파일에 저장하여 실행시키는 것으로 사용 가능하며, 설명을 읽고 수정하여 사용하여야 한다.

```bash
#!/bin/bash

#아래 두 코드는 주석 처리하거나 삭제할 것.
#크리티컬한 부분을 수정하는 스크립트이므로
#파일 내용을 읽지 않고 실행시켜 문제가 발생하지 않도록 작성되었음.
echo "please read the file before execution"
exit 1

#vmlinuz, initrd.img, grub이 최신 커널 버전으로 되어있는 것을 강제로 다운그레이드 하는 방법
#기존 커널 버전의 관련 파일들을 모두 삭제하고 vmlinuz, initrd.img, grub을 다시 세팅

#다운그레이드한 커널 버전
#config-{버전 명}, initrd.img-{버전 명}에 들어가는 버전 명을 아래에 넣을 것
#$version 이외의 버전의 설정 파일은 다 삭제 예정
version="5.15.21"

#initrd.img 관련 파일 전부 삭제
rm `ls /boot/initrd.img* | grep -v $version`
#initrd.img 심볼릭 링크 생성
ln -s /boot/initrd.img-$version /boot/initrd.img

#vmlinuz 관련 파일 전부 삭제
rm `ls /boot/vmlinuz* | grep -v $version`
#vmlinuz 심볼릭 링크 생성
ln -s /boot/vmliuz-$version /boot/vmlinuz

#grub config파일 재설정
#grub-mkconfig -o {grub.cfg 위치}
#grub.cfg의 위치의 경우 기존 grub.cfg 파일 위치를 확인 후 그곳으로 설정
grub-mkconfig -o /boot/grub/grub.cfg

#커널 버전에 따라 각 파일들의 위치가 다를 수 있으니 필수적으로 확인 후 실행할 것
```

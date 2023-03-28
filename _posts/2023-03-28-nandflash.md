---
layout: post
title: "Nand flash memory"
subtitle: "Nand flash memory의 특징"
categories: storage
tags: [hardware, fileIO]
---

# Nand flash memory
- 전기적으로 작동하는 비휘발성 메모리로, 전기적으로 지우고 리프로그래밍 할 수 있음
- 흔히 SATA SSD, NVMe SSD와 같은 고성능 스토리지
    - 그 외에도 SD 카드, USB 플래시 드라이브 같은 것들이 존재

## 하드웨어 구조
![ssd](https://user-images.githubusercontent.com/57282971/228248628-93749fad-3fff-49ce-a2a5-4b554d5d9a95.png)

SSD의 내부 사진 (출처: http://www.storagereview.com/samsung_ssd_840_pro_review)

- PC의 CPU와 같은 역할을 하는 SSD 컨트롤러
- 캐시 메모리 역할을 하는 DRAM
- 데이터 저장을 하는 낸드 플래시 메모리 
    - Die
      - 낸드 플래시 구성요소의 가장 높은 레벨

![die](https://user-images.githubusercontent.com/57282971/228248835-c552d9d6-3ed1-4313-8a74-0c9af138993c.png)

![block](https://user-images.githubusercontent.com/57282971/228248995-139a0844-ccbe-4ec1-a838-f0017d68452a.png)

- 읽기/쓰기 최소 단위인 페이지는 낸드 플래시 메모리마다 다르며 2KB, 4KB, 8KB, 16KB 중 하나
- block은 128 혹은 256 페이지로 구성

### 접근 방식
|하드디스크|플래시 메모리|
|---|---|
|![hdd](https://user-images.githubusercontent.com/57282971/228249117-244b954e-dd3f-4917-8852-a942636fffa2.png)<br>HDD의 구조 그림 (출처: https://www.britannica.com/technology/hard-disk#/media/1/1521171/68191)<br>|![ssd](https://user-images.githubusercontent.com/57282971/228248628-93749fad-3fff-49ce-a2a5-4b554d5d9a95.png)|
|자기 원반이 내부에 여러 개 들어 있고, 이 원반이 고속으로 **회전**하며 읽기/쓰기 처리|회로를 통한 **전기적인 접근**|
|**회전 구조로 인해서 물리적인 접근 시간**이 필요 → 느림|물리적인 접근 시간을 필요로 하는 하드디스크보다 빠름<br> -> 랜덤 액세스에 유리한 것도 이러한 접근 방식 덕분|

### 왜 "NAND" 플래시 메모리인가?
- NAND 게이트를 사용하므로 NAND 플래시 메모리

![nandgate](https://user-images.githubusercontent.com/57282971/228249286-13cd59e8-565d-4c17-9d9a-39074d3b803d.png)

#### NOR 플래시 메모리는 없는가?
- NOR 플래시 메모리도 존재

||NAND 플래시 메모리|NOR 플래시 메모리|
|---|---|---|
|배열|반도체 셀이 직렬로 배열|반도체 셀이 병렬로 배열|
|읽기 속도|셀이 직렬로 배열되어 순차적으로 접근해야 하므로 **느림**|셀이 병렬로 배열되어 **빠름**|
|비트당 비용|저가|고가|
|용량|좁은 공간에 많은 셀을 집적할 수 있어 고용량화 가능|집적이 어려워 고용량화가 힘들다|
|쓰기 속도|페이지 단위 쓰기를 하므로 **빠름**|비트 단위 쓰기를 하므로 **느림**|
|접근 단위|페이지 단위 접근|비트 단위 접근|

- 고가에 저용량이므로 NOR 플래시 메모리는 잘 사용하지 않음

### SLC/MLC/TLC/QLC
- 크기는 비슷한데 SSD의 용량이 늘어나는 건 어떤 원리일까?

![slc](https://user-images.githubusercontent.com/57282971/228249466-ab3a9b7c-b9d4-4441-804b-1d9ede925b81.png)

- 하나의 셀을 여러 개의 비트로 나눠서 쓸 수 있음
  - 셀의 개수가 동일해도 용량 증가
    - 비트 수가 작을수록 안정적이고 수명이 길고 빠름
	- 비트 수가 클수록 고용량

### P/E cycle
#### 플래시 메모리의 쓰기(Programmed)와 삭제(Erase)

![pe](https://user-images.githubusercontent.com/57282971/228249592-e7956a19-eaf9-45b7-94ad-54686811ad8e.png)

- 쓰기와 삭제를 반복하면 그 과정에서 통과해야 하는 **oxide 층이 점차 마모되어 전자를 보관할 수 없는 순간**이 옴
  - 플래시 메모리에는 P/E(프로그램/삭제) 사이클이라고 불리는 **수명**이 존재

## 읽기/쓰기
### 페이지 단위 읽기/쓰기
- 512 bytes 크기를 가진 섹터(sector) 단위로 접근했던 하드디스크와 달리 플래시 메모리는 페이지(page, 주로 4 KB) 단위로 접근

### 덮어쓰기
- 플래시 메모리는 "free" 상태인 페이지에만 쓰기를 할 수 있음
- 데이터가 변경되면 덮어 쓸 수 없음
  - 데이터가 변경되면 페이지의 내용은 내부 레지스터로 복사된 후 레지스터에서 변경되어 새로운 "free" 상태의 페이지로 기록 (Read-Modify-Write)
  - 변경된 데이터가 새로운 페이지에 완전히 기록되면 원본 페이지는 "stale"(invalid)  상태가 되고 지워지기(erase) 전까지 그 상태로 존재한다
  - 지워지면 "free" 상태의 페이지가 됨

### 삭제
- "stale" 상태의 페이지는 반드시 삭제(erase) 하여서 "free" 상태로 전환될 수 있음
- 블록 단위로 실행
  - 단일 페이지 단위로 처리 불가능
  - 페이지가 포함된 블록 전체를 삭제해야 함
- 사용자가 직접 사용할 수 있는 명령이 아니며 SSD 컨트롤러가 "free" 페이지 확보를 위해 Garbage collection을 일으킬 때 사용

### Wear leveling
- NAND 플래시 메모리는 프로그램-삭제(P/E cycles) 제한이 있으므로 제한된 수명을 가짐
- 특정 블록에만 P/E가 빈번히 일어나면 해당 블록은 사용 불가능 상태가 됨 → 용량 감소
- SSD 컨트롤러에서는 이러한 상태를 막기 위하여 Wear leveling을 실행
  - 전체 블록에 P/E가 골고루 분산되도록 하는 것
  - 목적은 모든 블록이 한번에 P/E 제한에 도달하여 한꺼번에 사용 불가능 상태가 되는 것
  - 특정 블록의 위치를 옮겨서 새로 써야할 수도 있으므로 write amplification 발생

> **Write amplification** <br>
> 실제 쓰고자 하는 데이터보다 더 많은 쓰기가 발생하는 것

### Garbage collection
- 새로운 "free" 공간을 얻기 위하여 "stale" 상태의 페이지들을 정리하는 것
- GC를 수행하며 valid한 데이터를 옮기는 과정에서 write amplification 발생 → **SSD 수명에 악영향**

#### GC 수행 방법
1. 초기 상태

![gc1](https://user-images.githubusercontent.com/57282971/228249715-07aed4fb-bf86-48f0-8dd3-9fd278c4b61a.png)

2. 블록의 valid 데이터를 다른 블록에 복사

![gc2](https://user-images.githubusercontent.com/57282971/228251389-d0eaf7d6-2892-4139-a98a-7106e91e4473.png)

3. 복사를 마친 데이터 invalid 처리

![gc3](https://user-images.githubusercontent.com/57282971/228251520-8dec4f1a-4f90-43ed-b779-5ddf0221c9ff.png)

4. 비운 블록을 erase

![gc4](https://user-images.githubusercontent.com/57282971/228251633-0df76235-d6c5-415d-991e-9073e8bbaaa5.png)

#### 핫/콜드 데이터 분리
- 핫 데이터: 빈번하게 변경되는 데이터
- 콜드 데이터: 빈번하게 변경되지 않는 데이터
- 동일 블록에 **핫/콜드 데이터가 섞여있지 않는 것이 GC가 효율적**으로 작동하는 데 도움이 됨
  - 섞여있는 경우 위와 같이 콜드 데이터를 migration시켜줘야 하며 **write amplification** 증가

#### 랜덤 쓰기
- 하드 디스크와 비교하면 매우 빠른 랜덤 쓰기 성능
- SSD는 랜덤 쓰기도 괜찮다? → **괜찮지 않음!**
  - 하드 디스크보다 랜덤 쓰기 성능이 좋지만 그렇다고 해서 랜덤 쓰기가 SSD에 영향이 없는 건 아님
  - 누적될 경우 GC를 빈번하게 유발하는 원인이 되어서 수명과 성능에 악영향을 끼칠 수 있음

#### GC에서 랜덤 쓰기의 영향
- 순차 쓰기 후 GC

![seqgc](https://user-images.githubusercontent.com/57282971/228251737-0d3c74e3-a09b-4f34-98fb-3400215cd2f0.png)

- 랜덤 쓰기 후 GC

![randgc](https://user-images.githubusercontent.com/57282971/228252139-c5ea3351-530d-41fd-9ff1-10ba141058dd.png)

- 랜덤 쓰기의 경우 GC 오버헤드가 훨씬 크다

### Over-provisioning
- SSD 용량은 왜 250GB, 500GB와 같이 2의 지수승이 아닌 것이 있는가? → over-provisioning 영역 때문
- 일정 비율의 물리 블록을 SSD 컨트롤러는 볼 수 있지만 운영체제나 파일 시스템은 보지 못하도록 예약해두는 것
  - 256GB SSD의 over-provisioning 영역이 16GB이면 240GB SSD, 6GB이면 250GB SSD, 0GB이면 256GB SSD
  - 하드웨어 스펙이 다른 것이 아니라 over-provisioning 영역의 크기로 인해 용량이 달라지는 것

#### 왜 OP 영역이 필요한가?
- SSD의 **공간 사용률**은 **성능과 수명에 영향**을 끼침
  - free 공간이 충분하면 비어있는 공간을 확보하기 위해 실행되는 GC가 잘 일어나지 않음
    - GC로 인한 write amplification 감소 → nand 수명 증가
    - GC로 인한 성능 저하 발생 횟수 감소 → nand 성능 향상
  - GC가 발생하여도 충분한 free space로 간섭이 줄어들어 랜덤 쓰기 성능 향상
  - over-provisioning을 통해 **빈 공간을 확보**하여 성능 저하를 막음

**_→ OP 영역을 늘리면 nand의 랜덤 쓰기 성능 및 수명 증가_**

- SSD에는 프로그램-삭제(P/E cycles) 횟수 제한이 있음 → 제한을 초과하면 해당 셀은 쓸 수 없게 됨
- **쓸 수 없게 되는 셀**이 발생하면 over-provisioning 영역을 이용해서 커버

### TRIM
- 해당 데이터가 더 이상 사용되지 않는다는 것을 호스트에서 SSD 컨트롤러로 전달해주는 것
- NVMe에서는 "Deallocate", SCSI/SAS에서는 "unmap"이라고 부름
- 왜 필요한가?
  - SSD 컨트롤러는 삭제된 논리 블록의 주소를 알지 못함
    - 파일 시스템에게서 덮어 쓰기 명령이 전달되어야 해당 공간이 비어있다는 사실을 알게 됨
  - 이미 삭제된 블록이 valid 블록으로 보여서 GC 때 이동되며 불필요한 write ampliciation이 증가할 수 있음
  - 삭제된 블록을 알려줌으로서 불필요한 복사를 막을 수 있음
  - **하지만 최근 SSD 내부 작동 방식이 많이 최적화되며 TRIM으로인한 성능 향상이 크지 않음**
- SSD 컨트롤러, 운영체제, 파일 시스템이 모두 TRIM을 지원해야 사용 가능
  - 리눅스 커널 2.6.33 이후 버전 지원
  - ext4, XFS 등 지원

![trim](https://user-images.githubusercontent.com/57282971/228252321-b444b2ed-2bdd-42d9-b205-ce3a9a17628b.png)

## NVMe SSD
### SSD vs NVMe?
- **NVMe SSD도 SSD의 일종**
  - SSD와 전혀 다른 하드웨어가 아님
- 낸드 플래시 메모리의 특성을 똑같이 가지고 있음
  - **물리적 구성 동일**
- **인터페이스** 타입이 달라 성능 차이가 발생
  - SATA SSD: **ATA** 기반의 SSD
  - NVMe: **PCIe** 기반의 SSD
  - 컨트롤러, DRAM, 캐패시터 등 세부 기술에도 차이점이 존재하나 제일 큰 차이점은 인터페이스의 차이
- NVMe SSD는 NVMe라는 전송 프로토콜 사용하여 더 향상된 성능을 보임
  - NVMe는 **PCIe 버스**를 통해 플래시 메모리에 접근
  - **몇 만 개의 큐를 제공하여 병렬처리**할 수 있도록 하여 단일 큐에 비해 빠른 속도

#### PCIe
- 고속 직렬 컴퓨터 확장 버스 표준
- 여러 개의 레인을 가지고 있을 수 있음: x1, x2, x4...
  - x 다음에 오는 숫자만큼의 레인을 가지고 있는 것
  - x1의 경우 레인이 1개이고, 데이터를 한 싸이클에 1비트를 전송
  - x2의 경우 레인이 2개이고, 데이터를 한 싸이클에 2비트를 전송

##### SATA/PCIe 처리량
||x1|x2|x4|
|---|---|---|---|
|SATA 1세대|150MB/s|-|-|
|SATA 2세대|300MB/s|-|-|
|SATA 3세대|600MB/s|-|-|
|PCIe 1세대|250MB/s|500MB/s|1000MB/s|
|PCIe 2세대|500MB/s|1000MB/s|2000MB/s|
|PCIe 3세대|1GB/s|2GB/s|4GB/s|
|PCIe 4세대|2GB/s|4GB/s|8GB/s|



> VirtIO-trace: 가상화 환경에서 NVMe SSD 입출력 특성 분석을 위한 통합 도구
>
> > 김세욱, 최종무 (단국대학교)
> >
> > 정보과학회논문지 제 45권 제 4호 (2018.4)

# VirtIO-trace: IO 분석 통합 도구

## 현재 상황

- 현재 리눅스 커널에서 사용가능한 I/O 추적 도구는 `blktrace` 하나뿐이나 마찬가지
- `VM`의 도입으로 인해서 최소 3개의 레이어에서 I/O를 처리 (아래 그림 참조)

![1523862378580](http://dl.dropbox.com/s/eodp2kne90ao5l1/1523862378580.png?dl=0)



## 현재 시스템의 문제점

![1523862405942](http://dl.dropbox.com/s/8jgsvo6x9kpsk0q/1523862405942.png?dl=0)

- VM 상황에서의 정확한 I/O 추적을 위해서는 VM의 커널 I/O, QEMU의 I/O 처리부, 호스트의 커널 I/O에 대한 로그 추적 필요 
- `blktrace`만 사용할 경우
  - VM 커널에서 `blktrace`를 수행
  - 호스트 커널에서 `blktrace`를 수행
  - blktrace를 두 레이어에서 사용하므로 **로그의 양 2배**
  - **수동**으로 두 시스템 간의 로그에 대한 동기화/분석
  - VM에서 사용한 `IO address`와 호스트의 `IO address`가 맞지 않음
  - QEMU의 I/O 처리부에 대한 정보는 QEMU 자체 로깅 시스템으로만 취합
- QEMU I/O 처리부에 대한 로그 시스템이 존재 하지 않음
  - QEMU의 I/O 처리부에서 VM이 요청한 IO에 대해 스케쥴링을 다시하고 `IO merge` 작업 수행
  - 실제 호스트 커널 까지 전달되는 I/O 가 달라짐 -> 동기화 및 분석을 더욱 어렵게 함
- NVMe를 저장장치로 사용하는 경우 PRP와 같은 NVMe 특화 정보에 대한 수집이 불가



## VirtIO-trace

![1523862751151](http://dl.dropbox.com/s/yufc17vomyw0wed/1523862751151.png?dl=0)

### 구성

- I/O 로그를 추적하는 디바이스 드라이버 모듈
- 디바이스 드라이버 모듈에 애플리케이션 I/O 정보를 입력하도록 도와주는 인터페이스 라이브러리
- VirtIO-trace의 옵션을 설정하고, 수집한 정보를 가공하는 유저 레벨 애플리케이션
  - VirtIO controller, VirtIO Analyzer

### 기존과의 차별점

- 인터페이스 라이브러리를 통해 QEMU에서 호스트 커널의 VirtIO-trace 디바이스 드라이버로 I/O 처리부의 동작에 따른 로그 입력 가능
- scatter/gather 명령시에 사용되는 PRP 정보 획득 가능
- 원하는 경우 데이터 무결성 검사를 위한 유저 데이터 획득 가능
- 로그 분석 도움을 위한 `message` 입력 가능 
- 편의를 위한 분석 도구 제공
  - 기본적으로 IO size, IO latency, IO 처리 상황, Queue access rate 분석 제공



## 장점 및 아쉬운점

### 장점

- QEMU의 IO 처리 루틴과 Host 커널의 IO 간에 동기화를 기존보다 쉽게 할 수 있음
- PRP가 사용될 경우 해당 PRP 정보 획득 가능

### 아쉬운점

- QEMU의 IO 처리부와 동기화만 진행하므로, VM 커널과 QEMU와의 동기화가 제대로 되지 않음
- 당연하지만 호스트 커널 및 QEMU 커널 모두 수정이 불가피함
- 기존 시스템에서 획득 가능한 정보들을 단순 취합하는 수준에 불가함
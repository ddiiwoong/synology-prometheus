# 시놀로지 NAS 모니터링

이번 포스팅에서는 다들 하나씩 구비하고 있을 시놀로지 NAS를 자체적으로 모니터링하기 위한 방법중에 Prometheus를 활용해서 모니터링 하는 방법을 정리하고자 함이다.

## 구성방안

구성방법은 아래와 같다. 모든 구성요소는 시놀로지내 도커 컨테이너로 구성될 예정이다.


## Synology 모니터링

기본적으로 시놀로지 UI에서도 관련된 데이터들을 모두 확인할수 있고 디스크 오류나 전원 등 특별한 이벤트가 발생했을 경우는 별도로 이메일이나 커스텀 스크립트를 통해 알림을 받을 수 있지만 메트릭을 기반으로 하는 특정 상황에서 알림을 받거나 내가 원하는 모니터링 대시보드를 꾸미기 위한 용으로 시놀로지는 SNMP기반의 모니터링을 제공한다. 다른 NMP또는 모니터링 도구등을 사용하기 보다는 실제 SNMP exporter를 구성하고 프로메테우스와 연동을 통해 그라파나 대시보드를 통해 모니터링을 해보는데 목적이 있다.

## SNMP Exporter

[https://github.com/prometheus/snmp_exporter](https://github.com/prometheus/snmp_exporter)

SNMP Exporter는 프로메테우스가 수집할 수 있는 형식으로 SNMP 메트릭 데이터를 Expose 할때 사용하는 방법으로 전통적인 모니터링 도구와 마찬가지로 MIB를 사용하게 된다. 

레포지토리 컨셉에도 설명이 적혀있듯이 SNMP 데이터는 계층형 데이터 구조를 가지고 있고 프로메테우스는 다차원 매트릭스를 사용하고 있기 때문에 두 시스템은 잘 맞는다고 볼 수 있다.

SNMP 데이터 구조를 여기서 자세히 설명하지는 않겠지만 SNMP index와 label을 매핑하는 방식으로 데이터를 처리한다. 

다른 익스포터와 동일하게 daemon 형태로 실행이 되고 `http://localhost:9116/snmp?module=if_mib&target=1.2.3.4` 와 같이 module과 target을 설정하는 방식으로 데이터 수집을 할 수 있다.

기본 config 파일을 `snmp.yml`을 참조하게 되는데 수동으로 작성하는 것이 아니라 generator를 통해 생성하게 된다. 

[https://github.com/prometheus/snmp_exporter/tree/main/generator](https://github.com/prometheus/snmp_exporter/tree/main/generator)

해당 링크에서 확인할 수 있듯이 generator에서 벤더별 MIB 파일과 generator.yml를 참조해서 빌드, 실행하고 결과값으로 snmp.yml이 생성되게 된다. 

다행이도 시놀로지에서는 다음과 같이 SNMP MIB 정보가 제공이 잘 되고 있다.
- [http://www.synology.com/support/snmp_mib.php](http://www.synology.com/support/snmp_mib.php)  

아래 MIB 파일을 다운로드 받는다.
- [https://global.download.synology.com/download/Document/Software/DeveloperGuide/Firmware/DSM/All/enu/Synology_MIB_File.zip](https://global.download.synology.com/download/Document/Software/DeveloperGuide/Firmware/DSM/All/enu/Synology_MIB_File.zip)  

그리고 snmp exporter github를 복제하시고 

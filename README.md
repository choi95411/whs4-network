# Network Packet Sniffer Assignment

## 개요
C 언어와 `libpcap` 라이브러리를 사용하여 구현한 간단한 패킷 스니퍼 프로그램입니다.

프로그램은 지정한 네트워크 인터페이스에서 TCP 패킷을 캡처하고, 캡처한 패킷의 Ethernet Header, IP Header, TCP Header, HTTP Message를 파싱하여 출력합니다.

출력하는 정보는 다음과 같습니다.

* Ethernet Header

  * 출발지 MAC 주소
  * 목적지 MAC 주소
* IP Header

  * 출발지 IP 주소
  * 목적지 IP 주소
* TCP Header

  * 출발지 포트 번호
  * 목적지 포트 번호
* HTTP Message

  * HTTP 요청 또는 응답 메시지

## 파일 구성

```text
assignment.c   # 패킷 캡처 및 분석을 수행하는 메인 소스 코드
myheader.h     # Ethernet, IP, TCP 헤더 구조체 정의
README.md      # 프로젝트 설명 및 실행 방법
```

## 필요 환경

이 프로그램은 `libpcap` 라이브러리를 사용합니다.

Ubuntu 또는 Kali Linux 환경에서는 다음 명령어로 설치할 수 있습니다.

```bash
sudo apt update
sudo apt install libpcap-dev
```

## 컴파일 방법

다음 명령어로 컴파일합니다.

```bash
gcc -Wall -Wextra assignment.c -o assignment -lpcap
```

`-lpcap` 옵션은 `pcap_open_live()`, `pcap_compile()`, `pcap_setfilter()`, `pcap_loop()` 등의 libpcap 함수를 사용하기 위해 필요합니다.

## 실행 방법

패킷 캡처에는 권한이 필요하므로 `sudo`를 사용하여 실행합니다.

```bash
sudo ./assignment
```

현재 코드는 eth0  인터페이스에서 패킷을 캡처하도록 설정되어 있습니다.

```c
pcap_open_live("eth0", BUFSIZ, 1, 1000, errbuf);
```

또한 현재 BPF 필터는 80번 포트의 TCP 패킷을 캡처하도록 설정되어 있습니다.

```c
char filter_exp[] = "tcp port 80";
```

## 테스트 방법

WSL 환경에서 HTTP 통신을 발생시키기 위해 터미널 3개를 사용합니다.

### 터미널 1: 패킷 스니퍼 실행

```bash
sudo ./assignment
```

### 터미널 2: 로컬 HTTP 서버 실행

```bash
python3 -m http.server 80 --bind 0.0.0.0
```

### 터미널 3: HTTP 요청 전송

```bash
curl http://192.168.64.1:80/
```

## 실행 결과 예시

```text
Src Mac: 00:15:5d:66:23:71
Dst Mac: 00:15:5d:a1:fd:1d
Src ip: 192.168.78.125
Dst ip: 192.168.64.1
Src port: 39310
Dst port: 80
HTTP Messages:
GET / HTTP/1.1
Host: 127.0.0.1:80
User-Agent: curl/8.5.0
Accept: */*
```

서버 응답 패킷이 캡처되면 다음과 같이 HTTP 응답 메시지도 확인할 수 있습니다.

```text
Src Mac: 00:15:5d:66:23:71
Dst Mac: 00:15:5d:a1:fd:1d
Src ip: 192.168.78.125
Dst ip: 192.168.64.1
Src port: 80
Dst port: 39310
HTTP Messages:
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.12.3
Content-type: text/html; charset=utf-8
```

## 로컬호스트 테스트 관련 설명

로컬호스트 테스트에서는 출발지 IP와 목적지 IP가 모두 `127.0.0.1`로 출력됩니다.

또한 MAC 주소는 다음과 같이 출력될 수 있습니다.

```text
Src Mac: 00:00:00:00:00:00
Dst Mac: 00:00:00:00:00:00
```

127.0.0.1` 통신은 실제 Ethernet 네트워크 카드를 거치지 않고 loopback 인터페이스 내부에서 처리되기 때문에 실제 MAC 주소가 사용되지 않습니다.


따라서 loopback 환경에서는 MAC 주소가 `00:00:00:00:00:00`으로 출력될 수 있지만, IP 헤더, TCP 포트 정보, HTTP 요청/응답 메시지는 정상적으로 파싱됩니다.

## 동작 설명

프로그램의 동작 흐름은 다음과 같습니다.

1. `pcap_open_live()`를 사용하여 네트워크 인터페이스를 엽니다.
2. `pcap_compile()`을 사용하여 BPF 필터를 컴파일합니다.
3. `pcap_setfilter()`를 사용하여 필터를 적용합니다.
4. `pcap_loop()`를 통해 패킷을 반복적으로 캡처합니다.
5. 캡처된 패킷에서 Ethernet Header를 파싱합니다.
6. IPv4 패킷인 경우 IP Header를 파싱합니다.
7. TCP 패킷인 경우 TCP Header를 파싱합니다.
8. TCP payload가 존재하면 HTTP Message로 출력합니다.

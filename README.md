# NTP 란?

- Network Time Protocol의 줄임말로 IETF(Internet Engineering Task Force)에서 제정하는 시간 동기화 프로토콜이다
- NTP를 통한 시간 동기화의 정확도는 Local-Clock Hardware의 정확도와 Device의 process 지연시간에 영향을 받음
- stratums 높아질수록 정확도 낮아지며 통상 stratum 1은 atomic clock (원자 시계)를 뜻함. 원자시계를 Source로부터 순차적으로 NTP 동기화가 이루어지는 경우, Stratum은 1씩 증가함.
  (여기서 말하는 시간은 GPS시간이며 통상적으로 사용하는 UTC시간에 비해 18초 느림. 
  매년 12월 31일 도는 1월 30일에 Bureau International des Poids et Mesures (BIPM) 라는 국제 도량형국에서  Leap second를 적용하여 GPS와 UTC의 시간오차를 결정함. 이 작업을 수행하는 경우, International Earth Rotation and Reference Systems Service 라는 자전 관련 국제기구에 6개월 전 고지르 함.
 
   
  GPS시간은 지구 자전의 미세한 변화에 따른 시간 오차를 적용하지 않음. GPS시간은 1980년 1월 1일부터 시작해서 원자시계로만 동작하므로 보정되는 UTC와 시간차 존재함 (위성의 원자시계에 leap second적용 안해주고 navigational data다운로드 받은 후에 leap secon적용해줌)

# 1.RedHat Enterprise Linux 7.6 NTP Setting
## 1.1 RHEL 7.6 에서의 NTP 서비스는?

### 1.1.1 

## 1.2 RHEL 7.6의 NTP설정 방법

chronyd 해제


# 2. Windows 10 Pro NTP Setting

## 2.1 Windows 10 Pro에서의 NTP 서비스는?
- Windows 10 Pro에서의 NTP는 'RFC 1305 (Network Time Protocol (Version 3)'를 기본으로 하고 있다. 
- Windows 개발자들은 W32Time서비스를 개발하여 Windows 2000과 네트워크 시간 동기화를 요구하는 Kerberos V5 authentication protocol의 규격을 지원하기 시작함.
- 단, W32Time은 full-featured NTP solution과 같은 섬세한 시간 어플리케이션은 아니며 Kerberos version 5 authentication protocol작동에 필요한 것이며 client computer와 server computer의 시간 동기화를 어느정도만 가능하게해줌. 즉, NTP 알고리즘을 활용하는 서비스이며 완벽하게 적용을 하지는 않는다.

(DELL사 워크스테이션의 Windows 10 Pro for Workstaiton은 sNTP를 사용하며?? )

(추가적인 authentication설정을 통해 nanosecond까지 동기화 할 수 있지만 해당 작업은 CPU-intensive하기에 수십ms의 정확도를 요구하는 시스템에서는 불필요함) 

### 2.1.1 W32Time 서비스 설명
Internet / GPS Device에서 Input Provider에게 Time sample 전송

Input Provider는 Windows Time Service Manager에게 Time sample 전송

Windows Time Service Manager는 Service Control Manager, Clock Discpline (System Clock), Output Provider와 시간정보를 주고 받음

## 2.2 Windows의 NTP 설정 방법
### 2.2.1

#### References
- https://www.rfc-editor.org/rfc/rfc1305.pdf
- https://docs.microsoft.com/ko-kr/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings

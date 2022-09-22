# * NTP 란?

- **Network Time Protocol**의 줄임말로 IETF(Internet Engineering Task Force)에서 제정하는 컴퓨터 시스템 간 시간 동기화를 위한 네트워크 프로토콜이다(응용계층 프로토콜).
- 안전성 stability (일정한 frequency유지) / 정확도 accuracy (표준 gps시간과의 차이의 정도) / 정밀도 precision (안전성, 정확도의 양이 일정하게 유지되는지)를 기준으로 NTP에 대한 평가가 가능하다

- stratums 높아질수록 정확도 낮아지며 통상 stratum 1은 atomic clock (원자 시계)를 뜻함. 원자시계를 Source로부터 순차적으로 NTP 동기화가 이루어지는 경우, Stratum은 1씩 증가함.

  (여기서 말하는 시간은 GPS시간이며 통상적으로 사용하는 UTC시간에 비해 18초 느림. 
  매년 12월 31일 또는 1월 30일에 Bureau International des Poids et Mesures (BIPM) 라는 국제 도량형국에서  Leap second를 적용하여 GPS와 UTC의 시간오차를 결정함.
  해당 작업 수행하는 경우, International Earth Rotation and Reference Systems Service 라는 자전 관련 국제기구에 6개월 전 사전 고지 진행.
 
## NTP 용어 설명
- **NTP Stratum model** : 0~15까지 있으며 device's distance to the reference clock을 뜻함 (a representation of the hierarchy of time servers in an NTP network) Stratum값이 올라갈수록 timing accuracy와 stability는 안좋아짐 (staratum0은 GPS antenna와의 소통이므로 cesium atomic clock의 정확성 필요함 : Primary Reference Clock)
- **Root dispersion** : An estimate of the total amount of error/variance between that server and the correct time. (서버에서 돌려주는 값임 : NTP Server가 external clock에서 시간 받은 후 알려주는데 externa과의 시간오차 + ntp client와의 시간오차가 합해진 것임)
  다른 요소들로인한 오차 보여줌 (ex. clock frequency의 부정확성) / root dispersio + (1/2) * root delay = root distance 
- **jitter** : 네트워크 통해 packet 전송 중 delay가 발생한 정도
- **Roundtrip delay**: : reference clock에 정확한 시간에 message가 도달하도록 기능 제공
- **Dispersion** : local clock의 최대 오류 표시함  -> 시간의 질 quality판단 가능 (e.g. peer와의 시간차이와 host와의 시간차이 비교 가능)
- **Offset** : NTP로 동기화된 클라이언트 시간과, 클라이언트 하드웨어 시간 차이를 나타냄->reference clock과 동기화 시키기 위해서 보정해야할 값
           서로 주고받을때 t2-t1과 t3-t4의 합을 반으로 나눈것임 (client의 시간이 느리게가면 +offset, server의 시간이 느리게가면 +offset)
  NTP Client는 받은 8개의 packet의 weighted 평균을 구해 dispersion계산하고 delay가 weighting factor임(smaller delay가 더 큰 weigh을 줌)-> quality of time server를 평가하기 좋음 (가는 시간/오는시간있을 때 offset은 오고 가는 시간 합을 2로 나눔 : 서버와 로컬 시간 차이 / ,delay는 오고 가는 시간의 차를 구함 : 네트워크 지연시간)
- **Root delay** :  Round-trip delay계산하는 것 : (network asymmetry인 경우 delay무한대 + 모든 client-server의 delay의 합임/ 여러 개 server가지고 있을때는 best root delay를 사용함)
- **GPS시간** : 은 지구 자전의 미세한 변화에 따른 시간 오차를 적용하지 않음. GPS시간은 1980년 1월 1일부터 시작해서 원자시계로만 동작하므로 보정되는 UTC와 시간차 존재함 (위성의 원자시계에 leap second적용 안해주고 navigational data다운로드 받은 후에 leap secon적용해줌)
**결론적으로 offset은 두 시계의 시간 차이 (1차 미분) / skew는 frequency 차이 (2차 미분)이며 Client나 Server의 Local 시간이 흐르는 속도의 변화(drift)가 발생하며 시간오착 발생한다*

### Linux NTP 용어 설명
- **Remote** : 상위 NTP서버의 호스트명 (*는 연동된 것, +는 진행가능한 상태)
- **refid** : 서버가 시간정보 참조하는 서버 (두 단계 상위의 서버 IP주소임)
- **Stratum** : 참조하는 것이 gov쪽이면 높은 stratum(적은 숫자)일 것임
- **t** : unicast, multicast,  broadcast
- **When** : client, serve가 동기화 후 얼마나 시간 지났는지 확인 (poll값 이하로 나타난다)
- **Poll** : 쿼리 날리는 빈도 (minpoll, maxpoll 2진수로 표시됨!!) -> minpoll은 3(8초), maxpoll은 17 (36시간)까지 가능함!, 정의하지 않으면 minpoll 6, maxpoll 10으로 정해짐!
- **Reach** : 0~377 중 하나의 값 최근 8번 동기화 진행 시 성공여부를 8진법으로 표시함 (1은 성공, 0은 실패 -> 1111 1111(2진수)-> 255(10진) -> 377(8진법) )
- **Delay** : 로컬 컴퓨터는 하드웨어 자체 시간, 운영체제 시간 별도로 운영됨 (운영체제가 하드웨어 시간 따라감)-> 운영체제 시간 확바꿔도 다시껐다 키면 하드웨어 시간을 따라가도록 함. -> delay는 실제 NTP서버 시간차이의 2배이니 반으로 나누면 ntp서버와 클라이언트 운영체제의 시간차 계산 가능

## NTP Implemenation Model

**Client-server model**
- hierarchy에 따라 작동함. client가 NTP message를 server로 보내고, server는 IP, port정보들 수정하고 checksum새로 계산하고 reply함, client는 reply를 process함 -> local time과 server time의 차이에 대해 필터링 작업 후 local time Update함 ( server여러개인 경우 best를 고름 : stratum과 synchronization time을 이용하는 Bellman-Ford distributed routing algorithm 활용) -> Update는 Section4의 알고리즘 이용 ->이로써 timekeeping accuracy 와 reliability를 유지한다

           host (local processor)와 peer (네트워크 연결된 remote processor)로 구분한다
          
          
## NTP 작동 방식
- **Message 흐름** : NTP Client에서 request 보냄 / NTP Server에서 client가 보내는 message에 주소, 포트 등의 정보 채워서 보냄
- **시간 교정 방법** : . reference timestamp . originate timestamp . receive timestamp . transmit timestamp  네개의 타임스탬프 사용함
- **NTP Message 분석**: NTP Query -> NTP Reply가 있고 MAC Header, IP Header, UDP Header, NTP Message가 있음
- NTP Message에 LI(Leap Warning Indicator 2비트 윤초 시간 알려줌), VN(3비트, version number), Mode(0 reserved, 1 sysmmetric active --- 7(private use)), Strat(8비트 stratum 0~15), Poll(log2), Prec (정확도, log2) 정보 있음
- **Data integrity**는 IP 와 UDP Checksum에 의해 증명됨
- **Data format** : 64bit로 이루어지며 2036년에는 초기화된다 (64bit크기가 다참)

   
## NTP 정확도에 영향을끼치는 요소들
**NTP를 통한 시간 동기화의 정확도는 Local-Clock Hardware의 정확도와 Device의 process 지연시간에 영향을 받음**
- 컴퓨터는 내장된 RTC (Real Time Clock)에서 시간을 계산한다. RTC는 외부에 크리스탈 오실레이터를 연결하여 사용하고 RTC 시계의 정밀도는 크리스탈의 정밀도에 의해 결정된다.
- 일반적인 크리스탈의 오차는 100 ppm이다. ppm 이란 parts per million의 약자로 1ppm은 100만분의 1의 오차를 가진다는 것을 의미하고 100 ppm은 1만분의 1의 오차를 가진다.
- RTC가 하루에 1초 이내의 오차를 가지기 위해서는 86,400분의 1 이내의 오차를 가져야 한다. 1만분의 1의 오차를 가진다면 하루에 8.64초의 오차가 생기게 된다.
- 일반 시계에도 유사한 크리스탈이 사용되지만 컴퓨터만큼 오차가 많이 발생하지는 않는다. 그 이유는 시계를 만들 때 각 크리스탈에 맞게 보정을 해주기 때문이다. 하지만, 컴퓨터를 만들 때는 이러한 보정을 하지 않기 때문에 큰 오차가 발생한다.
- 컴퓨터 RTC의 오차가 크기 때문에 타임 서버 (Time Server)를 통해 일정 주기마다 시계를 다시 맞춰주어야 한다. 타임 서버에는 매우 정밀한 RTC가 내장되어 있고 타임 서버에 접속하는 클라이언트에 정확한 시간 정보를 전달한다. 타임 서버에서 가장 많이 사용되는 프로토콜은 NTP (Network Time Protocol)이다.
- 컴퓨터 시계 : reference oscillator : quartz crystal (power grid)을 활용-> HW,SW서로 연동되서 Processor에게 시간 알려줌-> 항상 monotonically increasing한데, 이는 !!절대 뒤로가지 않는!! 시스템 타임을 유지하기 위함

# 1.RedHat Enterprise Linux 7.6 NTP Setting
## 1.1 RHEL 7.6 에서의 NTP 서비스는?

- NTP Server 역할을 하는 장비는 NTP Daemon이 작동되어야함 / NTP Client역할을 하는 장비는 ntpdate -u를 통해 주기걱 동기화 되거나 vi /etc/ntp.conf 에 들어가 바라보는 server IP 주소를 추가해야함

## 1.2 RHEL 7.6의 NTP설정 방법

chronyd 해제(ntpd와의 충돌로 ntpd종료 방)  

    systemctl disable chronyd


NTP Daemon동작 실행 (서버 시작시 Daemon작동을 위해 enable 명령어) 

    systemctl enable ntpd
    systemctl start ntpd

HW Clock을 Local clock에 동기화 시키기

    hwclock -w
        

동기화 여부 확인 방법 

    timedatectl
  
    ntpdate -d [Server IP] 
    
ntpq – standard NTP query program 했을 때 바라보는 서버 IP 출력되어야함
ntpstat – show network time synchronization status
(ntpd와 ntpdate는 같은 UDP port (#123)을 사용하므로 ntpd가 동작중이라면 ntpdate -u를 통해 Port #123를 제외한 다른 Port로 시간 동기화 해야함)

# 2. Windows 10 Pro NTP Setting
- Windows의 경우, Active Directory내에서 Domain 계정/인증/권한 설정이 가능하며 이를 통해 더 정교한 NTP 설정이 가능함. 현재, 외부 NTP Server와 NTP 동기화를 하기에 하단의 내용은 Domain관련 내용은 없음

## 2.1 Windows 10 Pro에서의 NTP 서비스는?
- Windows 10 Pro에서의 NTP는 'RFC 1305 (Network Time Protocol (Version 3)'를 기본으로 하고 있다. (현재 NTP Version 4까지 나옴)
- Windows 개발자들은 W32Time서비스를 개발하여 Windows 2000과 네트워크 시간 동기화를 요구하는 Kerberos V5 authentication protocol의 규격을 지원하기 시작함.
- 단, W32Time은 full-featured NTP solution과 같은 섬세한 시간 어플리케이션은 아니며 Kerberos version 5 authentication protocol작동에 필요한 것이며 client computer와 server computer의 시간 동기화를 어느정도만 가능하게해줌. 즉, NTP 알고리즘을 활용하는 서비스이며 완벽하게 적용을 하지는 않는다.

(사용중인 DELL사 워크스테이션의 Windows 10 Pro for Workstaiton은 RFC2030으로 SNMP version 4)
SNTP is a simplified access strategy for servers and clients using NTP. SNTP synchronizes a computer's system time with a server that has already been synchronized by a source such as a radio, satellite receiver or modem.
SNTP supports unicast, multicast and anycast operating modes. In unicast mode, the client sends a request to a dedicated server by referencing its unicast address. Once a reply is received from the server, the client determines the time, roundtrip delay and local clock offset in reference to the server. 

(추가적인 authentication설정을 통해 nanosecond까지 동기화 할 수 있지만 해당 작업은 CPU-intensive하기에 수십ms의 정확도를 요구하는 시스템에서는 불필요함) 

### 2.1.1 W32Time 서비스 설명
* Internet / GPS Device에서 Input Provider에게 Time sample 전송

+ Input Provider는 Windows Time Service Manager에게 Time sample 전송
  + W32Time Manger는 NTP Time Provider의 action을 시작시킨다 
    + Time sync step은 Input Provider가 NTP time source에게 요청하면 Input Provier는 time sample수신함 -> 이는 Windows Time Service Manager에게 전달되고 clock discipline subcomponent에게 전달 ->clock discipline subcomponent가 하단의 NTP 알고리즘 적용해 best time source / time sample정함 
      + 1) Clock–filtering algorithm : server에서 온 time samples 중 best time sample정함
      + 2. Clock-selection algorithm : most accurate time serve고름-> clock discipline algorithm 에게 전달되고 컴퓨터의 local clock고침 + 네트워크 지연, 시계부정확성에 대해 보완함
    +  clock discipline subcomponent는 system clock의 시간 조정함
      + client는 time skew의 차이가 크면 한번에 시간을 바꾸고
      + 차이가 적으면 local clock rate를 빠르게 또는 느리게 작동시켜 시간을 수렴한다

- Windows Time Service Manager는 Service Control Manager, Clock Discpline (System Clock), Output Provider와 시간정보를 주고 받음
  - Service Control Manager가 W32Time Service의 Start/ Stop을 담당함

## 2.2 Windows의 NTP 설정 방법
### 2.2.1

**Internet Time Settings**에서 time server IP 입력 후 Update now 클릭 
**Windows Group Policy Configuration** : gpedit.msc 통해 Local Group Policy Editor 들어간 후 Enable Global Configuration Settings / MaxPollInterval과 MinPollInterval 설정(이진법이므로 2를 설정할 경우 4초의 Interval가짐) / Enable Windows NTP client 후에 Configure Windows NTP client : NtpServer에 ‘NTP_SERVER_IP, 0x08’ 입력
**Windows Services Configuration** : services.msc 통해 Windows Service 들어간 후 Windows Time의 Startup type을 ‘Automatic (Delayed Start)’로 변경 (Default : ‘Manual’)
**Windows Task Scheduler** : Task Scheduler 들어간 후 Synchronize Time 클릭 후 ‘Disable’ 설정 / 

w32time 재시작 : CMD 창에서 하단의 명령어 입력

    net stop w32time
    net start w32time
    w32tm /resync

w32time통한 동기화 및 상단의 설정이 정확히 설정되었는지 확인 : CMD 창에서 하단의 명령어 입력

    w32tm /query /status
    

**시간 오차 확인 방법** : CMD 창에서 하단의 명령어 입력

    w32tm /stripchart /dataonly /computer:NTP_SERVER_IP

*1분마다 강제 resync 명령어를 w32time에 보내는 방법** : (단, 이 명렁어가 담긴 txt파일을 windows time scheduler 가 실행해야함) 

    @echo off
    :1
    set "params=%*"
    cd /d "%~dp0" && ( if exist "%temp%\getadmin.vbs" del "%temp%\getadmin.vbs" ) && fsutil dirty query %systemdrive% 1>nul 2>nul || (  echo Set UAC = CreateObject^("Shell.Application"^) : UAC.ShellExecute "cmd.exe", "/k cd ""%~sdp0"" && %~s0 %params%", "", "runas", 0 >> "%temp%\getadmin.vbs" && "%temp%\getadmin.vbs" && exit /B )

    w32tm /resync
    timeout -t 60 /nobreak
    goto 1

   


#### References
- https://www.rfc-editor.org/rfc/rfc1305.pdf
- https://docs.microsoft.com/ko-kr/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings

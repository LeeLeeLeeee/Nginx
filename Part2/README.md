## Nginx 기본 구성

### 구성 파일 구문

**구성 지시어**  

엔진엑스 구성 파일에서 동작을 정의하는 지시어
> 지시어는 항상 (;)로 끝나야한다.
> #은 주석을 의미한다.

**구조와 포함**

아래와 같이 읽어드릴 파일을 포함한다.
```bash
    include mime.type
```
`include`지시어는 `와일드카드`로 특정 패턴에 맞는 파일명을 지정하는 방식도 지원한다.

**지시어 블록**

지시어 블록은 구성을 논리적으로 구성할 수 있는 단위를 제공한다.
```bash
    events { # 이벤트 모듈에서 제공하는 이벤트 블록.
        # worker_connections는 해당 블록안에서만 영향을 미친다.
        worker_connections 1024;
    }
```
- 예시)
```bash
    http { # http 블럭
        server { # 서버 설정용 블럭
            listen 80; # 80 포트
            server_name example.com; # example.com과 정확히 일치하는 요청에 적용될 구성
            access_log /var/log/nginx/example.com.log; # 모든 HTTP 요청 텍스트에 기록
            location ^~ /admin/ {
                index index.php;
            }
        }
    }
```
**변수**

변수는 항상 `$`로 시작하며 기본으로 nginx에서 지원해주는 변수가있다.

### 기반 모듈 지시어

**기반 모듈 소개**

기본적인 기능을 가진 매개변수를 정의할 수 있는 지시어를 제공한다.  
기반 모듈은 다음과 같은 세 가지로 구분할 수 있다
- 핵심 모듈 : 프로세스 관리나 보안 같은 필수 기능 및 지시어로 이루어짐 
- 이벤트 모듈 : 네트워킹 기능의 내부 동작 방식 구성
- 구성 모듈 : 구성을 외부 파일에서 가져와 포함  


  
**핵심 모듈 지시어**
```bash
    # 데몬 모드
    daemon on | off ;
    # 디버그 지점 활성화
    debug_points stop | abort ;
    # 환경 변수 정의 및 덮어씌움
    env;
    # 오류 로그 쓰기
    error_log info <path> | notice | warn | error | crit | alert | emerg ;
    # 동적 모듈 활성화
    load_module <path>;
    # 프로세스 분할 여부 => on이면 다중
    master_process on | off;
    # pcre 정규식을 JIT(JUST-IN-TIME)에 컴파일 여부
    pcre_jit on | off;
    #nginx의 데몬 pid 경로
    pid <path>;
    # SSL 가속 명령 (: SSL 처리를 부하분산 장치가 대신하는 기능)
    ssl_engine <시스템에서 사용가능한 engine_name>;
    # 스레드 풀 참조 정의(: 스레드 풀은 말 대용량 처리를 위한 스레드 집합)
    thread_pool <name> threads=<number> [max_queue=<number>];
    # 유저 권한
    user <user_name> <group_name>;
    # 작업자 프로세스 우선 순위
    worker_priority 0;
    # 작업자 프로세스 수 지정
    worker_processes 4;
    # 진행 중인 작업자 프로세스가 끝날 때 대기 시간
    worker_shutdown_timeout 10s;
    # 작업자 프로세스가 코어 파일의 위치를 지정하는 용도
    # user 지시어로 지정되는 작업자는 이 디렉터리의 쓰기 권한이 반드시 있어야 함.
    working_directory <path>
```

- **이벤트 모듈 지시어**  
최상위 수준에 있는 `events`블록에 위치 시킬 것
```bash
    # 소켓 연결을 위한 상호 배제를 여부
    accept_mutex on | off;
    # 재 요청 시 대기 시간
    accept_mutex_delay <time>;
    # 일치 IP의 요청에 대해 상세 로그를 남김
    debug_connection <IP>;
    # 수신 큐를 일괄 받을 지 여부
    multi_accept off;
    # 작업자 프로세스가 동시에 처리할 수 있는 연결 수
    worker_connections 1024;
```

**성능 테스트**  
아래의 도구를 사용하여 테스트를 진행
- httperf
- Autobench
- OpenWebLoad
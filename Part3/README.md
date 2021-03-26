## HTTP 구성

### HTTP 핵심 모듈

**구조 블럭**

세 개의 논리 블럭 `http, server, location`을 제공한다.
- http: 블럭 구성 최상위, http 관련 모듈 전부 지시어는 이 안에만 정의 가능
- server : 이 블럭으로 웹 사이트 하나를 선언할 수 있음
- location : 특정 위치(Link)에만 적용되는 설정 정의할 때 사용

> !! 설정은 상속 및 오버라이딩이 가능하다.
```bash
    http {
        gzip on;
        server {
            server_name localhost;
            listen 70;
            # gzip은 상속되어 여전히 on
            location /downloads/ {
                gzip off; # 이 지시어는 해당 블럭에서 off를 시켜줌
            }
        }
    }
```

**소켓과 호스트 구성**  
간략하게 나열된 지시어로 호스트를 구성할 수 있다.

```bash
    # 단위 : server
    # 소켓을 여는 데 사용하는 주소와 포트 정의
    listen [주소][:포트] [옵션]
    --> 옵션 목록(default_server, ssl, http2, proxy_protocol)
    
    # 단위 : server
    # server블록에 하나 이상의 호스트 이름을 할당
    # nginx는 HTTP요청 받을 때 Host 헤더를 Server블록과 모두 비교하여 맞는 Server로 보냄
    # default 옵션이 활성화된 첫 블록에 우선권이 주어짐
    # 정규식 가능 *단 '~' 문자로 시작하여야 함 ~ ^(www)\.example
    server_name 호스트_이름1 [호스트_이름2...]

    # 단위 : http, server, location
    # 내부적인 경로 재설정의 경우에 적용된다.
    # on - server_name에 적용된 첫 호스트 이름 사용, off - 요청 호스트 헤더 사용
    server_name_in_redirect on | off

    # 단위 : http, server, location
    # 경로 재설정 상황에 포트 번호 삽입 여부
    port_in_redirect on | off
```
**경로와 문서**  
각 웹사이트가 제공할 문서를 구성하는 지시어를 설명한다.  
최상위 문서 위치, 사이트 색인, 오류 페이지 같은 것이다.
```bash
    # 단위 : http, server, location, if
    # 최상위 문서 위치 정의
    root <path>

    # 단위 location
    # alias는 특정 요청에서 별도 경로의 문서를 읽도록 할당
    alias <path>

    # 단위 : http, server, location, if
    # HTTP 응답 코드에 맞춰 URI 조작하거나 다른 코드로 대체
    error_page code1 [code2...] [=대체 코드] [@block | URI]
    # ex) 
    > error_page 404 /not_found.html
    > error_page 404 =200 /index.html # 404 => 200으로 교체 후 index로 반환

    # 단위 : http, server, location 
    # 'If-Modified-Since' HTTP 헤더를 처리하는 방법
    if_modified_since off | exact | before

    # 단위 : http, server, location
    # 요청에 아무런 파일명도 지정되지 않았을 때 기본 제공 페이지
    # 선언된 순서대로 확인함.
    index file1 [file2...]

    # 단위 : http, server, location
    # 오류 페이지 재귀 호출을 막는다.
    recursive_error_pages on | off

    # 단위 : server, location
    # 마지막 경로 인자(location 블럭, URI)를 제외한 지정된 모든 파일을 탐색하며 없을 땐 
    # 마지막 경로 인자로 이동한다
    location / {
        try_files $uri $uri2 $uri.html [@location_block_name | /index.html]
    }
    location @location_block_name {
        # 내용
    }
```

**클라이언트 요청**  
```bash
    # 단위 : http, server, location
    # 연결을 닫지 않고 유지하면서 제공할 최대 요청 횟수
    keepalive_requests <number>

    # 단위 : http, server, location
    # 서버가 유지되는 연결을 끊기 전에 몇 초를 기다릴지 정의
    keepalive_timeout <time>

    # 단위 : http, server, location
    # 연결 유지 기능 비활성화할 브라우저 선택 가능
    keepalive_disable browser1 browser2;

    # 단위 : http, server, location
    # 클라이언트가 데이터 전송을 중단한 순간부터 지정된 시간이 지난 후에 연결을 닫는다.
    send_timeout <time>

    # 단위 : http, server, location
    # HTTP 요청의 본문 데이트럴 실제 파일로 저장
    client_body_in_file_only off | clean | on

    # 단위 : http, server, location
    # 엔진엑스가 요청 본문 데이터를 메모리에 있는 단일 버퍼에 저장할 지 여부
    client_body_in_single_buffer on | off

    # 단위 : http, server, location
    # 클라이언트 요청 본문 데이터 보관할 버퍼의 크기
    client_body_buffer_size <size> # body
    client_header_buffer_size <size> # header

    # 단위 : http, server, location
    # 비활성 시한을 정의 기한이 차면 408 요청 시한 만료 HTTP 오류 반환
    client_body_timeout <time>
    client_header_timeout <time>    

    # 단위 : http, server, location
    # 본문 데이터 최대 크기. 초과할 경우 413 요청 내용 용량 초과 HTTP 오류 반환
    client_max_body_size <size>

    # 단위 : http, server
    # 잘못된 요청 헤더가 들어올 경우 "400 잘못된 요청" HTTP 오류 반환
    ignore_invalid_headers on | off
```
**MIME 타입**  
```bash
    # 단위 : http, server, location
    # MIME 타입과 파일 확장자의 상관관계를 맺는 블록
    types {
        MIME타입1 확장자1;
        MIME타입2 확장자2[확장자3...];
        [...]
    }

    # 특정 경로에서 MIME타입을 삭제하여 모두 다운로드 되게 하는 예시
    http {
        include mime.types;
        [...]
        location /downloads/ {
            types {} # 모든 타입 제거
            default_type application/octet-stream;
        }
        [...]
    }

    # 단위 : http, server, location
    # types블록과 일치하지 않을 때 사용
    default_type MIME타입
```

**제한과 제약**  
클라이언트가 서버의 특정 위치나 문서에 접근하려 할 때 적용할 제약 추가
```bash
    # 단위 : location
    # 명시적 허용한 것을 제외한 모든 HTTP 메서드를 막아준다.
    limit_except
    # 예시 
    location /admin/ {
        limit_except GET { # GET만 사용할 수 있다.
            allow 192.168.1.0/24; # 내부 IP 허용
            deny all
            [ allow | deny | auth_basic | auth_basic_user_file | proxy_pass | perl ]
        }
    }

    # 단위 : http, server, location, if
    # 개별 클라이언트 연결의 전송률을 제한할 수 있음
    limit_rate <size>

    # 단위 : location
    # 클라이언트가 모든 접근 조건 충족 옵션을 정의한다.
    satisfy any | all
    # 예시
    location /admin/ {
        allow 192.168.1.0.24; # 내부 IP 허용
        deny all; # 모두 거절 * 위의 내용은 변경할 수 없음
        auth_basic "인증이 필요합니다"; 
        auth_basic_user_file conf/htpasswd; # 유효한 사용자 정보를 입력한 클라이언트만 허용
    }
    # "statisfy all"일 경우 두 조건 모두 만족, "satisfy any"일 경우 하나만 만족해도 됨

    # 단위 : location
    # location블록을 내부용으로 지정한다. 즉 내부 경로 재설정으로만 접근할 수 있게 만든다.
    internal;
    #예시 
    server {
        [...]
        server_name .website.com;
        location /admin/ {
            # /admin/  페이지를 열 경우 404 오류가 발생
            internal; # 이제 /admin/은 내부 경로 재설정을 통해서만 접근할 수 있다
        }
    }
```

**파일 처리와 캐시**  
```bash
    # 심볼릭 링크 제어 기능
    disable_sysmlinks 

    # 지정된 값보다 큰 파일은 다이렉트 I/O로 읽혀진다.
    # 다이렉트 I/O는 중간 캐시 처리 없이 저장 장치에서 메모리로 직접 데이터를 읽게하는 것
    directio <size> | off

    # 열린 파일의 정보를 캐시에 보관하게 할 수 있다.
    # 파일 서술자, 존재 여부 등의 정보만 가지고 있다
    open_file_cache max-X [inactive=Y] 또는 off

    # 파일 오류를 캐시에 저장할 지
    open_file_cache_error
```

**기타 지시어**  
URI 조합, DNS, 로그저장 등
```bash
    # HTTP 404 오류를 로그에 기록 여부
    log_not_found on | off
    
    # 내부 경로 재설정 or SSI 요청에 의해 발생된 2차 요청 기록 여부
    log_subrequest on | off

    # 연속되는 슬래시를 하나로 합쳐준다. // => /
    merge_slashes

    # 브라우저 자체 오류 페이지를 방지
    # 512바이트가 될 때까지 다른 데이터로 채워지게 함
    msie_padding on | off 

    # DNS 서버를 지정한다
    resolver <DNS_SERVER_INFO>
    > resolver 127.0.0.1; # 자체 DNS
    > resolver 8.8.8.8 8.8.4.4 valid=1h # 구글 DNS사용 1시간동안 cached

    # DNS 타입 아웃
    resolver_timeout <time>

    # 사용자 정의 HTTP 헤더 이름에 (_) 허용 여부
    underscores_in_headers on | off

    # 요청 처리가 완료된 후 엔진엑스가 호출하는 URI 정의
    location /payment/ {
        post_action /scripts/done.php
    }
```

### 모듈 변수
HTTP 핵심 모듈은 많은 변수를 가지고 있으며 다양한 지시어의 값으로 사용될 수 있다.
소수의 지시어만 변수를 값에 정의할 수 있으니 가능 여부를 확인하면서 사용하자.

**요청 헤더**  
- $http_host : 호스트 이름
- $http_user_agent : 웹 브라우저를 나타내는 문자열
- $http_referer : 마지막 방문했던 이전 페이지 URL
- $http_via : 프록시에 대한 정보
- $http_x_forwarded_for : 프록시 거칠 경우 실제 클라이언트 IP 주소
- $http_cookie : Cookie HTTP 헤더 값. 클라이언트가 전송한 쿠키 데이터
- $http_ : 클라이언트가 보낸 헤더는 \$http_ 접두사 뒤에 특정 문자열로 만든 변수 이름으로 얻을 수 있다.
  - ABC-BD => $http_abc_bd, love => \$http_love

**응답 헤더**  
- $sent_http_content_type : Content-Type HTTP 헤더의 값, MIME 타입
- $sent_http_content_length : Content-Length HTTP 헤더의 값, 응답 본문의 길이
- $sent_http_location : Location HTTP 헤더의 값
- $sent_http_last_modified : Last-Modified HTTP 헤더 값, 수정 된 날짜 반환
- $sent_http_connection : Connection HTTP 헤더의 값, 연결이 유지될 것인지 닫힐 것인지
- $sent_http_keep_alive : Keep-Alive HTTP 헤더의 값, 연결이 유지되는 시간
- $sent_http_transfer_encoding : Transfer-Encoding HTTP 헤더 값
- $sent_http_cache_control : 클라이언트 브라우저가 자원을 캐시해야할 지 
- $sent_http_... : 요청 헤더와 비슷, 헤더 키에 따른 값 가져옴

**Nginx 생성**  
- $arg_XXX : `query string`에 접근한다. `XXX`는 원하는 매개변수 이름
- $args : `quert string` 전부
- $binary_remote_addr : 4바이트 이진 데이터 형태의 클라이언트 IP 주소
- $body_bytes_sent : 응답 본문으로 보내진 바이트 수
- $bytes_sent : 클라이언트에 보내진 바이트 수
- $connection : 연결을 식별하기 위한 일련번호
- $connection_request : 현재 사용되는 연결로 지금까지 처리된 요청 수
- $content_lengt : Content-Length HTTP 헤더와 동일
- $content_type : Content-Type 헤더와 동일
- $cookie_XXX : 쿠키 사용
- $document_root : 현 요청의 `root`나 `alias`지시어의 값이다.
- $document_uri : 현 요청의 URI다. 내부 경로 재설정 처리 결과에 따라 기존 URI와 다를 수 있다.
- $host : Host HTTP 헤더와 동일
- $hostname : 서버 컴퓨터의 시스템 호스트 명
- $https : HTTPS 연결일 때 on, 아니면 빈 값.
- $is_args : \$args 변수가 정의되었으면 `?`이고 없으면 빈 값이다.
- $limit_rate : 전송률 제한 값
- $msec : 현재 시간(초 + 밀리초)이다.
- $remote_addr : 클라이언트 IP
- $remote_prot : 클라이언트 소켓 포트
- $remote_user : 인증을 거쳤다면 사용자 명 반환
- $request_body : 클라이언트 요청 본문, 또는 본문이 비었다면 `-`
- $request_completion : 요청 완료되면 'OK' 아니면 빈 문자열
- $request_lenght : 클라이언트 요청 전체 길이
- $request_method : HTTP 메서드
- $request_time : 클라이언트에서 첫 바이트를 읽은 이후로 경과된 시간
- $request_url : 요청의 원래 URI
- $server_addr : 서버 IP 주소, 주의 해서 사용해라
- $server_name : 요청 처리하는 중 사용된 server_name 지시어 값
- $server_port : 서버 포트
- $status : 응답 상태 코드

### location 블록

**위치 조정 부호**  
패턴은 아래와 같다.
- 조정 부호 생략 [지정된 패턴 이외의 추가되는 문자는 무시]
- =  : [정확히 일치]
- ~  : [정규식에 일치]
- ~* : [정규식에 일치하며 대소문자 구별 X]
- ^~ : [지정된 패턴으로 시작]
- @ : [이름이 지정된 location 블록]
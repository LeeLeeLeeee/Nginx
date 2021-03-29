## 모듈 구성

### 재작성 모듈
URI 재작성하는 것이 목적인 모듈, 지원해주는 지시어는 많이 없지만 그에 비해 많은 기능을 제공한다.  
URI 재작서은 쿼리스트링으로 구별된 URI를 페이지의 속성이 들어나는 URI로 변경하여 `SEO`(검색 엔진 최적화)의  
핵심 요소이다.

**정규식**  
해당 재작성 모듈은 정규식을 아는 것이 매우 중요하다. 정규식 관련하여 간단히 정리해보겠다.

**PCRE 구문**  
*  `^` 시작 위치
*  `$` 끝 위치
*  `.` 모든 문자 매칭
*  `[]` 집합 내 문자 중 하나와 일치
*  `[^]` 부정 집합, 포함되지 않는 문자
*  `|` 문자 앞이나 뒤의 요소 중 하나에 일치
*  `()` 패턴이나 문자를 한 단위로 묶으며 종종 `|`와 같이 사용된다
    
**수량 표시**  
- `*` 없거나 1회 이상
- `+` 1회 이상
- `?` 없거나 1회
- `{x}` x회
- `{x,}` 최소 x회
- `{x, y}` x회 이상 y회 이하

**캡처**  
`()`로 묶은 문자를 캡처해서 보관할 수 있는 기능, 캡처되는 시점에서 임의의 변수로 지정할 수 있음
```bash
    # example - 1
    # 정규식 ^(hello | hi) (sir | mister)$
    # 문자 hello sir 
    # 캡처 $1 = hello, $2 = sir

    # example - 2
    # 정규식 ^(hello (sir))$
    # 문자 hello sir 
    # 캡처 $1 = hello sir, $2 = sir

    # example - 3
    # 정규식 ?/(?<folder>[^/]+)/(?<file>.*)$
    # 문자 /admin/doc
    # 캡처 $folder = admin, $file = doc
```

**내부 요청**  
`nginx`는 내부 요청 외부 요청을 구별하며 외부 요청은 클라이언트에서 직접 온 요청이며  
`location`블럭에 대응한다. 내부 요청은 `nginx`에서 특수한 지시어에 의해 발생한다.  
내부 요청은 다음 두 가지 유형으로 나뉜다.  
- 내부 경로 재설정 : `rewrite`같이 다른 `location` 블럭에 부합하도록 `URI`변경
- 부가 요청 : `add_after_body` `include` 원래 요청에 내용 추가

**error_page**  
특정 오류 코드가 발생했을 때 서버 행위를 지정함.  
```bash
    # 기본
    server {
        server_name webseite.com
        error_page 403 /errors.forbidden.html
        error_page 404 /errors.not_found.html
    }

    # 응용
    server {
        server_name website.com;
        root /var/www/vhosts/website.com/httpdocs/;
        error_page 404 /errors/404.html;
        location /errors/ {
            alias /var/www/common/errors/;
            internal;
        }
    }
```

**재작성**  
```bash
    server {
        server_name website.com
        root /var/httpdocs;
        location /storage/ {
            internal;
            alias /var/storage/;
        }
        location /documents/ {
            rewrite ^/document/(.*)$ / storage/$1;
        }
    }
```

**SSI**  
일반 HTML 파일 안에 엔진엑스가 실행할 명령에 태그를 삽입
```html
    <html>
        <head>
        <!--# include file="header.html" -->
        </head>
    </html>
```   

**조건부 구조**  
재작성 모듈에서 제공하는 새로운 지시어와 블록 중에는 if 조건부 구조도 있다.
```bash
    server {
        if($request_method == POST) {
            [...]
        }
    }
```
각 조건 연산자를 알아보자.
```bash
    if($string){
        # $string이 빈 문자열 or 0이 아닐 경우
    }

    if($string="A"){
        #string이 "A"
    }

    if($string!="A"){
        #string이 "A"가 아님
    }

    # 정규식 비교
    # ~
    # !~

    # 파일 유무
    if(-f $request_filename) {
        # 파일이 존재하면
    }

    # 디렉터리
    # -d

```

**지시어**  
```bash
    * rewrite <정규식> <대체> <플래그>
    # URI 재작성
    # 플래그 목록 : last, break, redirect, permanent

    * break
    # 더 이상의 rewrite 지시어를 방지하는 데 사용된다.
    # 이 지점 이후 URI가 고정돼 변경할 수 없다.
    if(-f $uri) {
        break;
    } 
    if($uri ~ ^/search/(.*)$) {
        set $query $1
        rewrite ^ /search.php?q=query?;
    }
    # 파일이 있다면 break를 걸어 재작성을 못하게 막는다.

    * return
    # 요청 처리하는 것을 중단하고 특정 코드나 지정된 문자 반환
    if(-f $uri) {
        return 403;
    }

    * set
    # 변수 초기화 및 재정의

```
**재작성 규칙**  
효과적인 SEO를 위한 각 페이지별 규칙들을 검색하여 찾아보자


### SSI 모듈

**변수 사용**  
SSI모듈로 변수를 사용할 수 있다.  
> \<!--# var="변수_이름" -->  

이 명령엔 세 가지 매개변수를 사용할 수 있다.
- var : 변수 이름 
- default : 변수 기본값
- encoding : encoding 방법

set을 사용하면 원하는 새 변수를 정의할 수도 있다.
> \<!--# set var="새_변수_이름" value="변수의 값" -->

**조건부 구조**  
```html
    <!--# if expr=""-->
    <!--# elif expor=""-->
    <!--# else-->
    <!--# endif-->
```

### 부가 모듈

**색인 모듈**  
`index`라는 지시어를 제공하며 기본으로 제공할 페이지를 정의
> index 파일 1 [파일2...]

**로그 모듈**  
엔진엑스가 접근 로그를 다루는 동작 방식을 제어
```bash
    access_log <경로> [format [buffer=크기]] | off
    # 접근 로그 파일 경로를 정함

    log_format <템플릿_이름> [escape=default | json | none] 형식
    # log_format simple '$remote_addr $request'

```

**제한과 제약**  
```bash
    auth_basic
    # 기본 인증 기능 제공, 특정 위치를 사용자 이름과 비밀번호로 인증한 사용자만 사용하게 제한

    auth_basic_user_file
    # 비밀번호가 저장된 파일 경로
```

**접근 모듈**  
`allow`와 `deny`라는 두가지 중요한 지시어가 제공됨.  
해당 지시어는 덮어써지는 형식이 아니기 때문에 순서가 중요함.
```bash
    location {
        allow 127.0.0.1; # 로컬 허용
        allow unix; # 유닉스 도메인 소켓 허용
        deny all; #나머지 전부 거절
    }
```

**접속 제한 모듈**  
특정 구역`(zone)`별 서버 최대 동시 접속 수를 정하게 해줌  
```bash
    # 첫 단계는 limit_conn_zone 지시어로 구역을 정의하는 것
    limit_conn_zone $변수 zone=이름:크기
    # 예시
    limit_conn_zone $binary_remote_addr zone=myzone:10m;
    
    # 구역을 정했으니 limit_conn으로 접속 수를 제한한다.
    limit_conn 구역_이름 접속_제한
    #예시
    limit_conn my_zone 1

```

**요청 제한 모듈**  
클라이언트 요청을 제한하는 모듈
```bash

    * limit_req_zone $변수 zone=최대_메모리_크기 rate=비율;
    #rate는 초당/분당 요청 횟수를 의미
    
    * limit_req zone=이름 burst=허용_초과_요청_횟수 [nodelay, delay=숫자] 

    # 예시
    limit_req_zone $binary_remote_addr zone=my_zone:10m rate=2r/s
    [...]
    location /downloads/ {
        limit_req zone=myzone burst=10;
        limit_req_status 404; # 한도 초과 404
    }

```

### 방문자 정보

**브라우저 모듈**  
User-Agent HTTP 헤더 분석해서 세가지 변수를 제공한다.  
- $modern_browser : 최신 브라우저 인지
- $ancient_broser : 구식 브라우저 인지
- $msie : IE브라우저 인지
>  modern_browser opera 10.0;
User-Agent HTTP 헤더에 Opera 10.0 문자가 포함돼 있으면 클라이언트는 최신으로 취급된다.

**맵 모듈**  
한 변수의 값에 따라 그에 대응하는 값을 새 변수에 할당한다.
```bash
    map $uri $variable {
        /page.html 0;
        /contact.html 1;
        /index.html 2;
        default 0;
    }
    rewrite ^ /index.php?page=$variable
```

**UserID 필터**  
이 모듈은 쿠키를 생성해서 클라이언트에 ID를 할당한다.  
ID는 $uid_got와 $uid_set 변수를 통해 얻을 수 있다.
```bash
    userid <on | off | v1 | log>
    # 쿠키를 발급하고 로그에 남기는 기능을 관리

    user_service <ip>
    # 쿠키 발급하는 서버 IP 주소를 정함

    user_name <이름>
    # 쿠키게 할당할 이름을 정함

    userid_domain <domain>
    # 쿠키를 할당할 도메인을 정함

    userid_path <경로>
    # 쿠키의 경로 부분을 정한다

    userid_expires <날짜 | max>
    # 쿠키 만료 날짜 설정
```

### SSL과 보안

**SSL 모듈**  
SSL모듈은 HTTPS를 지원한다.  
```bash
    ssl <on | off>
    # 구형 버전에서만 사용할 수 있다. 1.15.0 이전
    # 현재는 listen 인자로 넘긴다
    # listen 443 ssl

    ssl_certificate <경로>
    # PEM 인증서 경로 정함 여러번 사용할 수 있음

    ssl_certificate_key <경로>
    # PEM 비밀 키 파일 경로를 정함. 여러번 사용 가능

    ssl_client_certificate
    # 클라이언트 PEM 인증서의 경로를 정한다.

    ssl_crl
    # 엔진엑스에게 인증서의 폐기 상태를 확인할 수 있는 폐기 목록

    ssl_protocols
    # 사용할 프로토콜을 지정한다.

    ssl_ciphers
    # 사용할 암호화 방식을 지정한다.

    ssl_prefer_server_ciphers
    # 서버 암호화 방식을 클라이언트 방식보다 선호할 지 여부 지정함

    ssl_verify_client
    # 클라이언트에서 보내온 인증서 검증해서 결과를 $ssl_client_verify 변수에 저장함

    ssl_verify_dept
    # 인증 사슬의 검증 깊이를 지정한다.

    ssl_session_cache
    # SSL 세션의 캐시를 구성
```

**SSL 인증서 설정**  
SSL 모듈이 많은 기능을 제공하지만 주요한 몇 가지 지시어만 유용하게 쓰인다.  
SSL 설정을 단계별로 진행해보자

- `secure.website.com` 웹사이트라고 가정
```bash
    # 아래가 준비 되어있는 지 확인

    openssl genrsa -out secure.website.com.key 1024 
    # 위 명령으로 생성한 key 파일 
    openssl req -new -key secure.website.com.key -out secure.website.com.csr
    # 위 명령으로 생성한 csr 파일
    인증 기관에서 발급받은 경우 웹 사이트 인증서 파일
    인증 기관에서 발급받은 경우 인증기관 인증서 파일 # gd_bundle.crt


    # 첫 단계에는 아래 명령으로 웹사이트 인증서와 인증기관 인증서를 합친다.
    cat secure.website.com.crt gd_bundle.crt > combined.crt

    server {
        listen 443;
        server_name secure.website.com;
        ssl on;
        ssl_certificate /path/to/combined.crt
        ssl_certificate_key /path/to/secure.website.com.key;
        [...]
    }
```

## Nginx

### Nginx 파트

[1. Nginx 기본 구성](./Part2)  
[2. HTTP 구성](./Part3)  

### Nginx 개요

**Nginx란?**  
nginx는 경량 웹 서버이다.  
클라이언트 요청을 받았을 때 응답해주는 HTTP web Server로 활용되기도 하며  
부하를 줄이는 로드 밸런싱 서버로 활용되기도 한다.

**Nginx의 구조**  
Nginx는 하나의 `Master Process`와 다수의 `Worker Process`로 구성되어 실행됩니다.  
`Master Process`는 설정 파일을 읽고 유효성 검사 및 `Worker Process`를 관리한다.  
`Worker Process`는 모든 요청을 처리한다. `Worker Process`개수는 변경할 수 있으며  
정의된 프로세스 개수와 CPU코어 숫자에 맞게 자동 조정 된다.

**Nginx 설치**  
보통 Nginx설치는 `Yum`, `Yast`등 패키지 관리자를 통해 사용하나  
여기서는 직접 소스코드로 다운받아 설치하는 방식으로 진행  

_설치해야할 부가 모듈_  
- GCC : nginx는 C로 작성된 프로그램이기 때문에 컴파일러 도구가 설치되어야 함.
```bash
    gcc // => fatal error : no input files가 출력되면 설치되었다는 뜻
    apt-get install build-essential
```
- PCRE : nginx의 재작성 모듈과 HTTP 햇김 모듈은 해당 `정규식`구문을 사용한다.
```bash
    apt-get install libpcre3 libpcre3-dev
```
- zlib : zlib 라이브러리는 압축 알고리즘을 개발자에게 제공한다.
```bash
    apt-get install zlib1g zlib1g-dev
```
- OpenSSL : 강력한 범용 암호 라이브러리와 함께 보안 소켓 계층과 전송 계층 보안 프로토콜을 구현한다.
```bash
    apt-get install openssl libssl-dev
```
_Nginx 다운로드와 압축 해제_  
```bash
    wget http://nginx.org/download/[nginx_file_name]
    tar zxf [nginx_file_name]
    # 같은 경로의 configure 실행
    ./configure
    make
    make install
    # ./configure할 때 conf파일을 지정할 수 있음
    ./configure --conf-path=[static_file_path]
```

**Ngix 구동 옵션**  
_경로 구성 스위치_  
```bash
    # nginx 설치될 기준 폴더
    --prefix
    # nginx 실행 파일이 저장될 경로
    --sbin-path
    # 주 구성 파일의 경로
    --conf-path
    # 오류 로그 위치
    --error-log-path
    # 동적 모듈 설치될 위치
    --modules-path
    # 접속 로그의 위치 정의
    --http-log-path
    # 클라이언트 요청을 처리할 때 생성되는 임시 파일 위치
    --http-client-body-temp-path
    # 프록시가 사용하는 임시 파일 저장 위치
    --http-proxy-temp-path
    # 애플리케이션 빌드 위치
    --builddir
```
_모듈 옵션_
- `기본 활성 모듈`을 비활성화 하는 옵션
```bash
    # 웹페이지 재인코딩하는 문자 세트 모듈 비활성화
    --without-http_charset_module
    # Gzip 비활성화
    --without-http_gzip_module
    # SSI 모듈 비활성화
    --without-http_ssi_module
    # 쿠키로 사용자를 식별하는 사용자 모듈 비활성화
    --without-http_userid_module
    # 기본 인증 모듈 비활성화
    --without-http_auth_basic_module
    # 자동 색인 비활성화
    --without-http_autoindex_module
    # IP 주소 범위로 변수를 지정할 수 있는 지리 모듈 비활성화
    --without-http_geo_module
    # 키-값 형태 맵 모듈 비활성화
    --without-http_map_module
    # 참조 제어 모듈 비활성화
    --without-http_referer_module
    # 재작성 모듈 비활성화
    --without-http_rewrite_module
    # 멤캐시디 모듈 비활성화
    --without-http_memcached_module
    # 정의된 구역에 따라 자원 사용량을 제약하는 연결 제한 모듈 비활성화
    --without-http_limit_conn_module
    # 요청 복제해서 부가 요청 만드는 mirror 비활성화
    --without-http_mirror_module
    # 공백 Gif 이미지를 제공하는 빈 Gif 모듈 비활성화
    --without-http_empty_gif_module
    # 사용자 에이전트 문자열을 해석하는 브라우저 모듈 비활성화
    --without-http_browser_module
```
- `기본 비활성화 모듈`을 활성화 하는 옵션
```bash
    # SSL 모듈 활성
    --with-http_ssl_module
    # HTTP/2 지원 모듈 활성화
    --with-http_v2_module
    # 실제 IP 주소를 요청의 헤더 데이터에서 있는 모듈 활성화
    --with-http_realip_module
    # 응답 본문 앞이나 뒤에 데이터 덧붙일 수 있는 모듈 활성화
    --with-http_addition_module
    # 이미지 필터 모듈 활성화
    --with-http_image_filter_module
    # flv 모듈 활성화
    --with-http_flv_module
    # mp4 모듈 활성화
    --with-http_mp4_module
    # GZip 모듈 활성화
    --with-http_gzip_static_module
    # 디렉터리 색인 파일로 선정하는 무작위 색인 모듈 활성화
    --with-http_random_index_module
    # URL에 키워드의 유무 확인하는 보안 링크 모듈 활성화
    --with-http_secure_link_module
    # 요청을 여러 부가 요청으로 분할하는 분할 모듈 활성화
    --with-http_slice_module
    # 서버 통계와 정보 페이지를 생성하는 현황 모듈 활성화
    --with-http_stub_status_module
    # 구글 성능 도구 모듈 활성화
    --with-google_perftools_module
    # SSI에서 펄을 호출할 수 있게 해준다
    --with-http_perl_module
    # 인증 요청 모듈 활성화.
    --with-http_auth_request_module
```
- 여러 가지 다양한 옵션
```bash
    # 사용자 지정
    --user
    # 그룹 지정
    --group
    # HTTP 서버 비활성화
    --without-http
    # HTTP 캐시 기능 비활성화
    --without-http-cache
    # 경로 추가하여 컴파일 과정에 외부 모듈 추가. 제한 없음
    --add-module
    # 별도의 엔진엑스 빌드 이름을 지정
    --build
    # 부가 디버그 정보 로그에 남김
    --with-debug
    # 스레드 풀을 사용하게 활성화
    --with-threads
```
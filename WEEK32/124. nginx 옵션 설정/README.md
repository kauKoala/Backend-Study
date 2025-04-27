세팅하는건 봐도봐도 까먹음

```kotlin
worker_process 1;

클라이언트 요청을 처리하는 프로세스 개수 (요청 수락 및 응답 반환)
```

```kotlin
events {
	worker_connections 1024;
}

워커 프로세스가 동시에 처리할 수 있는 최대 연결 수
따라서 worker_process * worker_connections로 이론적 최대 동시 요청 수를 제어 가능함
```

```kotlin
// HTTP 서버 설정을 다루는 곳

http {
	
	// 설정 파일을 불러와서 브라우저가 어떻게 처리할지 알려줌
	// (Content-Type 헤더에 나오는 값들 - text/html, text/css, image/jpeg, ...)
	include        mime.types;
	
	// 파일을 보낼 때, 커널 레벨 고속 전송 기능을 사용하는지에 대한 여부
	// 동적 콘텐츠나, 서버에서 파일 내용을 실시간으로 수정/가공해야하면 캐싱 떄문에 off로 해야함
	sendfile        off;

	// nginx가 디렉토리 안에 index 파일이 없을 때, 디렉토리 안에 있는 파일들을 자동으로 보여줄지 여부
	// 그래서 /files/ 경로에 index.html이 없으면 /files/안에 있는 파일들이 모두 뜸
	// (보안을 위해 off)
  autoindex        off;

	// 에러 페이지와 서버 응답 헤더 필드에 nginx 버전을 명시할지에 대한 여부
	// (보안을 위해 off)
	server_tokens        off;
	
	// gzip으로 압축할지 여부
	gzip        on;
	// 압축 레벨 5 (레벨이 높아질수록 CPU 사용량 증가)
	gzip_comp_level        5;
	// gzip을 적용할 최소 응답 크기
	gzip_min_length        1000;
	// 프록시 환경에서는 gzip을 어느 값이 있는 응답도 압축할지
	gzip_proxied        expired no-cache no-store ...;
	// 어떤 MIME 타입에 gzip을 적용할지
	gzip_types        text/plain application-x-javascript text/xml ...;
	
	
	// 백엔드 서버 그룹 설정 (nginx가 요청을 받아 어디로 보내야할지 정하는 옵션)
	// 여기서는 backend라는 그룹을 설정함
	upstream backend {
	    server 127.0.0.1:8080;
	    keepalive 100;
	}
	
	
	// 가상 서버 설정
	// 이 주소로 특정 포트로 들어온 요청은 다음과 같이 처리한다는 세팅
	// 여기서부터는 값이 명시되어 있는건 기본값으로 전부 기본값임
	server {
	
	    // 특정 포트에서 요청을 받을지
	    listen 80;
	    
	    // 클라이언트가 요청할 본문을 1MB로 제한
	    client_max_body_size         1m;
	    // 버퍼 크기를 16k로 제한 (16k를 넘으면 디스크에 저장, 기본값 32/64bit 시스템 8k/16k)
	    client_body_buffer        16k;
	    // 요청 헤더 버퍼 크기를 1k로 제한
	    client_header_buffer_size        1k;
	    // 클라이언트가 헤더를 보내는 시간을 60초로 제한 
	    client_header_timeout        60s;
	    // 클라이언트가 본문을 보내는 시간을 60초로 제한
	    client_body_timeout        60s;
	    
	    // 연결이 유지되는 시간을 75초로 제한
	    keepalive_timeout        75s;
	    // DNS 타임아웃을 30초로 제한
	    resolver_timeout        30s;
	    // 서버가 클라이언트에게 보내는 최대 전송 시간을 60초로 제한
	    send_timeout        60s;
	    
	    // 헤더 이름에 _를 허용함 (nginx는 기본적으로 -만 사용 가능)
	    underscores_in_headers        off;
	
	
			// nginx 상태를 볼 수 있는 URL /basic_status에 대한 세팅
			// location = /basic_status는 정확히 특정 URL에 대한 경로만 처리
			location = /basic_status {
			    // 127.0.0.1만 접속 가능하고, 나머지는 차단
			    allow        127.0.0.1;
			    deny        all;
			    
			    // nginx 활동 중요 메트릭을 노출함
			    stub_status;
			}
			
			// URL / 에 대한 요청에 대한 세팅
			// location /는 접두어가 /로 시작하는 모든 요청에 대한 세팅
			location / {
			    // 프록시 서버 주소 (위에서 설정한 backend 그룹으로 프록시함)
			    proxy_pass        http://backend;
			    
			    // 프록시 서버와의 연결 시간을 60초로 제한
			    proxy_connect_timeout        60s;
			    
			    // 프록시 서버로부터 요청을 보내는 최대 시간을 60초로 제한
			    proxy_send_timeout        60s;
			    
			    // 프록시 서버로부터 응답을 읽는 최대 시간을 60초로 제한
			    proxy_read_timeout        60s;
			    
			    // 프록시 http 버전 설정
			    proxy_http_version        1.1;
			    
			    // 요청 본문 버퍼 크기 초과시 사용하는 임시파일 크기 설정 (0이면 메모리만 사용해서 처리함)
			    proxy_max_temp_file_size        1024m;
			    
			    // 리다이렉션 응답을 받을 때 그대로 사용할지에 대한 설정
			    proxy_redirect        on;
			    
			    // nginx는 기본적으로 서버 응답을 버퍼링해서 전체 응답을 수신할 떄까지 클라이언트에 전송하지 않음
			    // (off면 빠른 응답 가능)
			    proxy_buffering        on;
			    
			    // on이면 클라이언트가 요청을 중지해도 프록시 > 서버로 요청이 이동함
			    // off면 클라이언트가 요청을 중지하면 프록시 > 서버로 이동하지 않음
			    proxy_ignore_client_abort        off;
			    
			
			    // 요청을 서버로 전달할 때 Host 헤더 필드 값을 http_host로 생성
			    // $http_host는 host값 그대로 사용한다는 뜻
			    proxy_set_header        Host $http_host;
			    
			    // 요청을 서버로 전달할 때, Connection 헤더 값을 전달하지 않게 생성
			    // ""는 서버가 keepalive 같은 연결을 지속하지 않도록 사용함
			    proxy_set_header        Connection "";
			    
			    // 요청을 서버로 전달할 때, X-Forwarded-Host값을 host로 생성
			    // $host는 요청된 호스트명을 의미함
			    // X-Forwarded-Host 원본 요청의 호스트명을 의미
			    proxy_set_header        X-Forwarded-Host $host;
			    
			    // 요청을 서버로 전달할 때, X-Real-IP값을 remote_addr로 생성
			    // $remote_addr은 실제 요청한 IP 주소를 의미함
			    // X-Real-IP는 클라이언트의 실제 IP 주소를 뜻함
			    proxy_set_header        X-Real-IP $remote_addr;
			    
			    // 요청을 서버로 전달할 때, X-Forwarded-For값을 proxy_add_x_forwared_for로 생성
			    // 기존 X-Forwarded-For값에 $remote_addr을 추가함
			    // X-Forwared-For는 여러 IP 주소로 프록시 IP부터 클라이언트 IP까지 모두 기록함
			    proxy_set_header        X-Forwarded-For $proxy_add_x_forwared_for;
			    
			    // 요청을 서버로 전달할 때, X-Forwarded-Proto값을 scheme로 생성
			    // $scheme는 http인지 https인지 구분함
			    // X-Forward-Proto는 원본 요청이 http/https로 왔는지 전달
			    proxy_set_header        X-Forward-Proto $scheme;
			}
	}
}
```

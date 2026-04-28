# HTTP
## Introduction to HTTP

```

HTTP (Hypertext Transfer Protocol)
│
├──  Bản chất
│   ├── Protocol của World Wide Web
│   ├── Stateless (không lưu trạng thái)
│   |── Application Layer (TCP/IP)
│   ├── Không được mã hóa -> bất kỳ thiết bị trung gian(routers or proxies) có thể đọc và sửa đổi lưu lượng
│   └── Nó sử dụng headers trong cả req và resp để truyền. tải meta data và các thông tin khác
│   
├──  Architecture
│   ├── Client–Server Model
│   │   ├── Client
│   │   │   ├── Browser (Chrome, Firefox)
│   │   │   └── Tools (curl, Burp Suite)
│   │   │
│   │   └── Server
│   │       ├── Web Server (Apache, Nginx)
│   │       └── Application (PHP, Node.js, Java)
│
├──  Flow Request/Response
│   ├── Client → HTTP Request
│   │   ├── Method (GET, POST, PUT, DELETE)
│   │   ├── Headers
│   │   └── Body
│   │
│   └── Server → HTTP Response
│       ├── Status Code (200, 404, 500)
│       ├── Headers
│       └── Body (HTML, JSON, etc.)
│
├──  Thành phần trung gian (Intermediaries)
│   ├── Reverse Proxy
│   │   └── (Che giấu backend, routing)
│   │
│   ├── Load Balancer
│   │   └── (Phân phối traffic)
│   │
│   └── WAF (Web Application Firewall)
│       └── (Filter, detect attack)
│
├──  Port & Transport
│   ├── Default: TCP Port 80
│   ├── Có thể dùng port khác (8080, 8000…)
│   └── Chạy trên TCP
│
├──  Mở rộng
│   ├── HTTPS (HTTP + TLS)
│   │   └── Port 443
│   └── Encapsulation
│       └── Có thể nằm trong protocol khác (VPN, tunneling)
│
└──  Góc nhìn Pentest
    ├── Attack Surface
    │   ├── Headers manipulation
    │   ├── Methods abuse
    │   └── Request smuggling
    │
    ├── Intermediary bypass
    │   ├── Bypass WAF
    │   └── Confuse reverse proxy
    │
    └── Port-based attack
        ├── Scan port 80 / 8080
        └── Hidden HTTP services
```

## HTTP Communications
- `HTTP communications are based upon HTTP request and HTTP response.`
	- <font color="#ffc000">Client sends HTTP request to the server asking for a certain resource, and the server responds with the HTTP response</font>
- EX: 

```bash
# HTTP Request

GET /index.html HTTP/1.1 
Host: www.redseclabs.com 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430. 212 Safari/537.36 
Referer: https://google.com

# HTTP Response
HTTP/1.1 200 OK 
Date: Sun, 26 Nov 2023 12:00:00 GMT 
Server: Apache/2.4.41 (Unix) 
Content-Length: 450 
Content-Type: text/html; charset=UTF-8 
Connection: close

<html><body><h1>Welcome to RedSecLabs!</h1></body></html>
```

---
## DATA ENCODING
- có 4 loại:
	-  URL encoding 
	-  HTML encoding 
	- Base 64 encoding 
	- Unicode encoding
### URL encoding

- Table: Common characters encoded and encoded versions

| Character        | Encoded Version |
| ---------------- | --------------- |
| Space            | %20             |
| Double Quote (“) | %22             |
| Less Than (<)    | %3C             |
| Greater Than (>) | %3E             |
| Pound (#)        | %23             |
| Ampersand (&)    | %26             |
| Slash (/)        | %2F             |
| Plus (+)         | %2B             |
| Equal (=)        | %3D             |
| percent(%)       | %25             |
- double encoding: `%253C` -> WAF(%3C) ->BE(<)
### HTML Encoding


| Characters | Named Entity | Decimal Encoding  | Hexadecimal Encoding          |
| ---------- | ------------ | ----------------- | ----------------------------- |
| <          | &lt;         | &#60;<br>&#000060 | &#X3C;<br>&#0x3C<br>&#0000x3C |
| >          | &gt;         | &#62;<br>&#000062 | &#X3e;<br>&#0x3e<br>&#0000x3e |
| '          | &apos;       | &#39;<br>&#000039 | &#X27;<br>&#0x27<br>&#000027  |
| "          | &quot;       | &#34;<br>&#000034 | &#X22;<br>&#0x22<br>&#000022  |
|            |              |                   |                               |

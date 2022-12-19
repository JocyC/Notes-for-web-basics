#### 01 HTTP (hypertext transfer protocol) :

HTTP message format:
TCP (transmission control protocol) : TCP header + data

1. initial request line / initial response line

- initial request line: method (e.g. GET) + uri + http version

```js
GET /home HTTP/1.1
```

- Method:

  - GET (retrieve data with a given url)
  - HEAD (transfer the status line and header section only with a given url)
  - POST (sends data to the server)
  - PUT (replace all the current representations of the target resource with uploaded content)
  - DELETE (remove all the current representations of the target resource, which is given by url)
  - CONNECT (establish a tunnel to the server, identified by a given url)

- initial response line (aka status line): http version + response status code + status description

```js
HTTP/1.1 200 OK
```

2. header (request/response)
   field name: field header

   - not case sensitive
   - no sp nor \_ allowed
   - field name directly followed by :

3. empty line
   to separate head and body (if an empty line is added in head, then all the rest will be regarded as body)

4. body

#### 02 GET and POST

1. Syntax difference
2. Cache memory: GET request is by default cached by browser, POST no
3. Encoding: GET is only URL-encoded receiving only ASCII code, POST does not have this restrictioin
4. Idenpotence: GET is idenpotent (same result from one or multiple operation that's the same), POST not
5. TCP: GET sends its request at once, POST does that in two steps - send header, get 100 continue from browser, then send body (except from Firefox)

#### 03 URI (Uniform Resource Identifier)

Function: Identify resources on the internet
Structure: scheme :// user:passwd@ host:port path ?query #fragment

- scheme: protocol name, e.g. https, file...
- user:passwd@: the logged-in user's information (not safe, don't use)
- host:port (http default port:80 https:443)
- ?query: key=value

URI only takes ASCII code, so there is also a encoding system, converting non-ASCII code and delimiters to % + hexadecimal bytes
e.g. space is turned into %20

#### 04 Response Status Code

1xx: Informational responses, in process, need follow-ups

- 101 Switching Protocols: This code is sent in response to an Upgrade request header from the client and indicates the protocol the server is switching to.

2xx: Successful responses

- 200 OK
- 204 No Content: There is no content to send for this request, but the headers may be useful. The user agent may update its cached headers for this resource with the new ones.
- 206 Partial Content: This response code is used when the Range header is sent from the client to request only part of a resource. Use case: Download in chunks, Checkpoint restart

3xx: Redirection messages, the location of resources is changed, need to request again

- 301 Moved Permanently: The URL of the requested resource has been changed permanently. The new URL is given in the response. Browser will make buffer optimization by default and direct to the new url next time
- 302 Found: the url is just temporarily unaccessible
- 304 not modified: This is used for caching purposes. It tells the client that the response has not been modified, so the client can continue to use the same cached version of the response.

4xx: Client error responses, the request message has error

5xx: Server error responses

#### 05 HTTP characteristics

- Flexible in syntax and content
- Reliable (based on TCP/IP)
- Request-Response (client-server, server-server...)
- No context. Every http request is independent and irrelevant

Con:

- No context. May cause too much repetation in persistent connection
- Plaintext transmission. The messages take the form of text instead of binary code, easy to be attacked
- Head-of-line blocking. Process one request at a time, thus the others are blocked.

#### 06 Accept in Header

- Content-Type (request) Accept-Type (response)

  - MIME (Multipurpose Internet Mail Extensions) standard let HTTP marks the data type of body in Content-Type
  - text: text/html,text/plain,text/css
  - image: image/gif, image/jpeg, image/png
  - audio/video: audio/mpeg, video/mp4
  - application: application/json, application/javascript, application/pdf, application/octet-stream

- Content-Encoding (request) Accept-Encoding (response)

  - gzip(most popular), deflate, br(only for http)

- Content-Language / Accept-Language

  - zh-CN, zh, en

- Accept-Charset
  - There's no content-charset, put in Content-Type (Content-Type: text/html; charset=utf-8)

#### 07 HTTP transfer Fixed/Various-length data

For fixed-length, use Content-Length to specify.
(real length > Content-Length ? cut the data )
(real length < Content-Length ? transmission fails)

For various-length, use Transfer-Encoding: chunked

- If there is Transfer-Encoding: chunked in header, then Content-Length would be ignored, and send dynamic data based on keep-alive connection
- The structure of response body:
  chunk length (hexademical)
  first chunk
  chunk length
  second chunk
  ...
  0
  (there is a blank line at last)

#### 08 HTTP transfer large file: range request

Server adds "Accept-Ranges: none" in header to inform client that range request is supported

Client specifies the range with format:
bytes=x-y (e.g. 0-499, 500-, -100)

Server checks if the range is invalid (return 416 if yes), or returns the corresponding part with Contetn-Range in header and 206

- For Single range request:

  HTTP/1.1 206 Partial Content
  Content-Length: 10
  Accept-Ranges: bytes
  Content-Range: bytes 0-9/100

- For multiple range request, there is boundary between ranges:

  HTTP/1.1 206 Partial Content
  Content-Type: multipart/byteranges; boundary=00000010101
  Content-Length: 189
  Connection: keep-alive
  Accept-Ranges: bytes

  --00000010101
  Content-Type: text/plain
  Content-Range: bytes 0-9/96

  i am xxxxx
  --00000010101
  Content-Type: text/plain
  Content-Range: bytes 20-29/96

  eex jspy e
  --00000010101--

#### 09 HTTP form data

Two ways to submit form data with POST:

1. Content-Type: application/x-www-form-urlencoded
   URL-encoded to key value pairs separated with &
   e.g. {a:1,b:2}->"a%3D1%26b%3D2"
2. Content-Type: multipart/form-data
   - with boundary, which is assigned by browser by default
   - Every form element is independent
   - used for files like pictures

#### 10 HTTP 1.1 How to deal with HOL-blocking

Head-of-line (HoL) blocking occurs if there is a single queue of data packets waiting to be transmitted, and the packet at the head of the queue (line) cannot move forward due to congestion, even if other packets behind this one could.

Solutions:

1. Allow concurrent connections. RFC2616 in protocol allows for 2 at most, but now the upper limit rises much higher (chrome: 6)

2. Distribute secondary domain name to multiply the highest concurrent connections in one domain

#### 11 Cookie

1. Introduction
   Since HTTP does not have context, when it's needed, cookie will be used.
   Cookie is stored in a little text file in browser, inside data is key-value pair.
   When server gets request header with the same cookie, it can decode the cookie file and get the status of the client.

   - Cookie: a=xxx;b=xxx
     In turn, server can also write in Cookie through Set-Cookie in response header.
   - Set-Cookie: a=xxx
     Set-Cookie: b=xxx

2. Properties

   - Validation period / lifetime of a cookie
     - Session cookies are deleted when the current session ends (defined by browser)
       It can be set with Expires (when it's expired) and Max-Age (unit: second, count from the reception of messages)
       Cookie will be deleted if it's expired
   - Scope
     Bound Domain and Path to the cookie. If they are not right, then the request will not send cookie with it.
     - In the path, / means all the path allow the usage of Cookie
   - Security related
     - If there is Secure, then cookie can only be transferred by HTTPS
     - If there is HttpOnly, then it's only tranferrable with http protocol, not by JS (prevent XSS, or Cross-site scripting attacks)
     - SameSite to prevent CSRF, Cross-site request forgery
       - SameSite=Strict: prohibit third party request outside the domain with Cookie
       - SameSite=Lax: only allows cookie when using GET to submit forms or using a tag to send GET request (Currently by default)
       - SameSite=None: request takes Cookie

3. Downsides
   - Small, only 4kb max
   - Request carries Cookie whether it's needed or not, a waste of performance (can be solved with Scope assignment)
   - Security threat: as plaintext, cookie is easy to get hijacked

#### 12 HTTP proxy server

- Functions

  1. As it follows client requests - proxy - servers model, the proxy server can distribute the request to different origin servers so they have more equal burden
  2. Ensures security: Detect dead server with Heartbeat, filter data, Rate Limiting to illegal IP address
  3. Proxy caching

- Header
  - Via: (ordered)
    - proxy_server1, proxy_server2 in request
    - proxy_server2, proxy_server1 in response header
  - X-Real-IP: the original client's IP (X-Forwarded-Host for original client's domain, -Proto for protocol)
  - X-Forwarded-For: the IP address of request sender
    - Problem: encoding and changing http request header is not efficient; in https, original messages can not be changed
    - Solution: add Proxy Protocol (plaintext) before initial request line
      PROXY + TCP4/TCP6 + requester address + receiver address + request port + receive port

#### 13 HTTP cache and proxy caching

1. Cache-Control:
   If-Modified-Since/If-None-Match checks if the resources has been updated

   - Updated: return with resource and 200 OK
   - Not: return with 304, tell the browser to get resources directly from cache

2. Proxy-caching:
   Obtaining the cache in origin server every time can pose too much pressure on it. Therefore, let proxy server take over a part of server-side http cache, then client can obtain cache from the closest proxy server until the cache in proxy expires.
   Proxy caching is a feature of proxy servers that stores content on the proxy server itself, allowing web services to share those resources to more users. The proxy server coordinates with the source server to cache documents such as files, images and web pages.

3. Cache control from origin server side

   - Cache-Control: private (proxy caching not allowed) / public (proxy caching allowed)
   - Cache-Control: must-revalidate (client cache expires -> obtain from origin server) / proxy-revalidate (proxy cache expires -> obtain from origin server)
   - Cache-Control: s-maxage=2000 (cache is validate for 2000s in proxy, s for share. maxage indicates the validation time in client)

4. Cache control from client side
   - max-stale:5 when cache in proxy expires for less than 5s, still get from proxy
   - min-fresh:5 get cache from proxy until 5s before expiration
   - only-if-cached only gets cache from proxy but not origin server. if no valid cache from proxy, returns 504 Gateway Timeout

#### 14 Cross-Origin

1. cross-origin
   In two URI, if scheme, host and port are not all the same, then they are not from the same origin.
   They can't:

- read and change each other's DOM
- read each other's Cookie, IndexDB and LocalStorage
- send XMLHttpRequest unlimitedly

2. cross-origin request blocking
   Cross-origin request is normally **blocked by browser**.

   The process of browser (Chrome): render process, plugin process, browser process, GPU process, sandbox.

   During render process (happens in separated sandboxes): xhr.send called, ajax ready to be sent

   Render process cannot send ajax -> Inter Process Communication (IPC) -> send to Browser process who sends the request out -> server send a response back -> browser process finds out it's cross-origin without CORS header -> don't send the response body back to render process -> blocked

I. CORS

1. simple request:
   - methods: GET, POST, HEAD
   - in header: Accept, Accept-Language, Content-Language, Content-Type(with only application/x-www-form-urlencoded, multipart/form-data, text/plain)
2. For simple request
   - browser adds by default Origin: to clarify the origin of the request
   - server responses with Access-Control-Allow-Origin: \*
   - if Origin is not in Access-Control-Allow-Origin, then browser will block the response
   - Access-Control-Allow-Credentials (boolean) is set to false by default, meaning it's not allowed to send Cookie. (If need cookie: add this in header and set to true & set xhr.withCredentials=true in Frontend)
   - Access-Control-Expose-Headers : enable XMLHttpRequest object to get the value of this header with getResponseHeader() in frontend
3. For preflighted request
   - the browser first sends an HTTP request using the **OPTIONS method** to the resource on the other origin, in order to determine if the actual request is safe to send
     - Access-Control-Request-Method: CORS use which http method (e.g. POST)
     - Access-Control-Request-Headers: the header (a nonstandard useful header is X-PINGOTHER)
   - the response for preflighted request
     - Access-Control-Allow-Origin
     - Access-Control-Allow-Methods
     - Access-Control-Allow-Credentials
     - Access-Control-Allow-Headers
     - Access-Control-Max-Age
   - the CORS response
     if preflight request does not match the requirement of response header, then error & not sending CORS response
     just like that of simple request

II. JSONP

JSONP stands for JSON with Padding.

Requesting a file from another domain can cause problems, due to cross-domain policy. Requesting an external script from another domain does not have this problem. JSONP uses this advantage, and request files using the script tag instead of the XMLHttpRequest object.

But only GET is supported

III. Nginx: reverse proxy

Reverse proxy gets the requests from client side and transfer them to other servers

Nginx gives a middle uri with the same domain as client, then gets the response from a server and gives it back to the client

#### 15 TLS1.2 handshake

HTTPS = HTTP + SSL/TLS

Secure Sockets Layer 3.1 = Transport Layer Security 1.0

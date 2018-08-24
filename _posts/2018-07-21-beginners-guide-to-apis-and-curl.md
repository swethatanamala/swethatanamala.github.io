---
layout: post
title: Beginner's guide to APIs and curl
---
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
If you're in tech world, you should have heard about web APIs by now. You might be a newbie like me and still figuring out what exactly APIs are. 

<p align="center">
<img src="/assets/Images/api-post/apis-everywhere.jpg" alt="Architecture">
</p>

I learnt about them recently myself and now I will guide you in this post. After a brief introduction to APIs and HTTP, we will get our hands dirty with [`curl`](https://curl.haxx.se) and make a few API requests ourselves.

## What are APIs
APIs (Application programming interfaces) are interfaces that facilitate the users to programmatically interact with the services. Most common APIs are the web APIs (also called as cloud API services). HTTP (Hypertext transfer protocol) is an application level protocol for data communication in world wide web. Here are some key points about HTTP:

- HTTP is a communication protocol between client and server
- It uses TCP protocol, a transport layer protocol for reliable data transfer between client and server
- It is a stateless protocol, the server and client are aware of each other only during a current request. Afterwards, both of them forget about each other. 

HTTP protocol supports various 'methods' to communicate between client and server. The client requests the server using these methods and server in turn  responds to the request. There are several HTTP methods of which the most relevant ones are `GET`, `POST` and `DELETE`.

Client can also come in multiple forms. Your web browser, for example, is an HTTP client. Another common client is the unix command: `curl`. We can also call APIs programmatically via libraries such as [requests](http://docs.python-requests.org/en/master/) in python and [jquery](https://jquery.com) in javascript. Here, I'll illustrate `GET`, `POST` and `DELETE` methods with `curl`. 

### GET request
GET request is the most used method in HTTP. This method is used to retrieve the requested data from the server. 


```sh
$ curl -v -X GET 'https://swethatanamala.github.io/about/' 
```

Using `curl` command line utility, we can set our request method using optional argument `-X` followed by method, here `GET` and `-v` tells more information of the client side request and server side response.

Output:
```text
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying 185.199.108.153...
* TCP_NODELAY set
* Connected to swethatanamala.github.io (185.199.108.153) port 443 (#0)
* ALPN, offering http/1.1
.
.
.
*  issuer: C=US; O=DigiCert Inc; OU=www.digicert.com; CN=DigiCert SHA2 High Assurance Server CA
*  SSL certificate verify ok.
> GET /about/ HTTP/1.1
> Host: swethatanamala.github.io
> User-Agent: curl/7.52.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Type: text/html; charset=utf-8
< Server: GitHub.com
< Last-Modified: Tue, 17 Jul 2018 03:22:10 GMT
< Content-Length: 4422
.
.
.
< Vary: Accept-Encoding
< X-Fastly-Request-ID: 455dbb90f3cbf1d958522ef20f38fcc8ba5c1138
< 
<!DOCTYPE html>
<html lang="en-us">

  <head>
  <link href="http://gmpg.org/xfn/11" rel="profile">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta http-equiv="content-type" content="text/html; charset=utf-8">
.
.
.
  </body>
</html>
```

From the first line of the output, we can see that default method (`-X`) is `GET` as it is commonly used method to retrieve the information in any form like html, text, image, json output etc.

HTTPS is a secure version of HTTP where certificate of the server is validated before initiating the http request. The information about this validation process is prefixed with `*`. 
<!-- This information contains about TLS(Transport Layer Security) connection protocol, start and expiry dates of certificate, issuer of the certificate and whether it got validated or not. -->

The lines prepended with `>` are client request method followed by headers. Headers are the key value pairs describes the request/user.

```text
> GET /about/ HTTP/1.1
> Host: swethatanamala.github.io
> User-Agent: curl/7.52.1
> Accept: */*
```

The first line of the request is `<method> <directory of content> <protocol version>`. Then headers follow: `Host` value should be the root of the server. As request is sent by the curl, so `User-Agent` is curl. `Accept` header tells which MIME content we request (here we specified it as any type).

The lines prefixed with `<` are the server response followed by headers. 

```text
< HTTP/1.1 200 OK
< Content-Type: text/html; charset=utf-8
< Content-Length: 4422
< 
# message starts here
```

First line of the response has status code. Status code 200 means our request got processed successfully. I write more about status codes at the [end of this post](#status-codes).  `Content-Type` tells the MIME type of message we got from the server while `Content-Length` is the length of message. There should be a blank line after the message header and message body. In our example, message was a HTML page.

We have seen so far how to retrieve a HTML page at a URL. Now let's see an example of a web API which uses JSON. We will use [reqres.in](https://reqres.in), a fake API service for this purpose. For example, following `GET` request fetches details about user with id 2.

<!-- Let's see an scenario where this json format will be useful, on clicking a product image on the shopping website, a `GET` request is passed to fetch the more details like seller, specifications and features of the product in the json format. Then this json format is parsed accordingly to show a dynamic html web page. -->

```bash
$ curl https://reqres.in/api/users/2  | json_pp # json pretty print
{
   "data" : {
      "avatar" : "https://s3.amazonaws.com/uifaces/...",
      "first_name" : "Janet",
      "id" : 2,
      "last_name" : "Weaver"
   }
}
```

Now let's see list all the users using `GET`

```bash
$ curl https://reqres.in/api/users/ | json_pp
{
   "total" : 12,
   "page" : 1,
   "per_page" : 3,
   "total_pages" : 4,
   "data" : [
      {
         "first_name" : "George",
         "id" : 1,
         "avatar" : "https://s3.amazonaws.com/uifaces/...",
         "last_name" : "Bluth"
      },
      {
         "last_name" : "Weaver",
         "avatar" : "https://s3.amazonaws.com/uifaces/...",
         "id" : 2,
         "first_name" : "Janet"
      },
      {
         "first_name" : "Emma",
         "id" : 3,
         "avatar" : "https://s3.amazonaws.com/uifaces/...",
         "last_name" : "Wong"
      }
   ]
}
```

`api/users/` actually listed the users in page 1 output by default. How do we then get the users in page 2? We query the server using parameters encoded in the URL as `<url>?<key1>=<value1>&<key2>=<value2>`. Example request is shown below.

```bash
$ curl https://reqres.in/api/users?page=2 | json_pp
{
   "total" : 12,
   "page" : 2,
   "per_page" : 3,
   "total_pages" : 4,
   "data" : [
      {
         "id" : 4,
         "first_name" : "Eve",
         "avatar" : "https://s3.amazonaws.com/uifaces/...",
         "last_name" : "Holt"
      },
      {
         "first_name" : "Charles",
         "id" : 5,
         "avatar" : "https://s3.amazonaws.com/uifaces/...",
         "last_name" : "Morris"
      },
      {
         "last_name" : "Ramos",
         "first_name" : "Tracey",
         "id" : 6,
         "avatar" : "https://s3.amazonaws.com/uifaces/...""
      }
   ]
}
```


### POST request
POST request is mostly used in uploading the data to the server in HTTP. The common use case of POST request is submitting forms on web.

```bash
$ curl -v -d '{"name":"swetha", "job":"SDE"}' \
    -H "Content-Type: application/json" -X POST https://reqres.in/api/users
```

As `POST` request requires data for uploading, so we specify it using `-d`. We should also inform to the server that the MIME type of the data using header argument `-H`.

Ouput:

```text
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying 104.27.159.254...
* TCP_NODELAY set
* Connected to reqres.in (104.27.159.254) port 443 (#0)
* ALPN, offering http/1.1
.
.
.
*  SSL certificate verify ok.
> POST /api/users HTTP/1.1
> Host: reqres.in
> User-Agent: curl/7.52.1
> Accept: */*
> Content-Type: application/json
> Content-Length: 30
> 
* upload completely sent off: 30 out of 30 bytes
< HTTP/1.1 201 Created
< Content-Type: application/json; charset=utf-8
< Content-Length: 79
< Connection: keep-alive
.
.
.
< CF-RAY: 43de393f78fb7074-SIN
< 
{"name":"swetha","job":"SDE","id":"642","createdAt":"2018-07-21T14:07:07.176Z"}
```

When  `-d` is specified, curl infers that it is a POST request and therefore there is no need to set `-X POST` explicitly. In addition to the headers as in `GET` request, `Content-Type` and `Content-Length` are required in the request header itself because we are uploading data in a `POST` request. Note that the status code is 201 means this user is created successfully.

### DELETE request
The `DELETE` request deletes a specified resource in the request. 

```bash
$ curl -v -X  DELETE https://reqres.in/api/users/2
```
In the above command, we set -X as `DELETE` method.

Output:

```text
*   Trying 104.27.159.254...
* TCP_NODELAY set
* Connected to reqres.in (104.27.159.254) port 443 (#0)
.
.
.
*  SSL certificate verify ok.
> DELETE /api/users/2 HTTP/1.1
> Host: reqres.in
> User-Agent: curl/7.52.1
> Accept: */*
> 
< HTTP/1.1 204 No Content
< Date: Sat, 21 Jul 2018 14:28:02 GMT
< Connection: keep-alive
.
.
.
< CF-RAY: 43de9422ff502ddf-BOM
< 
```

Status code 2xx implies this is a success.

### Status codes
Status codes are issued by a server in response to a client's request made to the server. They are grouped into 5 categories.

| Status code   |      About      |  Examples |
|----------|:-------------:|------:|
| 1xx |  Information about the request | 100 Continue |
| 2xx |  Successful request  |   200 Ok |
| 3xx | Needs further action to process request | 301 Moved permanently|
| 4xx | Illegal request from client  | 404 Not found|
| 5xx | Server incapable of serving request | 504 Gateway timeout|

404 Not found is probably the most well known http status.

<p align="center">
<img src="/assets/Images/api-post/404.png" alt="Architecture">
</p>

Here's how a failed request looks in `curl`.

```text
$ curl -v https://swethatanamala.github.io/random-incorrect-url/
> GET /random-incorrect-url/ HTTP/1.1
> Host: swethatanamala.github.io
> User-Agent: curl/7.52.1
> Accept: */*
> 
< HTTP/1.1 404 Not Found
< Server: GitHub.com
< Content-Type: text/html; charset=utf-8
.
.
.
< X-Fastly-Request-ID: 00bf425ea29a7a1322d3ec9b8b6eefd38ddc6005
< 
<!DOCTYPE html>
<html lang="en-us">
  <head>
.
.
.
  </body>
</html>
```

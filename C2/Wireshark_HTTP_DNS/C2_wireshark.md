## HTTP

### conditional GET/response

连续两次请求同一网站，相比第一次请求，第二次请求增加了“IF-MODIFIED-SINCE” head line，同时第二次的回复中，因为对象并没有修改，服务器没有将对象的内容发送给客户。（如第一次wireshark实验所见）

### HTML Documents with Embedding Objects

给定的网页中包含两张图片（图片本身不在HTML文件中，但图片的URL在文件中），因此会依次发送三次请求分别获取HTML文件和两张图片

![29d30b2266464d3626103334560deb7](C2_wireshark.assets/29d30b2266464d3626103334560deb7.png)

### HTTP  Authentication

对于需要授权验证身份（基础情况即为输入用户名和密码）的网页对象。

![aa219240ab252a52d556adc9019b7cd](C2_wireshark.assets/aa219240ab252a52d556adc9019b7cd.png)

Client发送第一条GET请求后，Server会返回401 Unauthorized status code 并带有 WWW-Authentication head field 

![997f50815a8c222cce08eaf569daf87](C2_wireshark.assets/997f50815a8c222cce08eaf569daf87.png)

用户输入用户名密码后，Client再次发送一条请求，包含 Authorization head field，内容为Basic 和 用户名冒号密码的编码串。

![5bf9cc446819ab9e001d3232b9b68e8](C2_wireshark.assets/5bf9cc446819ab9e001d3232b9b68e8.png)

如果验证通过，Server会发送网页文件，否则会发送401 Unauthorized response表明验证未通过。

上面用户名和密码的编码是 Base64 format，通过  [Base64-Online Base64 Decoder and Encoder](https://www.motobit.com/util/base64-decoder-encoder.asp) 即可实现编解码，因此通过这种简单的方式实现授权是很不安全的。



## DNS

### nslookup

通过nslookup程序可以指定对权威DNS服务器查询指定网站的IP地址



### ipconfig

用于显示当前的TCP/IP信息

```dos
ipconfig /flushdns  #to clear the cache in your host
```



### Tracing DNS with wireshark

##### 打开网站[IETF | Internet Engineering Task Force](https://www.ietf.org/)

![fe8d2153a439b0a2f13eab9eeb51c77](C2_wireshark.assets/fe8d2153a439b0a2f13eab9eeb51c77.png)

![cf1d14db6dd245239b3909b13edb5ad](C2_wireshark.assets/cf1d14db6dd245239b3909b13edb5ad.png)

通过上图可以得知此DNS请求是通过UDP协议实现的，在UDP协议字段中可以获得Source Port和Destination Port，在IP字段可以看到Src为本机的IP地址，而Dst正是使用nslookup时默认的权威DNS服务器的IP地址。![b3881beba802c61d7d1840cf1b108ee](C2_wireshark.assets/b3881beba802c61d7d1840cf1b108ee.png)

观察DNS服务器的response

![0b752bff1ff55e1d5ff213cd3ee8d83](C2_wireshark.assets/0b752bff1ff55e1d5ff213cd3ee8d83.png)

可以得知此DNS query的类型是A（即回复主机名和其IP地址）。

Answers中包含三条信息，其中第一条类型为CNAME，可以获取主机的规范主机名。

![dfe56ca255c5c45fa7b73ab9b967eb4](C2_wireshark.assets/dfe56ca255c5c45fa7b73ab9b967eb4.png)

在紧接着的两个由我的主机发出的TCP SYN packet的目标IP地址正是上面DNS response报文中回复的IP地址。

##### nslookup


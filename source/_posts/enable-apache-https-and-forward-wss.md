---
title: CentOS 7 + Apache 配置https + wss转发
date: 2019-04-06 09:08:00
tags: 
  - centos7
  - apache
  - https
  - wss
categories: apache
---
### 安装openssl库

1.检查是否已安装 `openssl`

```bash
    openssl version -a
    # 打印出下面类似的信息表明已安装
    # OpenSSL 1.0.2k-fips  26 Jan 2017.....
    # built on: reproducible build, date unspecified
```
<!--more-->
>如果未安装，可以使用  `yum install openssl` 或参考其他方式

2.检查apache是否已安装ssl模块

> 查看  `/etc/httpd/conf.d/ssl.conf` 文件是否存在，如果不存在 使用命令安装 

```bash
    yum install mod_ssl
```

3.检查是否已启用apache的 `proxy`模块
    
>打开 `/etc/httpd/conf.modules.d/`文件夹，查看是否存在 `proxy.conf` 文件名(有可能是`00-proxy.conf`)的文件,如果不存在，可能需要重新编译安装apache，请联系服务器维护人员。如果存在查看时候有一下代码
```bash
    LoadModule proxy_module modules/mod_proxy.so
    LoadModule proxy_http_module modules/mod_proxy_http.so
    LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
    LoadModule proxy_connect_module modules/mod_proxy_connect.so
```
不存在请联系服务器维护人员

### 生成ssl证书

1.如果没有证书文件，可以创建本地文件测试
> 输入命令，生成2048位加密私钥 
```bash
    openssl genrsa -out server1.key 2048
```

> 然后输入命令，生成证书签名请求（CSR），这里需要填写许多信息，如国家，省市，公司，域名等，`域名` 必须填写正确，其他可以随便填写
```bash
    openssl req -new -key server1.key -out server1.csr
```

> 最后输入命令，生成类型为X509的自签名证书。有效期设置3650天，即有效期为10年
```bash
    openssl x509 -req -days 3650 -in server1.csr -signkey server1.key -out server1.crt
```

> 参考文件连接 https://www.linuxidc.com/Linux/2018-08/153555.htm

2.移动证书文件


> 在 `/etc/httpd/`文件夹下面创建一个 crt文件夹，将提供的证书文件移动到里面。

``` bash
    xxxx.crt # 证书文件
    xxxx.key # 证书秘钥文件
```

### 配置ssl

> 打开 `/etc/httpd/conf.d/ssl.conf` 文件，找到 `<VirtualHost _default_:443>` 这一行，在其后面添加


```apache ssl.conf
    DocumentRoot "/var/www/html" # 指向web的地址，一般不需要修改

    # 配置域名，域名必须和证书的名称一致 
    # 例如 aaaa.com  -> ServerName aaaa.com:443
    ServerName www.example.com:443 
```

> 然后找到 `SSLEngine on` 这一行代码，在下面添加

```apache ssl.conf
    SSLProxyEngine on
    ProxyRequests Off

    # 转发 streams流
    ProxyPass /streams http://127.0.0.1:6666/streams
    # 状态码302相关
    ProxyPassReverse /streams http://127.0.0.1:6666/streams

    # 转发wss 协议到 ws协议
    ProxyPass /wss ws://127.0.0.1:8888
    # 状态码302相关
    ProxyPassReverse /wss ws://127.0.0.1:8888
```

> 然后找到 `SSLHonorCipherOrder on`这行代码把前面的 # 号去掉

> 然后找到 `SSLCertificateFile /etc/pki/tls/certs/localhost.crt` 把里面的文件地址改为证书文件crt文件的地址`/etc/httpd/crt/xxxx.crt`

> 然后找到 `SSLCertificateKeyFile /etc/pki/tls/private/localhost.key` 把里面的文件地址改为证书秘钥文件key文件的地址`/etc/httpd/crt/xxxx.key`

> 然后保存，重启httpd服务 `systemctl restart httpd`


---
{% folding 配置过的ssl.conf文件 可参考 %}

```bash
    #
# When we also provide SSL we have to listen to the 
# the HTTPS port in addition.
#
Listen 443 https

##
##  SSL Global Context
##
##  All SSL configuration in this context applies both to
##  the main server and all SSL-enabled virtual hosts.
##

#   Pass Phrase Dialog:
#   Configure the pass phrase gathering process.
#   The filtering dialog program (`builtin' is a internal
#   terminal dialog) has to provide the pass phrase on stdout.
SSLPassPhraseDialog exec:/usr/libexec/httpd-ssl-pass-dialog

#   Inter-Process Session Cache:
#   Configure the SSL Session Cache: First the mechanism 
#   to use and second the expiring timeout (in seconds).
SSLSessionCache         shmcb:/run/httpd/sslcache(512000)
SSLSessionCacheTimeout  300

#   Pseudo Random Number Generator (PRNG):
#   Configure one or more sources to seed the PRNG of the 
#   SSL library. The seed data should be of good random quality.
#   WARNING! On some platforms /dev/random blocks if not enough entropy
#   is available. This means you then cannot use the /dev/random device
#   because it would lead to very long connection times (as long as
#   it requires to make more entropy available). But usually those
#   platforms additionally provide a /dev/urandom device which doesn't
#   block. So, if available, use this one instead. Read the mod_ssl User
#   Manual for more details.
SSLRandomSeed startup file:/dev/urandom  256
SSLRandomSeed connect builtin
#SSLRandomSeed startup file:/dev/random  512
#SSLRandomSeed connect file:/dev/random  512
#SSLRandomSeed connect file:/dev/urandom 512

#
# Use "SSLCryptoDevice" to enable any supported hardware
# accelerators. Use "openssl engine -v" to list supported
# engine names.  NOTE: If you enable an accelerator and the
# server does not start, consult the error logs and ensure
# your accelerator is functioning properly. 
#
SSLCryptoDevice builtin
#SSLCryptoDevice ubsec

##
## SSL Virtual Host Context
##

<VirtualHost _default_:443>

# General setup for the virtual host, inherited from global configuration
DocumentRoot "/var/www/html"
ServerName aaaaaaaaaa.com:443

# Use separate log files for the SSL virtual host; note that LogLevel
# is not inherited from httpd.conf.
ErrorLog logs/ssl_error_log
TransferLog logs/ssl_access_log
LogLevel warn

#   SSL Engine Switch:
#   Enable/Disable SSL for this virtual host.
SSLEngine on

SSLProxyEngine on

ProxyRequests Off
 
ProxyPass /streams http://127.0.0.1:3096/streams
 
ProxyPassReverse /streams http://127.0.0.1:3096/streams

ProxyPass /wss ws://127.0.0.1:3098
 
ProxyPassReverse /wss ws://127.0.0.1:3098

#   SSL Protocol support:
# List the enable protocol levels with which clients will be able to
# connect.  Disable SSLv2 access by default:
SSLProtocol all -SSLv2 -SSLv3

#   SSL Cipher Suite:
#   List the ciphers that the client is permitted to negotiate.
#   See the mod_ssl documentation for a complete list.
SSLCipherSuite HIGH:3DES:!aNULL:!MD5:!SEED:!IDEA

#   Speed-optimized SSL Cipher configuration:
#   If speed is your main concern (on busy HTTPS servers e.g.),
#   you might want to force clients to specific, performance
#   optimized ciphers. In this case, prepend those ciphers
#   to the SSLCipherSuite list, and enable SSLHonorCipherOrder.
#   Caveat: by giving precedence to RC4-SHA and AES128-SHA
#   (as in the example below), most connections will no longer
#   have perfect forward secrecy - if the server's key is
#   compromised, captures of past or future traffic must be
#   considered compromised, too.
#SSLCipherSuite RC4-SHA:AES128-SHA:HIGH:MEDIUM:!aNULL:!MD5
SSLHonorCipherOrder on 

#   Server Certificate:
# Point SSLCertificateFile at a PEM encoded certificate.  If
# the certificate is encrypted, then you will be prompted for a
# pass phrase.  Note that a kill -HUP will prompt again.  A new
# certificate can be generated using the genkey(1) command.
SSLCertificateFile /etc/httpd/crt/server1.crt

#   Server Private Key:
#   If the key is not combined with the certificate, use this
#   directive to point at the key file.  Keep in mind that if
#   you've both a RSA and a DSA private key you can configure
#   both in parallel (to also allow the use of DSA ciphers, etc.)
SSLCertificateKeyFile /etc/httpd/crt/server1.key

#   Server Certificate Chain:
#   Point SSLCertificateChainFile at a file containing the
#   concatenation of PEM encoded CA certificates which form the
#   certificate chain for the server certificate. Alternatively
#   the referenced file can be the same as SSLCertificateFile
#   when the CA certificates are directly appended to the server
#   certificate for convinience.
#SSLCertificateChainFile /etc/pki/tls/certs/server-chain.crt

#   Certificate Authority (CA):
#   Set the CA certificate verification path where to find CA
#   certificates for client authentication or alternatively one
#   huge file containing all of them (file must be PEM encoded)
#SSLCACertificateFile /etc/pki/tls/certs/ca-bundle.crt

#   Client Authentication (Type):
#   Client certificate verification type and depth.  Types are
#   none, optional, require and optional_no_ca.  Depth is a
#   number which specifies how deeply to verify the certificate
#   issuer chain before deciding the certificate is not valid.
#SSLVerifyClient require
#SSLVerifyDepth  10

#   Access Control:
#   With SSLRequire you can do per-directory access control based
#   on arbitrary complex boolean expressions containing server
#   variable checks and other lookup directives.  The syntax is a
#   mixture between C and Perl.  See the mod_ssl documentation
#   for more details.
#<Location />
#SSLRequire (    %{SSL_CIPHER} !~ m/^(EXP|NULL)/ \
#            and %{SSL_CLIENT_S_DN_O} eq "Snake Oil, Ltd." \
#            and %{SSL_CLIENT_S_DN_OU} in {"Staff", "CA", "Dev"} \
#            and %{TIME_WDAY} >= 1 and %{TIME_WDAY} <= 5 \
#            and %{TIME_HOUR} >= 8 and %{TIME_HOUR} <= 20       ) \
#           or %{REMOTE_ADDR} =~ m/^192\.76\.162\.[0-9]+$/
#</Location>

#   SSL Engine Options:
#   Set various options for the SSL engine.
#   o FakeBasicAuth:
#     Translate the client X.509 into a Basic Authorisation.  This means that
#     the standard Auth/DBMAuth methods can be used for access control.  The
#     user name is the `one line' version of the client's X.509 certificate.
#     Note that no password is obtained from the user. Every entry in the user
#     file needs this password: `xxj31ZMTZzkVA'.
#   o ExportCertData:
#     This exports two additional environment variables: SSL_CLIENT_CERT and
#     SSL_SERVER_CERT. These contain the PEM-encoded certificates of the
#     server (always existing) and the client (only existing when client
#     authentication is used). This can be used to import the certificates
#     into CGI scripts.
#   o StdEnvVars:
#     This exports the standard SSL/TLS related `SSL_*' environment variables.
#     Per default this exportation is switched off for performance reasons,
#     because the extraction step is an expensive operation and is usually
#     useless for serving static content. So one usually enables the
#     exportation for CGI and SSI requests only.
#   o StrictRequire:
#     This denies access when "SSLRequireSSL" or "SSLRequire" applied even
#     under a "Satisfy any" situation, i.e. when it applies access is denied
#     and no other module can change it.
#   o OptRenegotiate:
#     This enables optimized SSL connection renegotiation handling when SSL
#     directives are used in per-directory context. 
#SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire
<Files ~ "\.(cgi|shtml|phtml|php3?)$">
    SSLOptions +StdEnvVars
</Files>
<Directory "/var/www/cgi-bin">
    SSLOptions +StdEnvVars
</Directory>

#   SSL Protocol Adjustments:
#   The safe and default but still SSL/TLS standard compliant shutdown
#   approach is that mod_ssl sends the close notify alert but doesn't wait for
#   the close notify alert from client. When you need a different shutdown
#   approach you can use one of the following variables:
#   o ssl-unclean-shutdown:
#     This forces an unclean shutdown when the connection is closed, i.e. no
#     SSL close notify alert is send or allowed to received.  This violates
#     the SSL/TLS standard but is needed for some brain-dead browsers. Use
#     this when you receive I/O errors because of the standard approach where
#     mod_ssl sends the close notify alert.
#   o ssl-accurate-shutdown:
#     This forces an accurate shutdown when the connection is closed, i.e. a
#     SSL close notify alert is send and mod_ssl waits for the close notify
#     alert of the client. This is 100% SSL/TLS standard compliant, but in
#     practice often causes hanging connections with brain-dead browsers. Use
#     this only for browsers where you know that their SSL implementation
#     works correctly. 
#   Notice: Most problems of broken clients are also related to the HTTP
#   keep-alive facility, so you usually additionally want to disable
#   keep-alive for those clients, too. Use variable "nokeepalive" for this.
#   Similarly, one has to force some clients to use HTTP/1.0 to workaround
#   their broken HTTP/1.1 implementation. Use variables "downgrade-1.0" and
#   "force-response-1.0" for this.
BrowserMatch "MSIE [2-5]" \
         nokeepalive ssl-unclean-shutdown \
         downgrade-1.0 force-response-1.0

#   Per-Server Logging:
#   The home of a custom SSL log file. Use this when you want a
#   compact non-error SSL logfile on a virtual host basis.
CustomLog logs/ssl_request_log \
          "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"

</VirtualHost>                              
```

{% endfolding %}
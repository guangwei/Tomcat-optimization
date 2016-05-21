
Tomcat 优化 CentOS 6.x

1, Linux 网络参数优化

net.ipv4.tcp_syncookies = 1

net.ipv4.tcp_tw_reuse = 1

net.ipv4.tcp_tw_recycle = 1

net.ipv4.ip_local_port_range = 1024 65000

net.ipv4.tcp_max_syn_backlog = 8192

net.ipv4.tcp_max_tw_buckets = 5000

net.ipv4.tcp_max_syn_backlog = 65536

net.core.netdev_max_backlog = 32768

net.core.somaxconn = 32768

net.core.wmem_default = 8388608

net.core.rmem_default = 8388608

net.core.rmem_max = 16777216

net.core.wmem_max = 16777216

net.ipv4.tcp_timestamps = 0

net.ipv4.tcp_synack_retries = 2

net.ipv4.tcp_syn_retries = 2

net.ipv4.tcp_tw_recycle = 1

net.ipv4.tcp_tw_reuse = 1

net.ipv4.tcp_mem = 94500000 915000000 927000000

net.ipv4.tcp_max_orphans = 3276800

让改动立即生效 /sbin/sysctl -p

2, 修改ulimit vi /etc/security/limits.conf

soft nproc 65535

hard nproc 65535

soft nofile 65535

hard nofile 65535

vi /etc/security/limits.d/90-nproc.conf

soft nproc 65535

3, 修改JAVA VM启动参数 vi catalina.sh


JAVA_OPTS="-server -Xms1500M -Xmx1500M -Xss256K -Djava.awt.headless=true -Dfile.encoding=utf-8"

4, 启动APR提高性能

build apr

./configure --prefix=/home/runner/tools/apr

build apr-util

./configure --with-apr=/home/runner/tools/apr --prefix=/home/runner/tools/apr-util

build tomcat jni

cd /home/runner/tools/tomcat/bin/tomcat-native-1xx

./configure --with-apr=/home/runner/tools/apr/bin/apr-1-config --with-java-home=/home/runner/tools/jdk --prefix=/home/runner/tools/jdk

4, 数字证书配置

生成CSR 提交到 CA:

keytool -genkey -alias mykey -keyalg RSA -keystore yourwebsite.keystore -keysize 2048

从CA处下载CRT 导入到keystore中:

keytool -importcert -alias Root -trustcacerts -file AddTrustExternalCARoot.crt -keystore yourwebsite.keystore

keytool -importcert -alias Middle -trustcacerts -file COMODORSAAddTrustCA.crt -keystore yourwebsite.keystore

keytool -importcert -alias Verifier -trustcacerts -file COMODORSADomainValidationSecureServerCA.crt -keystore yourwebsite.keystore

keytool -import -trustcacerts -alias mykey -file yourwebsite.crt -keystore yourwebsite.keystore

storepass : yourpassword

用java keytool将 jks keystore 文件格式转换为 pkcs12格式:

keytool -importkeystore -srckeystore yourwebsite.keystore -destkeystore yourwebsite.p12 -srcstoretype JKS -deststoretype PKCS12 -srcstorepass yourpassword -deststorepass yourpassword -srcalias mykey -destalias mykey -srckeypass yourpassword -destkeypass yourpassword -noprompt

用openssl 将 pkcs12 转为 pem:

openssl pkcs12 -in yourwebsite.p12 -out yourwebsite.pem -nodes

修改 server.xml

    <Connector port="8443" 
               protocol="org.apache.coyote.http11.Http11AprProtocol"
               maxThreads="2048" 
               maxHttpHeaderSize="8192"
               URIEncoding="UTF-8"
               connectionTimeout="20000"
               minSpareThreads="20"
               acceptCount="1000"
               enableLookups="false"
               useBodyEncodingForURI="true"
               SSLCertificateKeyFile="/websecure/yourwebsite.pem"
               SSLPassword="yourpassword"
               SSLVerifyClient="false"
               SSLProtocol="TLSv1"
               SSLCertificateFile="/websecure/yourwebsite.crt" 
               SSLEnabled="true" 
               scheme="https" 
               secure="true"                
 />

vi server.xml

禁用 unpackWARs 和autoDeploy
<Host name="localhost"  appBase="webapps"  unpackWARs="false" autoDeploy="false">



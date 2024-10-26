# test_despliegues_DNS
1. Repositorio en GitHub
Creación del Repositorio: Se accedió a GitHub y se configuró un nuevo repositorio titulado "TEST_DESPLIEGUES_DNS".
Archivos Iniciales: Se crearon los archivos esenciales:
README.md: Documento explicativo que detalla el propósito de la práctica y proporciona instrucciones de ejecución. Incluye capturas de pantalla de los pasos importantes, especialmente de la transferencia de zona.
LICENSE: Archivo de licencia que especifica los términos de uso del proyecto, eligiendo una licencia adecuada.
.gitignore: Se añadió para evitar que el directorio .vagrant y los archivos de backup sean seguidos por Git, manteniendo el repositorio limpio.
Clonación del Repositorio: Se clonó el repositorio en el ordenador local utilizando el comando git clone, permitiendo trabajar con los archivos de forma local.

2. Datos del Problema
2.1. Red
Definición de la Red: Se estableció que todas las máquinas involucradas en la práctica estarían en la red 192.168.57.0/24, asegurando un esquema de red coherente.
2.2. Equipos
Configuración de Equipos: Se configuraron las siguientes máquinas virtuales en la red:
mercurio.sistema.test (IP: .101): Máquina Linux gráfico para tareas de usuario (IMAGINARIA).
venus.sistema.test (IP: .102): Máquina Debian en modo texto, configurada como servidor esclavo de DNS.
tierra.sistema.test (IP: .103): Máquina Debian en modo texto, actuando como servidor maestro de DNS.
marte.sistema.test (IP: .104): Máquina Windows (gráfico o server) que actúa como servidor de correo (IMAGINARIA).

2.3 Se estableció una configuración unitaria para ambas máquinas y la configuración individual.
3. Datos del DNS
Configuración del Servidor DNS:

Escucha para IPv4: Se configuró el servidor DNS para escuchar únicamente en la dirección IPv4 (192.168.57.103).
Validación de DNSSEC: Se estableció dnssec-validation como yes para asegurar las respuestas DNS.
Control de Acceso: Se aplicaron listas de control de acceso (ACL) para permitir consultas recursivas solo desde las redes 127.0.0.0/8 y 192.168.57.0/24.
Definición de Zonas:

Servidor Maestro: Se configuró tierra.sistema.test como el maestro, encargado de la zona directa e inversa. Configuración del archivo named.conf.local de tierra.
Servidor Esclavo: Se configuró venus.sistema.test como esclavo, que recibe transferencias de zona desde el servidor maestro. Configuración del archivo named.conf.local de venus.
Configuración de Cache y Reenvío:

Tiempo de Cache: El tiempo de caché para respuestas negativas se estableció en 7200 segundos => max-ncache-ttl 7200 en el archivo named.conf.options.
Reenvío de Consultas: Se configuró el reenvío de consultas no autorizadas al servidor DNS público de OpenDNS (208.67.222.222) => configuración del forwarders en el archivo named.conf.options con la dirección del DNS
Configuración de Alias: Se definieron varios alias para simplificar la gestión del DNS:

ns1.sistema.test como alias de tierra.sistema.test. en db.sistema.test
ns2.sistema.test como alias de venus.sistema.test. en db.sistema.test
mail.sistema.test como alias de marte.sistema.test, que también actuará como servidor de correo para el dominio, también en db.sistema.test. En este caso, con la opción 
4. Comprobación
Verificación de Resolución DNS:

Se utilizaron herramientas como dig y nslookup para comprobar la resolución de registros tipo A, asegurando que las IPs correspondieran a los FQDNs configurados.
Se realizaron pruebas de resolución inversa para verificar que las direcciones IP resolviesen correctamente a sus nombres de dominio.
Resolución de Alias y Consultas:

Se comprobaron los alias ns1.sistema.test y ns2.sistema.test, verificando que apuntaran a las direcciones correctas.
Se consultaron los registros NS y MX para sistema.test, confirmando que los servidores listados eran tierra.sistema.test y venus.sistema.test.
Transferencia de Zona:

Se intentó realizar una transferencia de zona usando el comando AXFR para comprobar que venus.sistema.test pudiera recibir la configuración del maestro. Inicialmente, la transferencia falló, lo que se revisó posteriormente en los logs.
Se utilizó el comando sudo tail -f /var/log/named/bind.log para monitorear los logs del servidor y detectar cualquier error o notificación relacionada con las transferencias de zona.
Se comprobó que el error estaba en comprobar el AXFR en el maestro en vez de en el esclavo. 
Comprobación de Respuestas:

Finalmente, se verificó que tanto el servidor maestro como el esclavo pudieran responder a las mismas preguntas DNS, confirmando la sincronización de las configuraciones entre ambos. 


5. Formación de archivos para Vagrant
Una vez comprobado que todo funciona correctamente, se crearon los archivos para poder cargarlos desde Vagrant al iniciar las máquinas virtuales para que puedan arrancar con la configuración deseada.


6. Comprobaciones con test.sh y dig


 ./test.sh 192.168.57.103
+ set -euo pipefail
+ nameserver=@192.168.57.103
+ resolver mercurio.sistema.test
+ dig @192.168.57.103 +short mercurio.sistema.test
192.168.57.101
+ resolver venus.sistema.test
+ dig @192.168.57.103 +short venus.sistema.test
192.168.57.102
+ resolver tierra.sistema.test
+ dig @192.168.57.103 +short tierra.sistema.test
192.168.57.103
+ resolver marte.sistema.test
+ dig @192.168.57.103 +short marte.sistema.test
192.168.57.104
+ resolver ns1.sistema.test
+ dig @192.168.57.103 +short ns1.sistema.test
tierra.sistema.test.
192.168.57.103
+ resolver ns2.sistema.test
+ dig @192.168.57.103 +short ns2.sistema.test
venus.sistema.test.
192.168.57.102
+ resolver sistema.test mx
+ dig @192.168.57.103 +short sistema.test mx
10 marte.sistema.test.
+ resolver sistema.test ns
+ dig @192.168.57.103 +short sistema.test ns
venus.sistema.test.
tierra.sistema.test.
+ resolver -x 192.168.57.101
+ dig @192.168.57.103 +short -x 192.168.57.101
mercurio.sistema.test.
+ resolver -x 192.168.57.102
+ dig @192.168.57.103 +short -x 192.168.57.102
venus.sistema.test.
+ resolver -x 192.168.57.103
+ dig @192.168.57.103 +short -x 192.168.57.103
tierra.sistema.test.
+ resolver -x 192.168.57.104
+ dig @192.168.57.103 +short -x 192.168.57.104
marte.sistema.test.


dig @192.168.57.103 tierra.sistema.test A

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.103 tierra.sistema.test A  
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33230
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7bd78f1f92e561a401000000671b818b0c36a5d6727b9109 (good)
;; QUESTION SECTION:
;tierra.sistema.test.           IN      A

;; ANSWER SECTION:
tierra.sistema.test.    86400   IN      A       192.168.57.103

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Fri Oct 25 11:31:23 UTC 2024
;; MSG SIZE  rcvd: 92


dig @192.168.57.103 venus.sistema.test A

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.103 venus.sistema.test A   
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64373
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 13e515162a71231d01000000671b81ebb3e766e46013588c (good)
;; QUESTION SECTION:
;venus.sistema.test.            IN      A

;; ANSWER SECTION:
venus.sistema.test.     86400   IN      A       192.168.57.102

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Fri Oct 25 11:32:59 UTC 2024
;; MSG SIZE  rcvd: 91

dig @192.168.57.103 marte.sistema.test A

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.103 marte.sistema.test A   
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58547
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 3d5942e7b5e08fd601000000671b8261368baf025203d668 (good)
;; QUESTION SECTION:
;marte.sistema.test.            IN      A

;; ANSWER SECTION:
marte.sistema.test.     86400   IN      A       192.168.57.104

;; Query time: 4 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Fri Oct 25 11:34:57 UTC 2024
;; MSG SIZE  rcvd: 91


dig @192.168.57.103 mercurio.sistema.test A

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.103 mercurio.sistema.test A
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18490
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: ef9ce1ede263759101000000671b82884bf030123ea59e05 (good)
;; QUESTION SECTION:
;mercurio.sistema.test.         IN      A

;; ANSWER SECTION:
mercurio.sistema.test.  86400   IN      A       192.168.57.101

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Fri Oct 25 11:35:36 UTC 2024
;; MSG SIZE  rcvd: 94


dig @192.168.57.103 ns1.sistema.test CNAME

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.103 ns1.sistema.test CNAME 
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26183
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: eec4ad78e2cbf8b501000000671b840ba99d9fdb490c9cf3 (good)
;; QUESTION SECTION:
;ns1.sistema.test.              IN      CNAME

;; ANSWER SECTION:
ns1.sistema.test.       86400   IN      CNAME   tierra.sistema.test.

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Fri Oct 25 11:42:03 UTC 2024
;; MSG SIZE  rcvd: 94


dig @192.168.57.103 ns2.sistema.test CNAME

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.103 ns2.sistema.test CNAME 
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50396
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a27e38255695583a01000000671b84577727866e9999f501 (good)
;; QUESTION SECTION:
;ns2.sistema.test.              IN      CNAME

;; ANSWER SECTION:
ns2.sistema.test.       86400   IN      CNAME   venus.sistema.test.

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Fri Oct 25 11:43:19 UTC 2024
;; MSG SIZE  rcvd: 93


dig @192.168.57.103 sistema.test NS

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.103 sistema.test NS        
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2237
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b9d097104cecf66501000000671b8493db445864d5df2089 (good)
;; QUESTION SECTION:
;sistema.test.                  IN      NS

;; ANSWER SECTION:
sistema.test.           86400   IN      NS      tierra.sistema.test.
sistema.test.           86400   IN      NS      venus.sistema.test.

;; ADDITIONAL SECTION:
venus.sistema.test.     86400   IN      A       192.168.57.102
tierra.sistema.test.    86400   IN      A       192.168.57.103

;; Query time: 4 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Fri Oct 25 11:44:19 UTC 2024
;; MSG SIZE  rcvd: 142



dig @192.168.57.103 sistema.test MX

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.103 sistema.test MX        
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45160
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 6f62dbda2a092dd901000000671b84ad4ad2972fc4187bee (good)
;; QUESTION SECTION:
;sistema.test.                  IN      MX

;; ANSWER SECTION:
sistema.test.           86400   IN      MX      10 mail.sistema.test.

;; ADDITIONAL SECTION:
mail.sistema.test.      86400   IN      A       192.168.57.104

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Fri Oct 25 11:44:45 UTC 2024
;; MSG SIZE  rcvd: 106

;;COMPROBAMOS QUE SE RESPONDEN DE IGUAL MANERA A LAS MISMAS PREGUNTAS

dig @192.168.57.102 tierra.sistema.test A

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.102 tierra.sistema.test A
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22383
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 03a08d9516de637001000000671b8a5e88f33c7ecc95fa11 (good)
;; QUESTION SECTION:
;tierra.sistema.test.           IN      A

;; ANSWER SECTION:
tierra.sistema.test.    86400   IN      A       192.168.57.103

;; Query time: 4 msec
;; SERVER: 192.168.57.102#53(192.168.57.102) (UDP)
;; WHEN: Fri Oct 25 12:09:02 UTC 2024
;; MSG SIZE  rcvd: 92

dig @192.168.57.102 sistema.test MX

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.102 sistema.test MX
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13301
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a6f7ceedc543ca3d01000000671b8ac1769f707e8755c0e0 (good)
;; QUESTION SECTION:
;sistema.test.                  IN      MX

;; ANSWER SECTION:
sistema.test.           86400   IN      MX      10 mail.sistema.test.

;; ADDITIONAL SECTION:
mail.sistema.test.      86400   IN      A       192.168.57.104

;; Query time: 0 msec
;; SERVER: 192.168.57.102#53(192.168.57.102) (UDP)
;; WHEN: Fri Oct 25 12:10:41 UTC 2024
;; MSG SIZE  rcvd: 106

 dig @192.168.57.103 sistema.test AXFR

; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @192.168.57.103 sistema.test AXFR
; (1 server found)
;; global options: +cmd
sistema.test.           86400   IN      SOA     tierra.sistema.test. admin.sistema.test. 2023102501 3600 1800 1209600 86400
sistema.test.           86400   IN      MX      10 mail.sistema.test.
sistema.test.           86400   IN      NS      venus.sistema.test.
sistema.test.           86400   IN      NS      tierra.sistema.test.
mail.sistema.test.      86400   IN      A       192.168.57.104
marte.sistema.test.     86400   IN      A       192.168.57.104
mercurio.sistema.test.  86400   IN      A       192.168.57.101
ns1.sistema.test.       86400   IN      CNAME   tierra.sistema.test.
ns2.sistema.test.       86400   IN      CNAME   venus.sistema.test.
tierra.sistema.test.    86400   IN      A       192.168.57.103
venus.sistema.test.     86400   IN      A       192.168.57.102
sistema.test.           86400   IN      SOA     tierra.sistema.test. admin.sistema.test. 2023102501 3600 1800 1209600 86400
;; Query time: 3 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (TCP)
;; WHEN: Fri Oct 25 15:55:30 UTC 2024
;; XFR size: 12 records (messages 1, bytes 340)
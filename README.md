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


6. Comprobación con el test.sh
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
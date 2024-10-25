# test_despliegues_DNS
1. Repositorio en GitHub
Creación del Repositorio: Se accedió a GitHub y se configuró un nuevo repositorio titulado "sistema-test-practica".
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
mercurio.sistema.test (IP: .101): Máquina Linux gráfico para tareas de usuario.
venus.sistema.test (IP: .102): Máquina Debian en modo texto, configurada como servidor esclavo de DNS.
tierra.sistema.test (IP: .103): Máquina Debian en modo texto, actuando como servidor maestro de DNS.
marte.sistema.test (IP: .104): Máquina Windows (gráfico o server) que actúa como servidor de correo.
3. Datos del DNS
Configuración del Servidor DNS:

Escucha para IPv4: Se configuró el servidor DNS para escuchar únicamente en la dirección IPv4 (192.168.57.103).
Validación de DNSSEC: Se estableció dnssec-validation como yes para asegurar las respuestas DNS.
Control de Acceso: Se aplicaron listas de control de acceso (ACL) para permitir consultas recursivas solo desde las redes 127.0.0.0/8 y 192.168.57.0/24.
Definición de Zonas:

Servidor Maestro: Se configuró tierra.sistema.test como el maestro, encargado de la zona directa e inversa.
Servidor Esclavo: Se configuró venus.sistema.test como esclavo, que recibe transferencias de zona desde el servidor maestro.
Configuración de Cache y Reenvío:

Tiempo de Cache: El tiempo de caché para respuestas negativas se estableció en 7200 segundos.
Reenvío de Consultas: Se configuró el reenvío de consultas no autorizadas al servidor DNS público de OpenDNS (208.67.222.222).
Configuración de Alias: Se definieron varios alias para simplificar la gestión del DNS:

ns1.sistema.test como alias de tierra.sistema.test.
ns2.sistema.test como alias de venus.sistema.test.
mail.sistema.test como alias de marte.sistema.test, que también actuará como servidor de correo para el dominio.
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
Comprobación de Respuestas:

Finalmente, se verificó que tanto el servidor maestro como el esclavo pudieran responder a las mismas preguntas DNS, confirmando la sincronización de las configuraciones entre ambos.

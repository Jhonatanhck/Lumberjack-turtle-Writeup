
Reconocimiento

![[Pasted image 20260111191529.png]]
Obtenemos esto del reconocimiento de nmap, voy a enumerar la pagina web

![[Pasted image 20260111191601.png]]
tenemos esto, nos dice que busquemos mas profundo asi que voy a utilizar la herramienta gobuster para buscar mas directorios

![[Pasted image 20260111194350.png]]
estos fueron los directorios que encontro

![[Pasted image 20260111191818.png]]
cuando entro me aparece esta pagina de error, el error parece ser de java, por lo que esta corriendo java por detras

![[Pasted image 20260111194436.png]]
en el otro directorio tenemos esto asi que hare otro reconocimiento 

![[Pasted image 20260111194548.png]]
y aqui encontramos este directorio que parece una pista de lo que tenemos que hacer ya sabemos que corre java, y ahora tenemos el nombre de una vulnerabilidad web de java

![[Pasted image 20260111194535.png]]
dentro del directorio nos encontramos esto 

![[Pasted image 20260111194752.png]]
para probar la vulnerabilidad estare usando este payload

![[Pasted image 20260111194932.png]]
![[Pasted image 20260111194944.png]]
en la cabecera accept nos mando una senal al netcat asi que es vulnerable 

para obtener una shell encontre este recurso https://github.com/christophetd/log4shell-vulnerable-app/blob/main/README.md

![[Pasted image 20260111203530.png]]
lo primero que tenemos que hacer es ejecutar el script para crear un servidor ldap

![[Pasted image 20260111203556.png]]
luego preparar nuestro listener

![[Pasted image 20260111203613.png]]
ahora ejecutar este payload con una reverse shell en base64 en el campo vulnerable

Le damos a send

![[Pasted image 20260111203725.png]]
y ya tenemos una shell 

al parecer estamos dentro de un contenedor, asi que debemos salir de el para vulnerar la maquina


![[Pasted image 20260111204712.png]]
enumerando con linpeas encontre estos dos binaros interesantes con el que podemos montarnos el disco duro de la maquina donde corre el docker

![[Pasted image 20260111210025.png]]
encontramos la primera flag en este directorio

![[Pasted image 20260111210101.png]]
como somos root podemos ver el disco duro de la maquina host y como tenemos permisos para montarlo con el binario mount nos podemos aprovechar de eso para escalar privilegios 


![[Pasted image 20260111210223.png]]
primero creamos una carpeta para montar el disco duro ahi

![[Pasted image 20260111210257.png]]
montamos el disco duro en el directorio que acabamos de crear

![[Pasted image 20260111210332.png]]
y aqui tenemos toda la raiz de la maquina host
ya tenemos la maquina, ahora como escapamos?

lo primero es que nos tenemos que crear dos pares de llaves con ssh-keygen -t rsa

![[Pasted image 20260111210953.png]]
ahora nos copiamos esto 

![[Pasted image 20260111211322.png]]
y lo tenemos que meter en la authorized_keys de la maquina victima

![[Pasted image 20260111211450.png]]
y ahora solo nos conectamos y ya estamos en la maquina victima, PWNED

![[Pasted image 20260111211548.png]]
y aqui tenemos la segunda flag 

# Que aprendi

### 1. Explotación de Log4Shell (CVE-2021-44228)

- **Vector de ataque en Headers:** Aprendiste que las vulnerabilidades no siempre están en los campos de entrada visibles (inputs de login), sino que pueden estar en cabeceras HTTP "invisibles" como `User-Agent` o `X-Api-Version`.
    
- **Inyección JNDI:** Entendiste cómo funciona el protocolo LDAP para engañar al servidor. No ejecutaste el código directamente; obligaste al servidor a conectarse a ti (`jndi:ldap://`) para descargar y ejecutar la clase maliciosa.
    
- **Evasión/Bypass:** Viste que a veces hay que modificar el payload (codificación Base64 en el comando) para que funcione.
    

### 2. "Dependency Hell" y Troubleshooting (Resolución de problemas)

- Esto fue clave en tu proceso. Aprendiste que **las herramientas de hacking envejecen**.
    
- Te enfrentaste a un error de Java (`IllegalAccessError`) porque `JNDIExploit` requería Java 8 y tu sistema tenía una versión moderna.
    
- **Lección vital:** Un hacker no se rinde cuando la herramienta falla; busca la dependencia correcta (descargar el JDK 8 portátil) y adapta su entorno.
    

### 3. Enumeración de Contenedores (Docker Enumeration)

- Aprendiste a identificar que estabas en un contenedor (probablemente por el hostname extraño o archivos como `.dockerenv`).
    
- **La clave:** El comando `fdisk -l`. Aprendiste que ver particiones del sistema host (como `/dev/nvme0n1` o `/dev/sda1`) desde dentro de un contenedor es una anomalía crítica.
    

### 4. Docker Breakout (Escapar del contenedor)

Esta es la joya de la corona de esta máquina.

- **Privileged Containers:** Explotaste un contenedor configurado con el flag `--privileged`. Esto otorga al contenedor acceso a los dispositivos del host.
    
- **Montaje de discos:** Aprendiste que si puedes ver el disco, puedes montarlo (`mount /dev/sda1 /mnt`).
    
- **Concepto:** El sistema de archivos del host se vuelve simplemente una carpeta más para ti.
    

### 5. Manipulación de SSH (Persistencia/Escalada)

- En lugar de intentar crackear hashes de contraseñas (que podría tardar años), utilizaste una técnica de **abuso de confianza**.
    
- Al tener escritura en el disco montado, inyectaste tu propia **llave pública** en `authorized_keys`.
    
- Esto te enseñó cómo convertir un acceso de archivos (File Write) en una ejecución de comandos (RCE/Shell) como root.

















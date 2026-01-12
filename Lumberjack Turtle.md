
Aquí tienes tu writeup formateado en Markdown listo para copiar y pegar en GitHub (como en un archivo README.md).

He realizado los siguientes cambios para asegurarme de que funcione perfecto:

    Formato de imágenes: He cambiado ![[imagen.png]] por ![Descripcion](images/imagen.png).

    Espacios en nombres: He reemplazado los espacios en los nombres de archivo por %20 (ejemplo: Pasted%20image...), ya que GitHub a veces falla si dejas espacios en blanco en las rutas de las imágenes.

    Estructura: He añadido algunos títulos para separar las fases (Reconocimiento, Explotación, Escalada).

Solo asegúrate de que tu estructura de carpetas en GitHub sea así:
Plaintext

/ (directorio raíz del repo)
├── README.md  (o el nombre de tu archivo)
└── images/    (carpeta con todas las fotos)

Aquí tienes el código:
Markdown

# Writeup: Lumberjack Turtle

## Reconocimiento

Obtenemos esto del reconocimiento de nmap, voy a enumerar la página web:

![Resultado Nmap](images/Pasted%20image%2020260111191529.png)

Al entrar tenemos esto, nos dice que busquemos más profundo, así que voy a utilizar la herramienta gobuster para buscar más directorios:

![Pagina Web](images/Pasted%20image%2020260111191601.png)

Estos fueron los directorios que encontró:

![Gobuster Scan](images/Pasted%20image%2020260111194350.png)

Cuando entro me aparece esta página de error. El error parece ser de Java, por lo que está corriendo Java por detrás:

![Error Java](images/Pasted%20image%2020260111191818.png)

En el otro directorio tenemos esto, así que haré otro reconocimiento:

![Directorio Logs](images/Pasted%20image%2020260111194436.png)

Y aquí encontramos este directorio que parece una pista de lo que tenemos que hacer. Ya sabemos que corre Java, y ahora tenemos el nombre de una vulnerabilidad web de Java:

![Pista Log4j](images/Pasted%20image%2020260111194548.png)

Dentro del directorio nos encontramos esto:

![Pagina Log4j](images/Pasted%20image%2020260111194535.png)

## Explotación (Log4Shell)

Para probar la vulnerabilidad estaré usando este payload:

![Payload Test](images/Pasted%20image%2020260111194752.png)

En la cabecera `Accept` nos mandó una señal al netcat, así que es vulnerable:

![Netcat Connection](images/Pasted%20image%2020260111194932.png)
![Burp Response](images/Pasted%20image%2020260111194944.png)

Para obtener una shell encontré este recurso: [Log4Shell Vulnerable App](https://github.com/christophetd/log4shell-vulnerable-app/blob/main/README.md)

Lo primero que tenemos que hacer es ejecutar el script para crear un servidor LDAP:

![JNDI Server](images/Pasted%20image%2020260111203530.png)

Luego preparar nuestro listener:

![Netcat Listener](images/Pasted%20image%2020260111203556.png)

Ahora ejecutar este payload con una reverse shell en base64 en el campo vulnerable. Le damos a send:

![Burp Exploit](images/Pasted%20image%2020260111203613.png)

Y ya tenemos una shell:

![Reverse Shell](images/Pasted%20image%2020260111203725.png)

## Post-Explotación & Docker Breakout

Al parecer estamos dentro de un contenedor, así que debemos salir de él para vulnerar la máquina.

Enumerando con `linpeas` encontré estos dos binarios interesantes con el que podemos montarnos el disco duro de la máquina donde corre el docker:

![Linpeas Fdisk](images/Pasted%20image%2020260111204712.png)

Encontramos la primera flag en este directorio:

![User Flag Location](images/Pasted%20image%2020260111210025.png)
![User Flag](images/Pasted%20image%2020260111210101.png)

Como somos root podemos ver el disco duro de la máquina host y como tenemos permisos para montarlo con el binario `mount` nos podemos aprovechar de eso para escalar privilegios.

Primero creamos una carpeta para montar el disco duro ahí:

![Mkdir](images/Pasted%20image%2020260111210223.png)

Montamos el disco duro en el directorio que acabamos de crear:

![Mount Command](images/Pasted%20image%2020260111210257.png)

Y aquí tenemos toda la raíz de la máquina host:

![Host Root](images/Pasted%20image%2020260111210332.png)

### Escalada de Privilegios (SSH Injection)

Ya tenemos la máquina, ahora ¿cómo escapamos?

Lo primero es que nos tenemos que crear dos pares de llaves con `ssh-keygen -t rsa`:

![SSH Keygen](images/Pasted%20image%2020260111210953.png)

Ahora nos copiamos esto:

![Copy Pub Key](images/Pasted%20image%2020260111211322.png)

Y lo tenemos que meter en el archivo `authorized_keys` de la máquina víctima (accesible a través del disco montado):

![Echo Key](images/Pasted%20image%2020260111211450.png)

Y ahora solo nos conectamos y ya estamos en la máquina víctima. **PWNED**:

![SSH Login](images/Pasted%20image%2020260111211548.png)

Y aquí tenemos la segunda flag.















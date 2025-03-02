
> ![[Pasted image 20250228155557-1.png]]

**En esta máquina se va al grano**
Levantamos la máquina 

![[Pasted image 20250228155413-1.png]]



## Reconocimiento 
Ejecutamos un escaneo completo de `nmap` para saber los puertos de la máquina junto a sus versiones

![[Pasted image 20250228155234-1.png]]

Por ahora solo tiene una web por lo mientras la veo en paralelo hago fuzzing de ficheros y directorios con gobuster

![[Pasted image 20250228155327-1.png]]



Tenemos la siguiente web:
![[Pasted image 20250228160922-1.png]]

Nos registramos y nos logeamos en este panel (No es vulnerable a SQLI al parecer)





![[Pasted image 20250228161048-1.png]]

![[Pasted image 20250228161022-1.png]]

Una vez registrados, en "_Mi carrito_" tenemos un panel de búsqueda:
![[Pasted image 20250228161113-1.png]]
Probando, este panel de vulnerable a sqli: 

![[Pasted image 20250228161156-1.png]]


![[Pasted image 20250228161214-1.png]]


## Explotación

Vamos a empezar por un ordenamiento de las columnas, adelante que el límite está en 5:
![[Pasted image 20250228164743-1.png]]


Ahora que sabemos las columnas, vamos a usar `union select` para mostrar más datos, en este caso, la base de datos en uso
![[Pasted image 20250228164813-1.png]]

Bien, parece que es una sqli basada en UNION SELECT ATTACK. Ahora sacamos todas las bases de datos.

Probando en cada una de las columnas, la nº 5 es la única que no me daba error, pero sin embargo, no me reportaba nada por lo que llevo la petición a Burpsuite para ver que pasa:

![[Pasted image 20250228165022-1.png]]


![[Pasted image 20250227212609-1.png]]

Lo que estaba pasando es que está representando los datos en el link de la imagen, por lo que seguimos con burp desde ahora

Sacar las tablas:
![[Pasted image 20250227213138-1.png]]


![[Pasted image 20250227213203-1.png]]

Sacar las columnas de la tabla users:

![[Pasted image 20250227214653-1.png]]


![[Pasted image 20250227214731-1.png]]


Sacar la data de la tabla users: 

![[Pasted image 20250227215010-1.png|1243]]


![[Pasted image 20250227215156-1.png]]


Tenemos las contraseñas y usuarios, ahora con `hash-identifier` vemos que tipo de hash es:

![[Pasted image 20250227215135-1.png]]



Estamos entre 2 hashes pero al ser mysql tiene pinta de que es el SHA-1 por lo que tiramos de `john` para crackearlo usando rockyou.txt si es que la contraseña esta contemplada en el rockyou.

![[Pasted image 20250227215340-1.png]]

Lo sacamos
![[Pasted image 20250227215401-1.png]]
Probando, de momento no sirve, asi que tiramos la del usuario admin 

![[Pasted image 20250227215606-1.png]]


Esta si que sirve para loguearnos en el panel de antes teniendo ahora una opción para un panel de administración:

![[Pasted image 20250227215624-1.png]]
En este, en el apartado de "_Configuración_" tenemos la posibilidad de subir archivos: 

![[Pasted image 20250227215640-1.png]]


![[Pasted image 20250227215706-1.png]]


![[Pasted image 20250227215714-1.png]]


Como es php, creamos un .php para conceder ejecución de comandas por el método GET mediante el parámetro "cmd", un clásico

![[Pasted image 20250227215803-1.png|627]]


![[Pasted image 20250227215835-1.png]]

Parece que hay que bypasearlo: 

![[Pasted image 20250227220034-1.png]]

Con un simple `phtml` sirve
![[Pasted image 20250227220256-1.png]]

En el directorio /uploads que nos reporto gobuster antes vemos que se subió el archivo

![[Pasted image 20250227220312-1.png]]


Ahora que funciona, nos ponemos a la escucha con `nc` y hacemos la típica:
![[Pasted image 20250227220431-1.png]]


## Escalada

Una vez dentro, sabiendo que hay archivos de configuración y para ir al grano uso grep para filtrar por la palabra password y nos sale algo:
![[Pasted image 20250227221929-1.png]]

Esta contraseña sirve para MYSQL pero adelanto que no encontramos nada allí 
![[Pasted image 20250227222125-1.png]]
Sabiendo la existencia del usuario _luisillo_o_ (lo vi antes en el /etc/passwd), en /tmp me traigo `suBF` para hace fuerza bruta de este usuario acompañado del rockyou que lo proporciono desde mi máquina mediante un servidor en python y con ayuda de `wget`:

![[Pasted image 20250227223541-1.png]]


![[Pasted image 20250227223312-1.png]]
Tras un rato largo, nos saca la contraseña de luisillo_o:


![[Pasted image 20250228161231-1.png]]![[Pasted image 20250228161518-1.png]]
Después, haciendo un id vemos que estamos en el grupo shadow y filtrando por archivos con este grupo vemos que podemos leer el /etc/shadow

![[Pasted image 20250228162709-1.png]]


![[Pasted image 20250228162730-1.png]]

Aquí podemos ver la contraseña de root

![[Pasted image 20250228164352-1.png]]

la intentamos crackear con `john` como antes :
![[Pasted image 20250228164413-1.png]]

`john` si que la saca (**Perdí la captura** pero es _rainbow2_) y somos root
![[Pasted image 20250228164606-1.png]]


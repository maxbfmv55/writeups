ChocolateFire - DockerLabs

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.001.png)














**Autor: Maximiliano Espinoza | Fecha: 09 de Octubre, 2025**

**Dificultad: Media / Muy Facil**

**Sistema Operativo: Linux**

**Tecnología: Web (Openfire)**






### **Tabla de Contenido**
1. [Información](https://ctf-writeup-builder.vercel.app/#section-preview-accordion-trigger-aae35016-3eba-4f91-aff3-cc3347405c39)
1. [Reconocimiento con nmap](https://ctf-writeup-builder.vercel.app/#section-preview-accordion-trigger-183c06ba-dc35-49d6-b352-0c47ce4306e5)
1. [Fingerprinting Web (Reconocimiento Web)](https://ctf-writeup-builder.vercel.app/#section-preview-accordion-trigger-cea2a0f7-5ae1-4c99-8734-86499459261e)
1. Fuerza bruta con Hydra
1. Escalada de privilegios.

[](https://ctf-writeup-builder.vercel.app/#section-preview-accordion-trigger-ca11e817-b738-4555-a1cb-1a5f8f747f61)




**1. Información**

En esta máquina “CholateFire” vamos a practicar un flujo realista de pentesting web y post-explotación centrado en **Openfire**. El objetivo es recorrer todas las etapas: reconocimiento de red y servicios, enumeración web, fuerza bruta de credenciales, bypass/explotación del panel de administración y finalmente el uso de frameworks para facilitar la explotación.

**2. Reconocimiento con nmap**

Una vez desplegado el docker de la maquina, procedemos a realizar un reconocimiento con nmap, pero antes que nada, realizamos un ping para comprobar conectividad con la maquina victima.

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.002.png)**	

**sudo nmap -sS -sV -Pn -T4 -p- --open -oA chocolatefire 172.17.0.2\
![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.003.png)**

Vemos que corre un servicio en el 9090. Asi que procedemos a investigar.

**3. Fingerprinting Web (Reconocimiento Web)**

Empezamos por hacer un reconocimiento web, abrimos la pagina en nuestro navegador para ver que nos encontramos:

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.004.png)










**	

Nos encontramos con un web panel login de Openfire en su version 4.7.4. Sabiendo esto podemos buscar algun exploit o probar credenciales por defecto.

NOTA 1: Openfire es un servidor de mensajería (chat) en tiempo real que utiliza el protocolo **XMPP** (también conocido como Jabber)

NOTA 2: en este caso vamos a realizar dos ataques, prueba de credenciales por defecto y un exploit.

Como vimos en pantalla las credenciales por defecto son admin:admin.

Ahora pasaremos a hacer uso del exploit, el cual se explota el CVE-2023-32315

<https://github.com/miko550/CVE-2023-32315>#

<https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT>

exploit A

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.005.png)

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.006.png)	




Exploit B

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.007.png)	





![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.008.png)






Bueno una vez dentro de la pagina de administración de Openfire, y habiendo ya enumerado los usuarios podemos hacer un ataque de fuerza bruta con hydra para poder acceder por ssh con alguno de los 3 primeros usuarios.

**4. Fuerza bruta con Hydra.**

Bueno una vez que analizamos la pagina y encontramos esos 3 user probamos cual podemos vulnerar con hydra. En este caso usamos el user “chocatitochingon”

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.009.png)	






**5. Escalada de prilegios (dos formas)**

Bueno vamos a demostrar primero la forma mas “dificil” para que practiquemos el uso de consola y moverse con mas soltura.

Una vez que nos conectamos por ssh con el usuario, realizamos la enumeración interna que se realiza comúnmente.

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.010.png)	

Como observamos tenemos dos usuarios en el home, uno es pinguinacio.

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.011.png)

Como vemos podemos hacer uso de /usr/bin/dpkg como el usuario pinguinacio.

Con esto podemos realizar un pivoting de usuario. De chocolatitochingon a pinguinacio.


Ejecutamos el binario dpkg

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.012.png)	

cuando ejecutamos nos aparece el menu de ayuda, podemos escapar simplemente tipiando !/bin/bash y le damos enter, esto nos da acceso a la terminal de pinguinacio.	



![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.013.png)	

Ahora que estamos como pinguinacio, procedemos a invetigar y escalar privilegios.

Luego de realizar un ls -la, nos encontramos con un script.sh, procedemos a leer el script con cat ya que tenemos permiso de lectura sobre el mismo.

Volvemos a ejecutar sudo -l sobre pinguinacio y nos encontramos con que podemos ejecutar como root el script.sh








La vulnerabilidad del script radica en que la shell realiza **expansión de comandos** ($(...)) al evaluar la línea del read y antes de la evaluación condicional. Aunque la comparación aritmética -eq usa comillas en "$numero", la sustitución $(/bin/sh) ocurre en la fase de expansión de la shell, y por tanto se ejecuta con los privilegios del proceso padre (que es root cuando ejecutamos sudo /bin/bash ...).

- En resumen: input no validado + ejecución del script como root → posibilidad de ejecutar $(...) como root.

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.014.png)	

NOTA: esta pagina muestra como hacerlo → https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/bash-eq/

Metodo “muy facil”, usando metasploit

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.015.png)	

![](Aspose.Words.9796b73b-72c1-4805-9945-b9b9e2ecd4a5.016.png)	

Con esto ya terminamos de comprometer la maquina “ChocolateFire” como vimos, hay varias formas de escalar privilegios.

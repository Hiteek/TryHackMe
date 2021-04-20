# Sala virtual Attacktive Directory
El 99% de las redes corporativas funcionan con AD. Pero, ¿Puedes explotar un Domain Controller vulnerable?

## Conceptos previos

## Programas necesarios

### Impacket
#### ¿Qué es Impacket?

Impacket es una colección de clases de Python para trabajar con protocolos de red. Impacket se centra en proporcionar acceso programático de bajo nivel a los paquetes y, para algunos protocolos (por ejemplo, SMB1-3 y MSRPC), la implementación del protocolo en sí.

#### Instalación

Ejecutar los siguientes comandos:

~~~
sudo git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket
sudo pip3 install -r /opt/impacket/requirements.txt
sudo cd /opt/impacket/
sudo pip3 install .
sudo python3 setup.py install
~~~

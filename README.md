# Proyecto DNS con Vagrant

Este proyecto configura un servidor DNS utilizando `bind9` en dos máquinas virtuales gestionadas por Vagrant.
A continuación se detallan los archivos clave que componen esta configuración.

## Estructura de Archivos

### Vagrantfile
Este archivo es la configuración principal para Vagrant. Define dos máquinas virtuales:

- **venus**:
  - Box: `debian/bookworm64`
  - IP: `192.168.57.102`
  - Hostname: `venus.sistema.test`
  - Provisión: Instala `bind9` y copia archivos de configuración.

- **tierra**:
  - Box: `debian/bookworm64`
  - IP: `192.168.57.103`
  - Hostname: `tierra.sistema.test`
  - Provisión: Instala `bind9` y copia archivos de configuración.

### named

- `OPTIONS="-u bind -4"`: Solo utiliza IPv4.

### named.conf.options
Archivo que contiene opciones de configuración:

- **acl "confiable"**:
    - Añadimos los servidores que permitirán consultas recursivas.
- **options**: 
    - Habilitamos el dnssec-validation `dnssec-validation yes;`.
    - añadimos los servidores del acl a los campos necesarios para permitir consultas recursivas. 
    - tenemos en cuenta el tiempo de cache en respuesta negativa a 2h: `max-ncache-ttl 7200;`.
    - reenviamos consultas no deseadas al OpenDNS: `forwarders { 208.67.222.222; };`.

### named.conf.local
Archivo que define las zonas DNS:

- **zone "sistema.test"**: Define la zona directa con su archivo de base de datos
                            correspondiente y permitimos la transferencia del esclavo.
- **zone "57.168.192.in-addr.arpa"**: Define la zona inversa con su archivo de base de datos
                            correspondiente y permitimos la transferencia del esclavo.

### db.sistema.test
Archivo de base de datos para la zona directa `sistema.test`:

- Contiene registros NS(name server), MX(mail exchange), CNAME (canonical name) 
y A(address) para los distintos dominios en la red.

### db.192.168.57
Archivo de base de datos para la zona inversa `192.168.57.0/24`:

- Contiene registros NS(name server) y PTR(pointer record) que permiten la resolución
inversa de direcciones IP a nombres de dominio.

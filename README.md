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

### tierra.named.conf.local
Archivo que define las zonas DNS:

- **zone "sistema.test"**: Define la zona directa con su archivo de base de datos
                            correspondiente y permitimos la transferencia del esclavo.
- **zone "57.168.192.in-addr.arpa"**: Define la zona inversa con su archivo de base de datos
                            correspondiente y permitimos la transferencia del esclavo.
### venus.tierra.named.conf.local
Archivo que define las zonas DNS:

- **zone "sistema.test"**: Define la zona directa con su archivo de base de datos
                            correspondiente y permitimos la transferencia del maestro.
- **zone "57.168.192.in-addr.arpa"**: Define la zona inversa con su archivo de base de datos
                            correspondiente y permitimos la transferencia del maestro.


### db.sistema.test
Archivo de base de datos para la zona directa `sistema.test`:

- Contiene registros NS(name server), MX(mail exchange), CNAME (canonical name) 
y A(address) para los distintos dominios en la red.

### db.192.168.57
Archivo de base de datos para la zona inversa `192.168.57.0/24`:

- Contiene registros NS(name server) y PTR(pointer record) que permiten la resolución
inversa de direcciones IP a nombres de dominio.


## Pruebas Realizadas
  A la hora de realizar las pruebas he usado el fichero test.sh y nslookupy dig por mi cuenta.

  ### Comprobacion de tipo A
  Las pruebas las he realizado tanto con nslookup y dig
  - `dig @192.168.57.102 mercurio.sistema.test A`
  - `nslookup mercurio.sistema.test 192.168.57.102`
  y asi con todas las ips

  ### Comprobacion de la resolución inversa de IP
  Las pruebas las he realizado tanto con nslookup y dig
  - `dig @192.168.57.102 -x 192.168.57.102`
  - `nslookup 192.168.57.103 192.168.57.102`
  y asi con todas las ips

  ### Comprobación de la resolución de los alias
  Las pruebas las he realizado tanto con nslookup y dig
  - `dig @192.168.57.102 ns1.sistema.test`
  - `nslookup ns1.sistema.test 192.168.57.102`
  y asi con todas las ips

  ### Comprobamos los nombres de servidor
  obtenemos el NS con dig y nslookup 
  - `dig @192.168.57.102 sistema.test NS` 
  - `nslookup -type=NS sistema.test 192.168.57.102`

  ### Comprobamos el MX 
  Obtenemos el mail exchanger con dig y nslookup
  - `dig @192.168.57.102 sistema.test MX`
  - `nslookup -type=MX sistema.test 192.168.57.102`
  
  ### Consulta del registro AXFR
  Realizamos esta consulta desde el dig solo nos dejaria hacerlo desde el slave
  - `dig @192.168.57.103 sistema.test AXFR`
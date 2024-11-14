# P6.Configuracion_cliente-servidor_DNS_AdrianRodriguezSebastian

**Engade ó docker-compose do DNS outro servicio (container) que faga a función de cliente.Usa unha imaxe alpine**

A continuacion voy poñer o archivo .yaml que esta no repositorio pero con comentarios al lado de cada linea de codigo para explicar un pouco que fai cada parametro e tamen porque puxe cada un deles:

  services:
    asir_bind9:
      container_name: asir_bind9          # Nome do contedor que executará o servizo DNS
     
      image: ubuntu/bind9  # Imaxe base do contedor para configurar o servidor DNS
  
      platform: linux/amd64  # Plataforma da imaxe (asegúranos de que sexa compatible co host)
      ports:
        - 54:53  # Mapeo de portas: expón o porto 53 do contedor no porto 54 do host
      networks:
        bind9_subnet:
          ipv4_address: 172.28.5.1  # Atribuímos a IP fixa 172.28.5.1 ao contedor dentro da subrede
      volumes:
        - ./conf:/etc/bind/  # Mapeamos o directorio local `./conf` ao directorio de configuración do contedor
        - ./zonas:/var/lib/bind/  # Mapeamos o directorio local `./zonas` ao directorio de zonas do contedor

    cliente:
      container_name: cliente  # Nome do contedor que actuará como cliente DNS
      image: alpine  # Imaxe base lixeira de Alpine Linux
      tty: true  # Habilitamos un terminal interactivo para o contedor
      stdin_open: true  # Permite que o contedor reciba entrada estándar (para interacción)
      dns:
        - 172.28.5.1  # O cliente usará o servidor DNS do contedor `asir_bind9`
      networks:
        bind9_subnet:
          ipv4_address: 172.28.5.2  # Atribuímos a IP fixa 172.28.5.2 ao cliente dentro da mesma subrede

    networks:
      bind9_subnet:
          driver: bridge  # Usamos un driver de rede tipo 'bridge' para permitir comunicación entre contedores
          ipam:
            config:
            - subnet: "172.28.0.0/16"  # Define o rango da subrede que abrangue todas as IPs da rede (172.28.0.0 a 172.28.255.255)
              ip_range: "172.28.5.0/24"  # Define un rango de IPs para asignar aos contedores (172.28.5.0 a 172.28.5.255)
              gateway: "172.28.5.254"  # Atribuímos unha IP para a porta de enlace (a que os contedores usaran para saír ao exterior)


Este ficheiro configura un servidor DNS (asir_bind9) accesible desde un cliente (cliente) na mesma subrede.
Para lanzar este .yml debemos facer docker compose up -d no directorio do yaml

**Instala, se é preciso, os paquetes que precises para usar no cliente os comando de rede do seguinte enlace.Comproba o seu uso.Fai a instalación de dig se é preciso.**

Para instalar o necesario no cliente de alpine primeiro debemos facer un docker exec -it cliente /bin/sh para entrar no sh do cliente.
Unha vez no cliente debemos facer un apk update && apk add bind-tools para asi poder instalar o dig e as demais ferreamentas (Usamos apk xa que e mediante sh).

Logo xa podemos facer diferentes consultas con dig como a de dig @172.28.5.1 test.asircastelao.int e todas as demais que aparecen no filleiro zonas.

**Configura o cliente para que o seu DNS sexa o otro container, modificando el resolv.conf ou usando o fichero docker-compose.yml (preferible). Compróbao con 'dig'.**

Para configura o cliente realmente non temos que facer nada xa que a ip do contenedor dns que no noso caso e un bind9 de ubuntu é 172.28.5.1 e a ip que ten por defecto no yaml o sistema do cliente alpine para responder as peticions dns e a 172.28.5.1.

Agora alguns exemplos do dig, poden ser os seguintes (non fai falta o @172.28.5.1 xa que o yaml o especifica, pero para que se vea dunha manera mais visual, prefiro escribilo):


    dig @172.28.5.1 test.asircastelao.int

    ; <<>> DiG 9.18.27 <<>> @172.28.5.1 test.asircastelao.int
    ; (1 server found)
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6517
    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
  
    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: cf315010c329fdd001000000672a76056a27fcbd01d56c52 (good)
    ;; QUESTION SECTION:
    ;test.asircastelao.int.		IN	A

    ;; ANSWER SECTION:
    test.asircastelao.int.	38400	IN	A	172.28.5.4

    ;; Query time: 0 msec
    ;; SERVER: 172.28.5.1#53(172.28.5.1) (UDP)
    ;; WHEN: Tue Nov 05 19:46:13 UTC 2024
    ;; MSG SIZE  rcvd: 94

Outra consulta pode ser a seguinte:

    / # dig @172.28.5.1 ns.asircastelao.int

    ; <<>> DiG 9.18.27 <<>> @172.28.5.1 ns.asircastelao.int
    ; (1 server found)
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28538
    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
  
    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: ce05af5f1e81aef101000000672a7631b3be60910c73b95c (good)
    ;; QUESTION SECTION:
    ;ns.asircastelao.int.		IN	A
  
    ;; ANSWER SECTION:
    ns.asircastelao.int.	38400	IN	A	172.28.5.1
  
    ;; Query time: 0 msec
    ;; SERVER: 172.28.5.1#53(172.28.5.1) (UDP)
    ;; WHEN: Tue Nov 05 19:46:57 UTC 2024
    ;; MSG SIZE  rcvd: 92


E como podemos observar, todo vai acorde co ficheiro zonas que configuramos anteriormente.

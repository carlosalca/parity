# Uso de Parity Ethereum node

Actualmente Parity se encuentra dockerizado y listo para desplegarlo, y sus imágenes se pueden encontrar en [1]. La información completa sobre Parity se puede encontrar en su wiki [2].

Aspectos interesantes de Parity Ethereum:

- Lo interesante de Parity Ethereum es utilizarlo vía su JSON-RPC API montada sobre un JSON-RPC HTTP Server en el puerto 8545 (el puerto se puede cambiar fácilmente). Para ello abrá que tener en cuenta el puerto a la hora de desplegar el contenedor. También será necesario indicar la interfaz de red a través de la cual se reciben las peticiones y los hosts que van a realizar dichas peticiones. Por defecto la interfaz de red está puesta en local y los hosts a ninguno permitido.
 ```bash
 --jsonrpc-interface=[IP]
 --jsonrpc-hosts=[HOSTS]
 ```
- Se puede conectar a una red de prueba de Ethereum como puede ser Kovan de manera simple, solamente cambiando la configuración.
 ```bash
 $ parity --chain kovan
 ```
- Se puede configurar mediante un archivo de configuración o mediante su CLI, si hay discordancia entre ambos tiene preferencia la configuración realizada en la CLI.
- Se puede generar un archivo de configuración TOML fácilmente con su [generador de archivos de configuración](https://paritytech.github.io/parity-config-generator/)
- Se puede reducir el ancho de banda que consume el nodo configurando que solo permita IP públicas y se desactive el servicio de discovery (solo desactiva el tráfico UDP, seguiría funcionando sin problemas). Incluso se puede limitar el número máximo de peers al que se conecte, siendo 10 el mínimo para garantizar una buena conexión, y también el número de peers que se conectan al nodo. Configuraciones: 
 ```bash 
 --allow-ips=public 
 --no-discovery 
 --max-pending-peers 
 --max-peers
 ```
- Se puede usar en modo de cliente ligero de forma que solamente se sincroniza lo mínimo necesario y se solicitan solo los datos que hagan falta *on-demand* a la red. De esta forma no se descarga toda la blockchain. Configuración:
 ```bash
 --light
 ```


> JSON-RPC API port: 8545

## Desplegar el contenedor

Para desplegar un contenedor de Parity Ethereum con Docker primero hay que descargar la imagen del repositorio de Docker Hub [1]:
```bash
$ docker pull parity/parity:tag
```
A continuación sería conveniente generar una configuración deseada, aunque sea mínimo poner el cliente en modo ligero si es lo que se quiere ya que de lo contrario, al iniciarlo, descargaría todo el ledger de la blockchain. Para ello lo primero es generar con el archivo de configuración TOML con el [generador de archivos de configuración](https://paritytech.github.io/parity-config-generator/). Un ejemplo con directorio personalizado del archivo de configuración y demás, cliente en modo ligero, peers limitados y JSON-RPC abierto sería:

````toml
# This config should be placed in following path:
# ~/.local/share/io.parity.ethereum/config.toml

[parity]
# Kovan Test Network
chain = "kovan"
# Blockchain and settings will be stored in ~/.ethereum/parity.
base_path = "~/.ethereum/parity"
# Parity databases will be stored in ~/.ethereum/parity/chains.
db_path = "~/.ethereum/parity/chains"
# Your encrypted private keys will be stored in ~/.ethereum/parity/keys.
keys_path = "~/.ethereum/parity/keys"
# Experimental: run in light client mode. Light clients synchronize a bare minimum of data and fetch necessary data on-demand from the network. Much lower in storage, potentially higher in bandwidth. Has no effect with subcommands.
light = true

[network]
# Parity will try to maintain connection to at least 10 peers.
min_peers = 10
# Parity will maintain at most 10 peers.
max_peers = 10
# Parity will allow up to 32 pending connections.
max_pending_peers = 32

[rpc]
# JSON-RPC will be listening for connections on IP all.
interface = "all"
# Allow connections only using specified addresses.
hosts = ["all"]
````

Tras esto ejecutamos el siguiente comando:
````bash
docker run -ti -v ~/ethereum/parity/:/home/parity/.local/share/io.parity.ethereum/ -p 8545:8545 parity/parity:v2.5.5-stable --config /home/parity/.local/share/io.parity.ethereum/config.toml
````

Otro comaando:

````bash
$ docker run -ti -p 8545:8545 -p 8546:8546 -p 30303:30303 -p 30303:30303/udp -v ~/ethereum/parity/:/home/parity/.local/share/io.parity.ethereum/   parity/parity:v2.5.5-stable --config /home/parity/.local/share/io.parity.ethereum/config.toml
````

 

## Referencias

[1] "parity/parity - Docker Hub", _Docker Hub_, 2019. [Online]. Available: https://hub.docker.com/r/parity/parity. [Accessed: 18- Jul- 2019].

[2] "Parity Documentation - Parity Ethereum", _wiki.parity.io_, 2019. [Online]. Available: https://wiki.parity.io/Parity-Ethereum. [Accessed: 18- Jul- 2019].

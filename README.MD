
<img src="https://www.vultr.com/favicon/favicon-32x32.png?v=1"> [vultr.com](https://www.vultr.com/?ref=8956372-8H)

:point_up_2: :point_up_2: :point_up_2: :point_up_2: Servidor **bom** e *barato*.

# docker-mysql
Teste de deploy docker em produção

## Implantação com Docker e MySQL

### 1 - Docker compose

*Antes verifique a versão atual [docker github releases](https://github.com/docker/compose/releases), torque abaixo se necessário* **1.27.4**

```sudo apt-get update```
```sudo apt-get upgrade docker-ce```

```
sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

**Permissões**

```sudo chmod +x /usr/local/bin/docker-compose```

***Verificar versão***
```docker-compose --version```

### 2 - Iniciar

start: ```bash ./start.sh```

stop: ```bash ./stop.sh```

build: ```docker-compose up -d --build```


### 3 - Configuração de rede interna
 > ref: [Ubuntu 17.10 à 20.04](https://www.vultr.com/docs/how-to-configure-a-private-network-on-ubuntu#Ubuntu_17_10_through_20_04)

 ```sudo nano /etc/netplan/10-ens7.yaml```
 
**Exemplo:**

```yaml
 network:
  version: 2
  renderer: networkd
  ethernets:
    ens7:
      mtu: 1450
      dhcp4: no
      addresses: [10.39.96.4/16]

```

```netplan apply```

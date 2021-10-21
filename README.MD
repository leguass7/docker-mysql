
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

## 4 - Recursos limitados
Por padrão, um contêiner não tem restrições de recursos e pode usar tanto de um determinado recurso quanto o planejador de kernel do host permitir. É importante não permitir que um contêiner em execução consuma muita memória da máquina host. Em hosts Linux, se o kernel detecta que não há memória suficiente para executar funções importantes do sistema, ele lança um **OOME**, ou **Out Of Memory Exception**, e começa a matar processos para liberar memória, incluindo **Docker** e outros aplicativos importantes.

**Faça caso** `docker info` resulte em `No swap limit support`
1) Edite o arquivo `/etc/default/grub`. Adicione ou edite a `GRUB_CMDLINE_LINUX` linha para adicionar os dois pares de valores-chave a seguir: 
   - `GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"`

2) Atualize e reinicie o sistema
   - ```$ sudo update-grub```
   - ```$ sudo reboot```


3) Docker compose
   - veja: [docker-compose.yml](docker-compose.yml)

**Exemplo:**

```yml
version: '3.7'

services:
  mysql8:
    # ...
    deploy:
      resources:
        limits:
          cpus: '0.92'
          memory: 1128M  #server com 2Gb de RAM
    #...
```

## Verificando memória disponível (Ubuntu)
Formas de verificando memória disponível para configuração de limites do docker:
  - a) ```$ free -m```
  - b) ```$ htop``` (```$ sudo apt install htop```)
  - c) ```$ top```
  - d) ```$ vmstat -s```


## Restaurar backup (mysqldump files)
**Cenário:**
Volume do docker compartilhando diretório com arquivos dump (*.sql)
```$ ~/provision```

ou copiando arquivos para volume compartilhado
```$ mv ~/provision/*.sql ~/docker-mysql/provision/```

importando arquivos:
```$ cd ~/docker-mysql/provision/```
```$ find . -name '*.sql' | awk '{ print "source",$0 }' | mysql --batch -u root -p <database_name>```

## SSH - subindo arquivos dump

```scp remote_user@remote_host:~/provision/ /path/to/local/provision```

---
## Otimizar restore de tabela com mais de 5 milhões de linhas
Quando se trata de carregamento em massa de tabelas InnoDB, uma das configurações mais freqüentemente ignoradas para desabilitar é [innodb_doublewrite](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_doublewrite).

Cada vez que uma página de 16K é gravada em uma tabela InnoDB ou log de transações, a página é gravada primeiro no buffer de gravação dupla. Tudo que você precisa fazer é desativá-lo, carregar seus dados e reativá-lo.

1) Desativar desligamento rápido
```sql
SET GLOBAL innodb_fast_shutdown=0
```

2) Reinicie o mysql sem o buffer de gravação dupla

```$ service mysql restart --innodb-doublewrite=0```

3) Carregue os dados InnoDB

```$ mysql -u user -p database_name < dump-table-file.sql```

4) Reinicie o mysql

```$ service mysql restart```

*Antes de carregar os dados também pode desligar os indices da tabela caso o arquivo de dump não contenha instruções para isso:*

```sql

ALTER TABLE `my_table` DISABLE KEYS;
-- Your insert statement
ALTER TABLE `my_table` ENABLE KEYS;

```

---

## Referências
[docker config](https://docs.docker.com/config/containers/resource_constraints/) [docker engine](https://docs.docker.com/engine/install/linux-postinstall/#your-kernel-does-not-support-cgroup-swap-limit-capabilities) [memory ubuntu](https://vitux.com/5-ways-to-check-available-memory-in-ubuntu/)

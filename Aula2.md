# Aula 2 - Segurança em redes

## Fases de um ataque

A execução de um ataque segue o seguinte ciclo:
1. Aquisição de informações
2. Infiltração
3. Aprendizagem
4. Propagação
5. Aprendizagem (novamente)
6. Agregação
7. Exfiltração

## *Jamming*

O *jamming* é uma técnica de ataque que consiste em interferir na comunicação entre dispositivos.

### Arquitetura AAA

A arquitetura AAA é uma arquitetura de controlo de acesso à rede que consiste em três componentes:
 - **Autenticação**: verifica a identidade do utilizador
 - **Autorização**: verifica o que o utilizador pode fazer
 - **Auditoria**: regista as ações do utilizador

## 802.1x

O 802.1x é um protocolo de controlo de acesso à rede (NAC).

 - 802.1x-2001 e 802.1x-2004: permite apenas autenticação.
 - 802.1x-2010: adiciona cifragem opcional no segmento LAN

## Extensible Authentication Protocol (EAP)

O EAP é um protocolo de autenticação que permite a utilização de diferentes métodos de autenticação em situações onde o IP não se encontra disponível.


# Prática 2 - Configuração de um servidor RADIUS

Nesta prática é configurado um servidor RADIUS para autenticação de utilizadores.

Na infraestrutura temos o nosso servidor RADIUS, que se encontra ligado a um *ethernet switch* que irá servir de autenticador, e o mesmo está ligado a um cliente que irá ser autenticado.

Primeiramente vamos instalar o *FreeRADIUS* no nosso servidor:
```bash
$ sudo apt-get install freeradius
```

Depois de instalado, vamos configurar o *FreeRADIUS* para que o mesmo possa ser utilizado para autenticação. 

Para isso, vamos editar o ficheiro `/etc/freeradius/3.0/clients.conf` e adicionar o seguinte conteúdo:
```conf
client 10.0.0.1 {
    secret = radiuskey
}
```
Isto irá definir um segredo partilhado entre o servidor RADIUS e o autenticador.

Depois disso vamos definir um utilizador no ficheiro `/etc/freeradius/3.0/users`:
```conf
"labredes" Cleartext-Password := "labcom"
```
Depois de configurado, vamos reiniciar o serviço.

Depois terá de ativar a funcionalidade do *RADIUS* autenticar pedidos. Para isso terá de editar o ficheiro `/etc/freeradius/3.0/radiusd.conf` e modificar a linha `auth` para:
```conf
auth = yes
```

Seguidamente vamos adicionar os endereços de IP no autenticador:
```cli
ESW1(config)# ip routing
ESW1(config)# int vlan 1
ESW1(config-if)# ip address 10.1.0.1 255.255.255.255
ESW1(config-if)# no shutdown
ESW1(config-if)# int fa 0/0
ESW1(config-if)# ip address 10.0.0.1 255.255.255.255
ESW1(config-if)# no shutdown
```

Seguidamente será configurar o autenticador. Isto será feito habilitando localmente o *802.1x* na porta ligada ao cliente e habilitar a autenticação RADIUS.
```cli
ESW1(config)# aaa new-model
ESW1(config)# aaa authentication dot1x default group radius
ESW1(config)# dot1x system-auth-control
ESW1(config)# radius-server host 10.0.0.100 auth-port 1812 key radiuskey
ESW1(config)# interface FastEthernet1/0
ESW1(config-if)# dot1x port-control auto
```

Depois irá adicinar as credenciais do utilizador `labredes/labcom` no cliente e o respetivo ip (10.1.0.100) e *gateway* (10.1.0.1).


E finalmente para ligar o servidor apenas terá de executar o seguinte comando:
```bash	
$ sudo systemctl start freeradius
```
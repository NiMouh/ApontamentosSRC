# Aula 3 - Sistemas de segurança em redes

## Firewall

Uma *firewall* é um sistema de segurança que fornece um **ponto de defesa único** entre as redes e protege uma rede, através de uma **política de controlo** entre duas ou mais redes.

Com uma *firewall* é possível controlar:
- Acesso;
- Fluxo;
- Conteúdo.

Serviços que podem ser usado numa *firewall*:
- NAT (*Network Address Translation*);
- *Proxy* (Redirecionamento de tráfego);
- VPN (*Site-to-Site* e *Remote Access*);
- *Packet Filtering*;

Tipos de *firewall*:
- *Network-level*: opera na camada de rede;
- *Circuit-level*: opera na camada de transporte;
- *Application-level*: opera na camada de transporte para cima.
- *Stateful*: opera na camada de rede e na camada de transporte.
- *Host level*: opera na camada de aplicação.

Podem dividir-se em dois tipos:
- *Stateful*: controla o tráfego com base em fluxos de tráfego/sessões;
- *Stateless*: análise todos os pacotes individualmente e aplica regras de filtragem.

As *firewalls* podem ser divididas em **multiplas zonas**, com diferentes níveis de segurança, podendo definir **regras de origem e destino** (um exemplo disso é a DMZ, ou *Demilitarized Zone*, que é uma rede de perímetro entre a rede interna e a rede externa).

As *firewalls* podem ser implementadas **antes** ou **depois** do *router*. 

- Se for implementada **antes**, a *firewall* pode ser usada para proteger o *router* de possíveis ataques, porém isso pode mais complexo e diminuir a disponibilidade da rede.
- Se for implementada **depois**, a *firewall* pode ser mais fácil de configurar e manter, porém o *router* pode ser mais vulnerável a ataques.

## IPS (Intrusion Prevention System)

Um IPS é um sistema de segurança que monitoriza o tráfego de dados e bloqueia pacotes que correspondam a padrões de ataque conhecidos.

## IDS (Intrusion Detection System)

Um IDS é um sistema de segurança que monitoriza o tráfego de dados e alerta o administrador de rede quando deteta padrões de ataque conhecidos. Isto é feito através de uma *deep-packet inspection* (DPI) e de uma *shallow-packet inspection* (SPI).


# Prática 3 - Configuração de uma firewall (VyOS)

Neste guia é simulado o controlo de trafego entre uma rede interna, uma DMZ (rede de perímetro) e uma rede externa.

## Configuração PC1 - Rede interna

```console
$ ip 10.1.1.100/24 10.1.1.1
```

## Configuração PC2 - Rede Externa

```console
$ ip 200.1.1.100/24 200.1.1.1
```

## Configuração PC3 e PC4 - DMZ

Para o PC3:
```console
$ ip 192.1.1.40/24 192.1.1.1
```

Para o PC4:
```console
$ ip 192.1.1.140/24 192.1.1.1
```

## Configuração do VyOS

Para atribuir um endereço IP à interfaces ligadas às redes:
```console
# set interfaces ethernet eth0 address 200.1.1.1/24
# set interfaces ethernet eth1 address 192.1.1.1/24
# set interfaces ethernet eth2 address 10.1.1.1/24
# commit
# save
```

**Nota**: Na *firewall* o simbolo `$` no prompt indica que está a usar o **modo de utilizador**, enquanto que o simbolo `#` indica que está a usar o **modo de administrador**.

Para configurar o *NAT*:
```console
# set nat source rule 100 outbound-interface eth0
# set nat source rule 100 source address 10.1.1.0/24
# set nat source rule 100 translation address 192.1.0.1-192.1.0.10
# commit
# save
```
Este comando permite que o tráfego da rede interna seja traduzido para um endereço IP da DMZ.

Para testar, basca usar o comando `$ show nat source rules`.

*Network security zones* permitem definir uma rede ou um conjunto de redes que estão protegidas por uma *firewall*.

Aqui vamos definir quais são as zonas existentes e qual a sua interface correspondente:
```console
# set zone-policy zone INSIDE description "Inside (Internal Network)"
# set zone-policy zone INSIDE interface eth2
# set zone-policy zone DMZ description "DMZ (Server Farm)"
# set zone-policy zone DMZ interface eth1
# set zone-policy zone OUTSIDE description "Outside (Internet)"
# set zone-policy zone OUTSIDE interface eth0
# commit
# save
```

Para verificar as zonas criadas, basta usar o comando `$ show zone-policy`.


*Zone-Policies* permitem definir o que entra/sai entre as zonas.

Aqui vamos definir as regras de tráfego entre as zonas:
```console
# set firewall name FROM-INSIDE-TO-OUTSIDE rule 10 description "Accept ICMP Echo Request"
# set firewall name FROM-INSIDE-TO-OUTSIDE rule 10 action accept
# set firewall name FROM-INSIDE-TO-OUTSIDE rule 10 protocol icmp
# set firewall name FROM-INSIDE-TO-OUTSIDE rule 10 icmp type 8
# set firewall name TO-INSIDE rule 10 description "Accept Established-Related Connections"
# set firewall name TO-INSIDE rule 10 action accept
# set firewall name TO-INSIDE rule 10 state established enable
# set firewall name TO-INSIDE rule 10 state related enable
# set zone-policy zone INSIDE from OUTSIDE firewall name TO-INSIDE
# set zone-policy zone OUTSIDE from INSIDE firewall name FROM-INSIDE-TO-OUTSIDE
# commit
# save
```

Nós aqui podemos verificar que criamos uma regra com duas ações:
- Para sair: é permitido o tráfego ICMP;
- Para entrar: é permitido o tráfego que foi iniciado a partir do interior da rede.

Para verificar as regras criadas, basta usar o comando `$ show firewall`.

(POR ACABAR A PARTIR DA ALINEA 6)

Aqui vamos definir as regras que permitem os dispositivos da rede interna aceder aos servidores DMZ (ICMP):
```console
 # set firewall name FROM-INSIDE-TO-DMZ rule 10 description "Accept ICMP Echo Request"
 # set firewall name FROM-INSIDE-TO-DMZ rule 10 action accept
 # set firewall name FROM-INSIDE-TO-DMZ rule 10 protocol icmp
 # set firewall name FROM-INSIDE-TO-DMZ rule 10 icmp type 8
 # set firewall name FROM-INSIDE-TO-DMZ rule 10 destination address 192.1.1.0/24
 # set zone-policy zone INSIDE from DMZ firewall name TO-INSIDE
 # set zone-policy zone DMZ from INSIDE firewall name FROM-INSIDE-TO-DMZ
 # commit
# save
```

Ou seja, vamos conseguir pingar os servidores da DMZ a partir da rede interna, ao contrário apenas é enviado tráfego que foi iniciado a partir da rede interna.

Aqui vamos definir as regras que permitem os dispositivos da rede externa aceder aos servidores DMZ (ICMP):
```console
 # set firewall name FROM-OUTSIDE-TO-DMZ rule 10 description "Accept ICMP Echo Request"
 # set firewall name FROM-OUTSIDE-TO-DMZ rule 10 action accept
 # set firewall name FROM-OUTSIDE-TO-DMZ rule 10 protocol icmp
 # set firewall name FROM-OUTSIDE-TO-DMZ rule 10 icmp type 8
 # set firewall name FROM-OUTSIDE-TO-DMZ rule 10 destination address 192.1.1.40
 # set firewall name FROM-DMZ-TO-OUTSIDE rule 10 description "Accept Established-Related Connections"
 # set firewall name FROM-DMZ-TO-OUTSIDE rule 10 action accept
 # set firewall name FROM-DMZ-TO-OUTSIDE rule 10 state established enable
 # set firewall name FROM-DMZ-TO-OUTSIDE rule 10 state related enable
 # set zone-policy zone OUTSIDE from DMZ firewall name FROM-DMZ-TO-OUTSIDE
 # set zone-policy zone DMZ from OUTSIDE firewall name FROM-OUTSIDE-TO-DMZ
 # commit
# save
```

Ou seja, vamos conseguir pingar o servidor `192.1.1.40` a partir da rede externa, ao contrário apenas é enviado tráfego que foi iniciado a partir da rede externa.

De seguida vamos definir também que os dispositivos da rede externa podem mandar pacotes UDP (apenas pela porta `8080`) para o servidor `192.1.1.140`
```console
 # set firewall name FROM-OUTSIDE-TO-DMZ rule 12 description "Accept UDP-8080"
 # set firewall name FROM-OUTSIDE-TO-DMZ rule 12 action accept
 # set firewall name FROM-OUTSIDE-TO-DMZ rule 12 protocol udp
 # set firewall name FROM-OUTSIDE-TO-DMZ rule 12 destination address 192.1.1.140
 # set firewall name FROM-OUTSIDE-TO-DMZ rule 12 destination port 8080
 # commit
# save
```

# Aula 4 - Firewalls (continuação)

## Disponibilidade em Firewalls

Existem dois cenários de disponibilidade em *firewalls*:
- **Active-Backup**: um *firewall* está ativo e o outro está em *standby*. 
- **Active-Active**: ambos os *firewalls* estão ativos.

## *Load Balancing*

O *load balancing* é uma técnica que permite distribuir o tráfego por diferentes servidores, de forma a evitar sobrecargas e a melhorar a disponibilidade.

Existem diferentes **tipos de *load balancing***:
- **IP Hash**: distribui o tráfego com base no endereço IP do cliente. Não necessita de sincronização de estados. O destino é escolhido com base no *hash* do endereço IP do cliente.
- **Round Robin**: distribui o tráfego por ordem. Não necessita de sincronização de estados.
- **Least Connections**: distribui o tráfego pelo servidor com menos ligações. Necessita de sincronização de estados.
- **"Smart"**: distribui o tráfego com base em métricas como a carga do servidor, a latência, etc. 

## Tipos de ACL's

As *Access Control Lists* (ACL's) são regras que definem o que é permitido e o que é bloqueado num *firewall*.

Podem ser:
- **Standard**: baseadas no endereço IP de origem.
- **Extended**: baseadas no endereço IP de origem e destino, e no protocolo.
- **Named**: regras que podem ser reutilizadas.
- **Reflexive**: regras que são criadas automaticamente para permitir o tráfego de resposta.
- **Context-based**: regras que são aplicadas com base no contexto.

# Prática 4 - *High Availability* em *Firewalls*

Nesta prática é simulado o controlo de tráfego entre uma rede interna e uma rede externa, com duas *firewalls*.

Primeiro vamos carregar a configuração default do VyoS:
```bash
sudo cp /opt/vyatta/etc/config.boot.default /config/config.boot
reboot
```

Depois de dar os endereços IP às interfaces, vamos adicionar nos routers (interno e externo) as rotas para a rede um do outro.

Para o router interno, onde qualquer tráfego que não seja para a rede interna será redirecionado para a firewall FW2:
```cli
ip route 0.0.0.0 10.1.1.2
```

Para o router externo, onde qualquer tráfego que seja para a rede interna (endereço NAT), será redirecionado para a firewall FW1:
```cli
ip route 192.1.0.0 255.255.254.0 200.1.1.1
```

Depois de configurar os IP's nas firewall FW1 e FW2, vamos configurar o *NAT/PAT* na firewall FW1:
```cli
# set nat source rule 10 outbound-interface eth0
# set nat source rule 10 source address 10.0.0.0/8
# set nat source rule 10 translation address 192.1.0.1-192.1.0.10
```

Ou seja, qualquer tráfego que saia da rede interna, será traduzido para um endereço da rede 192.1.0.0.

Para configurar o *high-availability* nas firewalls, vamos configurar o *VRRP* (Virtual Router Redundancy Protocol) de modo a que as duas firewalls tenham o mesmo endereço IP virtual:
```cli
# set high-availability vrrp group FWCluster vrid 10
# set high-availability vrrp group FWCluster interface eth5
# set high-availability vrrp group FWCluster virtual-address 192.168.100.1/24
# set high-availability vrrp sync-group FWCluster member FWCluster
# set high-availability vrrp group FWCluster rfc3768-compatibility
```

De seguida vamos configurar o *conntack sync* para sincronizar as tabelas de estado entre as duas firewalls, para os protocolos `TCP`, `UDP`, e `ICMP`:
```cli
# set service conntrack-sync accept-protocol 'tcp,udp,icmp'
 # set service conntrack-sync failover-mechanism vrrp sync-group FWCluster
 # set service conntrack-sync interface eth5
 # set service conntrack-sync mcast-group 225.0.0.50 // multicast address
 # set service conntrack-sync disable-external-cache
```

De seguida resta apenas criar uma regra de *firewall* para permitir o tráfego entre a rede interna e a rede externa, apenas através do protocolo `UDP`:
```cli
# set firewall name FROM-INSIDE-TO-OUTSIDE rule 10 action accept
# set firewall name FROM-INSIDE-TO-OUTSIDE rule 10 description "Allow UDP traffic from inside to outside"
# set firewall name FROM-INSIDE-TO-OUTSIDE rule 10 protocol udp
# set firewall name FROM-INSIDE-TO-OUTSIDE rule 10 destination port 5000-6000

# set firewall name FROM-OUSIDE-TO-INSIDE rule 10 action accept
# set firewall name FROM-OUSIDE-TO-INSIDE rule 10 description "Established-Related Connections"
# set firewall name FROM-OUSIDE-TO-INSIDE rule 10 state established enable
# set firewall name FROM-OUSIDE-TO-INSIDE rule 10 state related enable
```
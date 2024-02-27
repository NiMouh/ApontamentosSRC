# Aula 1 - Apanhado geral de redes

## Design de uma rede segura

Uma rede deve ser:
 - Modular: permitir crescimento e alterações (escalável);
 - Resiliente: tolerante a falhas (*up-time* perto dos 100%);
 - Flexível: permitir diferentes tipos de tráfego;

## Equipamentos de rede

Para a escolha dos equipamentos é necessário ter em conta:
 - Fabricante: fiabilidade, suporte, preço, etc.;
 - Modelo: funcionalidades, desempenho, etc.;
 - Capacidade: número de portas, velocidade, etc.;

### Switches

São equipamentos que ligam diferentes dispositivos numa rede local. A sua função é encaminhar pacotes de dados para o destino correto.

Permitem:
 - Conexão pela camada 2 OSI (ligação de dados);
 - Implementar VLANs;
 - *Spanning-tree based routing*

### Routers

São equipamentos cuja função é encaminhar dados entre diferentes redes (redes locais, como LANs, e redes externas, como a internet).

Permitem:
 - Conexão pela camada 3 OSI (rede);
 - *VPN Gateway*;
 - *Network Address Translation* (NAT);
 - *QoS* (Quality of Service);
 - Monitorização de tráfego.

### L3 Switches

São equipamentos que combinam as funcionalidades de um *switch* e de um *router*.


## Modelo de rede hierárquico

O modelo de rede hierárquico é uma forma de organizar uma rede de forma a facilitar a sua gestão e manutenção.

Nele temos três camadas:
 - **Core**: camada central, onde se concentra o tráfego de dados;
 - **Distribution**: camada intermédia, onde se faz a ligação entre a camada *core* e a camada *access*;
 - **Access**: camada onde se ligam os dispositivos finais.

## Classificação de redes

As redes de computadores podem ser classificadas em diferentes tipos, consoante a sua dimensão e alcance:
 - **WAN** (Wide Area Network): redes de grande dimensão, que ligam diferentes redes locais;
 - **MAN** (Metropolitan Area Network): redes de dimensão intermédia, que ligam diferentes redes locais numa área metropolitana;
 - **LAN** (Local Area Network): redes de pequena dimensão, que ligam dispositivos numa área geográfica limitada.
 - **PAN** (Personal Area Network): redes de muito pequena dimensão, que ligam dispositivos pessoais.

## Implementações a ter em conta

- *Daisy Chaining*: apesar de ser uma forma custo-eficiente de ligar dispositivos, pode trazer problemas de desempenho (ex: *Access-layer switch overload*) e de segurança.
- **Caminhos alternativos**: Evitar *single-path cores* pode trazer como vantages a redundância, a tolerância a falhas, balanceamento de carga e escalabilidade.
- **Triângulos de Redundância**: A redundância é uma forma de garantir a disponibilidade de uma rede em caso de falha de um dos seus componentes.

## *Routing*

O *routing* é o processo de encaminhamento de pacotes de dados entre diferentes redes. Existem diferentes tipos de *routing*:
 - **Static Routing**: rotas definidas manualmente;
 - **Dynamic Routing**: rotas definidas automaticamente, com base em algoritmos de encaminhamento.
 - **Policy-based Routing**: rotas definidas manualmente com base em políticas/regras de *routing*.

Estas rotas encontram-se definidas numa tabela de *routing*.

Existem dois tipos de **protocolos** de *routing*:
 - *Distance-vector*: é feito o melhor caminho baseado na informação enviada pelos vizinhos (ex: RIP, EIGRP);
 - *Link-state*: é feito o melhor caminho baseado na informação partilhada por todos os routers num estado *link*, descrevendo informação como a largura de banda, atraso, custo, de modo a construir um mapa completo da rede (ex: OSPF, IS-IS).

### Sistemas autónomos

É um conjunto de routers/redes com uma politica de *routing* comum. Cada sistema autónomo é identificado por um número único, o **ASN** (Autonomous System Number).

Existem dois tipos de sistemas autónomos:
 - **IBGP** (Internal BGP): *routing* dentro do mesmo sistema autónomo;
 - **EBGP** (External BGP): *routing* entre diferentes sistemas autónomos.

## Tipos de redes

Em termos de transporte, as redes podem ser:
 - **Trânsito/Transporte**: transportam tráfego de *hosts* para a internet e vice-versa;
 - **Stub**: transportam apenas tráfego para os seus *hosts*/rede local;
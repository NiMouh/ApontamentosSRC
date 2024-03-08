# Aula 3 - Sistemas de segurança em redes

## Firewall

Um *firewall* é um sistema de segurança que fornece um **ponto de defesa único** entre as redes e protege uma rede, através de uma **política de controlo** entre duas ou mais redes.

Com um *firewall* é possível controlar:
- Acesso;
- Fluxo;
- Conteúdo.

Serviços que podem ser usado num *firewall*:
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

Podem dividir-se em:
- *Stateful*: controla o tráfego com base em fluxos de tráfego/sessões;
- *Stateless*: análise todos os pacotes individualmente e aplica regras de filtragem.

## IPS (Intrusion Prevention System)

Um IPS é um sistema de segurança que monitoriza o tráfego de dados e bloqueia pacotes que correspondam a padrões de ataque conhecidos.

## IDS (Intrusion Detection System)

Um IDS é um sistema de segurança que monitoriza o tráfego de dados e alerta o administrador de rede quando deteta padrões de ataque conhecidos. Isto é feito através de uma *deep-packet inspection* (DPI) e de uma *shallow-packet inspection* (SPI).


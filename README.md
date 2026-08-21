# Projeto SIEM com Wazuh

## Visão geral

Este repositório documenta a implementação de uma infraestrutura de **Security Information and Event Management (SIEM)** baseada em **Wazuh**, criada para centralizar a recolha, análise e monitorização de eventos de segurança provenientes de diferentes sistemas de uma rede.

O projeto simula um ambiente próximo de um cenário empresarial, utilizando uma arquitetura distribuída com **Wazuh Manager, Wazuh Indexer, Wazuh Dashboard e Wazuh Agents**, além da integração de servidores Docker e Squid.

> **Autor:** Cleison Máquina  
> **Ano:** 2026  
> **Projeto:** SIEM / Monitorização de Segurança  
> **Website:** soc.cleisonmq.com

## Objetivos

A infraestrutura foi desenvolvida com os seguintes objetivos:

- Centralizar logs e eventos de segurança.
- Monitorizar endpoints e servidores.
- Detetar atividades potencialmente suspeitas.
- Gerar alertas de segurança automaticamente.
- Facilitar a investigação de incidentes.
- Monitorizar a integridade de ficheiros.
- Integrar diferentes fontes de logs.
- Criar uma base para futuras expansões do laboratório SOC.

## Arquitetura

```text
                         ┌─────────────────────┐
                         │       Agentes       │
                         │                     │
                         │ Docker / Squid /    │
                         │ outros servidores   │
                         └──────────┬──────────┘
                                    │
                                  Logs
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    Wazuh Manager    │
                         │                     │
                         │ Regras / Análise /  │
                         │ Correlação / Alertas│
                         └──────────┬──────────┘
                                    │
                                  Eventos
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    Wazuh Indexer    │
                         │                     │
                         │ Armazenamento /     │
                         │ Indexação / Pesquisa│
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Wazuh Dashboard   │
                         │                     │
                         │ Visualização / SOC  │
                         │ Investigação        │
                         └─────────────────────┘
```

## Componentes

### Wazuh Agent

Instalado nos sistemas monitorizados. É responsável por recolher informação dos endpoints e encaminhá-la para o Wazuh Manager.

Principais funções:

- Recolha de logs.
- Monitorização da integridade dos ficheiros.
- Inventário do sistema.
- Monitorização de atividades nos endpoints.
- Envio de eventos para o Manager.

### Wazuh Manager

É o componente central de processamento da solução.

Responsabilidades:

- Receber eventos dos agentes.
- Aplicar regras de deteção.
- Correlacionar eventos.
- Gerar alertas.
- Gerir os agentes.
- Executar mecanismos de resposta configurados.

### Wazuh Indexer

Responsável pelo armazenamento e indexação dos eventos.

Funções principais:

- Armazenamento de eventos.
- Indexação de logs.
- Pesquisa dos dados recolhidos.
- Disponibilização dos dados para análise e dashboards.

### Wazuh Dashboard

Interface gráfica utilizada para administração, monitorização e investigação.

Permite:

- Visualizar alertas.
- Consultar eventos.
- Gerir agentes.
- Investigar incidentes.
- Criar e consultar dashboards.
- Analisar o estado da infraestrutura.

## Infraestrutura do laboratório

### Servidor Wazuh Indexer

| Recurso | Configuração |
|---|---|
| CPU | 2 vCPU |
| RAM | 4 GB |
| Disco | 30 GB |
| Sistema operativo | CentOS Stream 10 |
| Função | Armazenamento e indexação |

### Servidor Wazuh Manager + Dashboard

| Recurso | Configuração |
|---|---|
| CPU | 2 vCPU |
| RAM | 4 GB |
| Disco | 30 GB |
| Sistema operativo | Ubuntu Server 22.04 LTS |
| Função | Processamento, alertas e interface |

## Segmentação de rede

A infraestrutura utiliza duas redes distintas.

### Rede NAT

Utilizada para:

- Agentes.
- Servidores monitorizados.
- Fontes externas de logs.
- Comunicação com redes externas.

### Rede Host-Only

Utilizada para os componentes centrais do SIEM:

- Wazuh Manager.
- Wazuh Indexer.
- Wazuh Dashboard.

Esta separação permite isolar os serviços críticos e controlar melhor as comunicações internas.

> Os endereços apresentados na documentação original são exemplos de redes privadas (`10.x.x.x/24` e `192.168.x.x/24`) e devem ser adaptados à infraestrutura onde o projeto for implementado.

## Hardening e segurança

Foram aplicadas medidas de hardening ao acesso SSH, incluindo:

- Utilização de chaves SSH ED25519.
- Desativação da autenticação por password.
- Alteração da porta SSH padrão.
- Bloqueio de passwords vazias.
- Gestão controlada dos acessos remotos.

Estas medidas reduzem a exposição do serviço SSH a ataques automatizados e tentativas de força bruta.

## Monitorização de Docker

Um servidor Docker foi integrado como endpoint monitorizado.

O objetivo é permitir:

- Recolha de logs dos containers.
- Monitorização da atividade do ambiente Docker.
- Centralização dos eventos no SIEM.
- Deteção de comportamentos potencialmente suspeitos.

## Integração com Squid

O projeto inclui também um servidor **Squid Proxy** como fonte de eventos.

A integração permite:

- Recolher logs de navegação.
- Monitorizar acessos HTTP/HTTPS.
- Centralizar eventos do proxy.
- Aumentar a visibilidade sobre a atividade da rede.
- Monitorizar alterações relevantes nos ficheiros de configuração através do Wazuh.

A pasta `servidor-squid/` contém documentação específica relativa à configuração do servidor Squid como agente Wazuh.

## Sistema de alertas

Foi configurado um mecanismo de notificações por email utilizando:

- Wazuh Manager.
- Postfix.
- Gmail SMTP.

Fluxo simplificado:

```text
Evento suspeito
      │
      ▼
Wazuh Manager
      │
      ▼
   Postfix
      │
      ▼
Email de alerta
```

O objetivo é permitir que eventos relevantes sejam comunicados rapidamente, reduzindo o tempo entre a deteção e a resposta.

## DNS

Foi configurado um registo DNS para facilitar o acesso ao Wazuh Dashboard.

Em vez de depender diretamente de endereços IP, a utilização de um nome DNS torna a administração e utilização da plataforma mais simples e preparada para futuras alterações de infraestrutura.

## Estrutura do repositório

```text
siem-wazuh-cleisonmq/
├── servidor-squid/
│   └── servidor_squid.md
├── README.md
├── resumo-projeto.md
└── .gitignore
```

O repositório é predominantemente **documental**: a infraestrutura é implementada nos servidores/laboratório e o GitHub funciona como base de documentação da arquitetura, configurações e decisões técnicas.

## Fluxo operacional

1. Os agentes recolhem eventos dos sistemas monitorizados.
2. Os eventos são enviados para o Wazuh Manager.
3. O Manager analisa os eventos através das regras configuradas.
4. Eventos relevantes originam alertas.
5. Os dados são indexados no Wazuh Indexer.
6. O Dashboard apresenta os eventos e alertas para análise.
7. Eventos críticos podem gerar notificações por email.
8. O analista utiliza os dados centralizados para investigação e resposta.

## Competências demonstradas

O projeto demonstra conhecimentos práticos em:

- SIEM e monitorização de segurança.
- Wazuh.
- Administração Linux.
- Gestão e análise de logs.
- Gestão de agentes.
- Hardening SSH.
- Segmentação de redes.
- Monitorização de Docker.
- Integração de Squid.
- Configuração SMTP/Postfix.
- DNS.
- Gestão de alertas.
- Conceitos de SOC e resposta a incidentes.

## Resultados

A infraestrutura permite centralizar eventos de segurança, monitorizar sistemas, gerar alertas automáticos e integrar diferentes fontes de informação numa única plataforma.

A arquitetura também foi concebida para permitir a adição futura de novos agentes, fontes de logs, regras de deteção e mecanismos de resposta.

## Possíveis evoluções

Como próximos passos, o projeto pode evoluir através de:

- Adição de regras Wazuh personalizadas.
- Mapeamento de deteções para **MITRE ATT&CK**.
- Integração de threat intelligence.
- Implementação de respostas automáticas.
- Monitorização de mais endpoints Windows e Linux.
- Integração de firewall, IDS/IPS e outros equipamentos de rede.
- Criação de dashboards específicos para SOC.
- Documentação de cenários de ataque e respetivas deteções.
- Automatização da instalação e configuração através de Ansible ou scripts.
- Implementação de backup e retenção de dados.
- Adição de testes de validação da infraestrutura.

## Referências

- [Repositório do projeto](https://github.com/cleisonmq/siem-wazuh-cleisonmq)
- [Documentação oficial do Wazuh](https://documentation.wazuh.com/)
- [Projeto Wazuh no GitHub](https://github.com/wazuh/wazuh)

## 📌 Conclusão

Este projeto apresenta uma implementação prática de uma infraestrutura **SIEM baseada em Wazuh**, com uma arquitetura distribuída e integração de diferentes fontes de eventos.

A combinação de **Wazuh Manager, Indexer, Dashboard, agentes, Docker e Squid**, juntamente com segmentação de rede, hardening SSH, DNS e alertas por email, cria um laboratório representativo de um ambiente de monitorização de segurança.

Mais do que a instalação de uma ferramenta SIEM, o projeto demonstra a construção de uma infraestrutura orientada para **visibilidade, deteção, análise e resposta a eventos de segurança**.

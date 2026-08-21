# Projeto SIEM com Wazuh


## Visão Geral

Este repositório contém informações e tutoriais práticos sobre a monitorização de agentes através do servidor Wazuh. Nele, encontrará guias passo a passo de como aplicar diversas configurações nos seus *endpoints* para otimizar e fortalecer a visibilidade de segurança.

> **Autor:** Cleison Máquina  
> **Ano:** 2026  
> **Projeto:** SIEM / Monitorização de Segurança  
> **Website:** [soc.cleisonmq.com](https://soc.cleisonmq.com)

---

## Ficheiros de Configuração:
- **No Agente:** `/var/ossec/etc/ossec.conf`
- **No Servidor (Configuração de Grupo):** `/var/ossec/etc/shared/<nome_do_grupo>/agent.conf` *(ou através do painel Wazuh em Gerenciamento de Grupos)*

---
## 1. Controlo de Integridade de Ficheiros (FIM)

O módulo **File Integrity Monitoring (FIM)** permite monitorizar os principais ficheiros e diretórios dos servidores ou *endpoints*. Todos os ficheiros incluídos nesta configuração passam a ser auditados continuamente pelo agente, gerando alertas no servidor Wazuh sempre que for detetada qualquer criação, modificação ou eliminação.

As configurações detalhadas podem ser consultadas no seguinte ficheiro:

1. [Servidor Proxy Squid](servidor-squid/01_integridade_de_ficheiros/File_Integrity_Monitoring_squid.md)

## 2. Monitoramento de logs (Thread Hunter)

Apoesar do wazuh ter uma vasta game de monitoramento nos endpoits ainda existem ficheiros de logs que o wazuh nao faz a recolha automatica, posto isso a a nessecidoe de realizar a recolha dos logs destes ficheiros para posterior analise.

As configurações detalhadas podem ser consultadas no seguinte ficheiro:

1. [Servidor Proxy Squid](servidor-squid/02_monitoramento_de_logs/Threat_Hunting_squid.md)
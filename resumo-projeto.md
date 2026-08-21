# 🛡️ Projeto SIEM com Wazuh

## Implementação de uma Infraestrutura de Monitorização e Segurança

> "Este projeto teve como objetivo implementar uma infraestrutura SIEM utilizando Wazuh, permitindo a recolha, análise e monitorização centralizada de eventos de segurança provenientes de diferentes sistemas da rede. A implementação permitiu desenvolver uma solução próxima de um ambiente empresarial real, integrando múltiplos componentes de segurança numa arquitetura distribuída."

---

# 🎯 Objetivo do Projeto

Implementar uma infraestrutura SIEM utilizando **Wazuh**, capaz de:

* Recolher logs de vários sistemas;
* Monitorizar eventos de segurança;
* Detetar atividades suspeitas;
* Gerar alertas automáticos;
* Centralizar toda a informação de segurança num único local.

---

# 🏗️ Arquitetura Implementada

```text
┌─────────────────┐
│     Agentes     │
│ Docker / Squid  │
│ Servidores      │
└────────┬────────┘
         │ Logs
         ▼
┌─────────────────┐
│ Wazuh Manager   │
│ Análise e Regras│
└────────┬────────┘
         │ Alertas
         ▼
┌─────────────────┐
│ Wazuh Indexer   │
│ Armazenamento   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Dashboard       │
│ Visualização    │
└─────────────────┘
```

---

# 🔍 Componentes da Infraestrutura

## 1️⃣ Wazuh Agent

Instalado nos endpoints monitorizados.

### Funções

* Recolha de logs;
* Monitorização da integridade dos ficheiros;
* Inventário do sistema;
* Envio de eventos para o Wazuh Manager.

---

## 2️⃣ Wazuh Manager

O componente central da plataforma.

### Responsabilidades

* Receber eventos dos agentes;
* Aplicar regras de deteção;
* Correlacionar eventos;
* Gerar alertas de segurança;
* Gerir os agentes da infraestrutura.

---

## 3️⃣ Wazuh Indexer

Responsável pelo armazenamento e indexação dos dados.

### Funções

* Armazenamento de eventos;
* Indexação de logs;
* Pesquisa rápida de informação;
* Suporte às análises e dashboards.

---

## 4️⃣ Wazuh Dashboard

Interface gráfica da plataforma.

### Permite

* Visualização de alertas em tempo real;
* Gestão de agentes;
* Investigação de incidentes;
* Criação de dashboards e relatórios.

---

# 🖥️ Infraestrutura do Laboratório

## Servidor Wazuh Indexer

### Recursos

* 2 vCPU
* 4 GB RAM
* 30 GB Disco
* CentOS Stream 10

### Função

Armazenar e indexar todos os eventos recebidos.

---

## Servidor Wazuh Manager + Dashboard

### Recursos

* 2 vCPU
* 4 GB RAM
* 30 GB Disco
* Ubuntu Server 22.04 LTS

### Função

Processar eventos, gerar alertas e disponibilizar a interface gráfica da plataforma.

---

# 🌐 Segmentação da Rede

A infraestrutura foi dividida em duas redes.

## Rede NAT

```text
10.x.x.x/24
```

Utilizada para:

* Agentes;
* Servidores monitorizados;
* Fontes externas de logs.

---

## Rede Host-Only

```text
192.168.x.x/24
```

Utilizada para:

* Wazuh Manager;
* Wazuh Indexer;
* Wazuh Dashboard.

### Benefícios

* Isolamento dos serviços críticos;
* Redução da superfície de ataque;
* Maior controlo sobre as comunicações internas.

---

# 🔐 Medidas de Segurança Implementadas

## Hardening SSH

Foram aplicadas diversas medidas de segurança:

* Utilização de chaves SSH ED25519;
* Desativação da autenticação por password;
* Alteração da porta padrão;
* Bloqueio de passwords vazias;
* Gestão segura de acessos remotos.

### Resultado

Redução significativa do risco de ataques de força bruta e acessos não autorizados.

---

# 📦 Integração de Agentes

## Docker Server

Foi integrado um servidor Docker na infraestrutura.

### Benefícios

* Monitorização de containers;
* Centralização de logs;
* Visibilidade dos eventos do ambiente Docker;
* Deteção de atividades suspeitas.

---

## Proxy Squid

Foi integrado um servidor Proxy Squid.

### Objetivos

* Registar acessos Web;
* Monitorizar tráfego HTTP e HTTPS;
* Centralizar logs de navegação;
* Melhorar a visibilidade da atividade da rede.

---

# 📧 Sistema de Alertas

Foi configurado um sistema de notificações por correio eletrónico.

## Componentes

* Wazuh Manager;
* Postfix;
* Gmail SMTP.

### Fluxo dos Alertas

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
 Email de Alerta
```

### Benefícios

* Notificação imediata de incidentes;
* Monitorização contínua;
* Resposta mais rápida a eventos críticos.

---

# 🌍 Integração DNS

Foi configurado um registo DNS para o acesso ao Dashboard.

### Vantagens

* Acesso simplificado;
* Maior facilidade de administração;
* Eliminação da necessidade de utilizar endereços IP diretamente.

---

# 🧠 Competências Demonstradas no Projeto

✅ Implementação de SIEM

✅ Administração Linux

✅ Configuração de ambientes distribuídos

✅ Hardening SSH

✅ Gestão e análise de logs

✅ Integração de agentes

✅ Configuração SMTP

✅ Segmentação de redes

✅ Monitorização de eventos

✅ Gestão de alertas

✅ Integração DNS

✅ Monitorização de ambientes Docker

---

# 🚀 Resultados Obtidos

A infraestrutura implementada permitiu:

* Centralizar eventos de segurança;
* Monitorizar sistemas em tempo real;
* Gerar alertas automáticos;
* Integrar diferentes fontes de logs;
* Melhorar a visibilidade sobre a infraestrutura;
* Facilitar a deteção de comportamentos suspeitos.

A solução encontra-se preparada para a integração de novos agentes e expansão futura da infraestrutura.

---

# 📌 Conclusão

Este projeto demonstrou a implementação de uma infraestrutura SIEM baseada em Wazuh, utilizando uma arquitetura distribuída composta por Manager, Indexer, Dashboard e agentes de monitorização.

A solução permitiu centralizar eventos de segurança, monitorizar sistemas, gerar alertas automáticos e melhorar a visibilidade da infraestrutura através de uma plataforma unificada.

Além dos aspetos técnicos, o projeto evidenciou competências em administração de sistemas Linux, segmentação de redes, hardening de serviços, integração de agentes e gestão de eventos de segurança, aproximando o ambiente laboratorial de um cenário real de produção.

🛡️ Segurança não consiste apenas em instalar ferramentas, mas em construir uma infraestrutura capaz de monitorizar, analisar e responder eficazmente a eventos e ameaças.

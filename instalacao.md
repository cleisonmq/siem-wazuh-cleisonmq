# 🛡️ Wazuh – Instalação Multi-Nó (Guia Simples)

Este guia explica de forma simples como instalar o **Wazuh em arquitetura distribuída (multi-nó)** com:

- 📦 Wazuh Indexer
- 🧠 Wazuh Manager
- 🌐 Wazuh Dashboard
- 📧 Alertas por email

---

# 🧠 1. Visão Geral da Arquitetura

```text
Agentes → Manager → Indexer → Dashboard
```

### 🔹 Função de cada componente:

- **Manager** → recebe e processa logs dos agentes
- **Indexer** → guarda e indexa logs (OpenSearch)
- **Dashboard** → interface web
- **Agentes** → enviam dados dos sistemas monitorizados

---

# ⚙️ 2. Requisitos Iniciais (TODOS OS NÓS)

Executar em todos os servidores:

```bash
sudo apt update && sudo apt upgrade -y
```

Instalar ferramentas básicas:

```bash
apt install curl unzip wget -y
```

---

# 📥 3. Download do Instalador Wazuh

Em todos os nós:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.14/config.yml
```

---

# 🧩 4. Gerar Configuração do Cluster

👉 Executar APENAS no nó principal (gerador):

```bash
bash wazuh-install.sh --generate-config-files
```

### 📦 Isso cria:

- Certificados SSL/TLS
- Passwords
- Ficheiro importante:

```
wazuh-install-files.tar
```

---

# 📦 5. Instalar Wazuh Indexer (primeiro nó)

No servidor Indexer:

```bash
bash wazuh-install.sh --wazuh-indexer node-1
```

---

## 🔍 Verificar Indexer

```bash
curl -k -u admin:SENHA https://IP_INDEXER:9200
```

---

# 📤 6. Enviar ficheiro TAR para outros nós

Depois de criar o cluster:

```bash
scp /root/wazuh-install-files.tar root@IP_MANAGER:/root/
scp /root/wazuh-install-files.tar root@IP_DASHBOARD:/root/
```

---

# 🧠 7. Instalar Wazuh Manager

No servidor Manager:

```bash
mv /root/wazuh-install-files.tar /root/
bash wazuh-install.sh --wazuh-server wazuh-1
```

---

## 🔥 Portas do Manager

- 1514/UDP → agentes
- 1515/TCP → registo
- 55000/TCP → API

---

# 🌐 8. Instalar Wazuh Dashboard

No servidor Dashboard:

```bash
mv /root/wazuh-install-files.tar /root/
bash wazuh-install.sh --wazuh-dashboard dashboard
```

---

## 🔥 Porta do Dashboard

- 443/TCP → acesso web HTTPS

---

# ⚠️ Problema comum (Dashboard não instala)

## ❌ Erro: pacotes não encontrados

### ✔️ Solução:

```bash
curl -fsSL https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
```

```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list
```

```bash
apt update
```

---

# 📧 9. Configurar Alertas por Email

Instalar Postfix:

```bash
apt install postfix mailutils -y
```

---

## ⚙️ Configuração SMTP

```
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_use_tls = yes
```

---

## 🔐 Criar credenciais

```bash
echo "[smtp.gmail.com]:587 USER@gmail.com:PASSWORD" > /etc/postfix/sasl_passwd
postmap /etc/postfix/sasl_passwd
```

---

## 🔄 Reiniciar serviço

```bash
systemctl restart postfix
```

---

# 📊 10. Ativar alertas no Wazuh

Editar:

```
/var/ossec/etc/ossec.conf
```

Adicionar:

```xml
<global>
  <email_notification>yes</email_notification>
  <smtp_server>localhost</smtp_server>
  <email_from>USER@gmail.com</email_from>
  <email_to>DESTINO@gmail.com</email_to>
</global>
```

---

# 🧾 11. Resultado Final

✔ Wazuh multi-nó funcional  
✔ Indexer ativo  
✔ Manager a processar logs  
✔ Dashboard online  
✔ Alertas por email ativos  

---

# 🛡️ Conclusão

Infraestrutura SIEM completa para monitorização e segurança.

# Monitorização e Threat Hunting com Logs do Squid no Wazuh

## Índice
1. [Coleta de Logs](#coleta-de-logs)
2. [Decodificadores Personalizados (local_decoder.xml)](#decodificadores-personalizados)
3. [Esquema de Hierarquia das Regras](#esquema-de-hierarquia-das-regras)
4. [Filtro e Regras de Deteção (local_rules.xml)](#filtro-e-regras-de-deteção)
5. [Teste de Validação](#teste-de-validação)
6. [Validação no Dashboard](#validação-no-dashboard)

---

## Coleta de Logs
Configure a leitura dos logs do Squid no agente Wazuh (`/var/ossec/etc/ossec.conf`):

```xml
<!-- Monitorizar um diretório completo -->
<localfile>
  <location>/var/log/squid/*.log</location>
  <log_format>syslog</log_format>
</localfile>

<!-- Monitorizar um ficheiro específico -->
<localfile>
  <location>/var/log/squid/access.log</location>
  <log_format>syslog</log_format>
</localfile>
```

> **Nota:** Certifique-se de que o formato de log do Squid inclui o cabeçalho *User-Agent* (`%{User-Agent}>h`) e os bytes transferidos para que os decodificadores consigam extrair os campos corretamente aceda o link para descarregar o ficheiro de configuração "https://github.com/cleisonmq/proxy-squid-cleisonmq/blob/main/proxy-squid-config.txt".

---

## Decodificadores Personalizados
**Ficheiro:** `/var/ossec/etc/decoders/local_decoder.xml`

Extrai os campos de IP de origem (`srcip`), ação (`action`), código de retorno HTTP (`id`), tamanho em bytes (`size`), método/informação extra (`extra_data`), destino (`url`) e utilizador (`srcuser`):

```xml
<decoder name="squid-custom">
  <prematch>^\d+\.\d+\s+\d+\s+\S+\s+\S+/\d+</prematch>
  <regex>^\d+\.\d+\s+\d+\s+(\S+)\s+(\S+)/(\d+)\s+(\d+)\s+(\S+)\s+(\S+)\s+(\S+)</regex>
  <order>srcip, action, id, size, extra_data, url, srcuser</order>
</decoder>
```

---

## Esquema de Hierarquia das Regras
A ordem e a relação pai/filho devem seguir esta estrutura para garantir que *downloads* e acessos específicos não sejam suprimidos por regras genéricas:

```text
rule_id="100349" (Regra Raiz Pai - Nível 0, agrupa o decoder customizado)
├── rule_id="100348" (Filha da 100349 - Nível 0, ignora CSS/JS/fontes)
├── rule_id="100371" (Filha da 100349 - Nível 8, download pesado em HTTPS/HTTP > 5 MB)
├── rule_id="100360" (Filha da 100349 - Nível 10, download de executáveis em HTTP)
├── rule_id="100361" (Filha da 100349 - Nível 6, download de arquivos compactados em HTTP)
├── rule_id="100362" (Filha da 100349 - Nível 4, download de documentos em HTTP)
└── rule_id="100350" (Filha da 100349 - Nível 5, regra base de acesso normal < 5 MB com supressão de 60s)
    ├── rule_id="100351" (Filha da 100350 - Nível 5, acesso via Microsoft Edge)
    ├── rule_id="100352" (Filha da 100350 - Nível 5, acesso via Mozilla Firefox)
    ├── rule_id="100353" (Filha da 100350 - Nível 5, acesso via Google Chrome)
    ├── rule_id="100354" (Filha da 100350 - Nível 5, acesso via Apple Safari)
    └── rule_id="100359" (Filha da 100350 - Nível 8, validação/conectividade de host)
```

---

## Filtro e Regras de Deteção

```xml
<group name="squid,proxy,">

  <!-- 1. Silencia requisições estáticas leves -->
  <rule id="100348" level="0">
    <if_sid>35000</if_sid>
    <match>\.css|\.js|\.woff2|\.woff|\.ico|\.svg</match>
    <description>Squid-ignore: Ignorar ficheiros estáticos/secundários</description>
  </rule>

  <!-- 2. Download Volumoso em HTTPS / HTTP (> 5 MB: 7 dígitos ou mais no log bruto) -->
  <rule id="100371" level="8">
    <if_sid>35000</if_sid>
    <match type="pcre2">\s+(?:[5-9]\d{6}|\d{8,})\s+(?:CONNECT|GET)</match>
    <description>Squid-Transferência-iniciada: Download volumoso detectado (> 5 MB) [IP: $(srcip)] -> $(url)</description>
    <group>download,media,heavy_traffic,</group>
  </rule>

  <!-- 3. Downloads HTTP diretos por extensão de ficheiro -->
  <rule id="100360" level="10">
    <if_sid>35000</if_sid>
    <id>^200$|^301$|^302$</id>
    <match>\.exe|\.msi|\.bat|\.ps1|\.sh|\.bin|\.cmd|\.vbs|\.apk|\.iso|\.img</match>
    <description>Squid-Transferência-iniciada: Download de ficheiro executável/instalador detectado [IP: $(srcip)] -> $(url)</description>
    <group>download,security_risk,</group>
  </rule>

  <rule id="100361" level="6">
    <if_sid>35000</if_sid>
    <id>^200$|^301$|^302$</id>
    <match>\.zip|\.rar|\.7z|\.tar\.gz|\.tgz|\.tar</match>
    <description>Squid-Transferência-iniciada: Download de arquivo compactado detectado [IP: $(srcip)] -> $(url)</description>
    <group>download,</group>
  </rule>

  <rule id="100362" level="4">
    <if_sid>35000</if_sid>
    <id>^200$|^301$|^302$</id>
    <match>\.pdf|\.docx?|\.xlsx?|\.pptx?</match>
    <description>Squid-Transferência-iniciada: Download de documento detectado [IP: $(srcip)] -> $(url)</description>
    <group>download,</group>
  </rule>

  <!-- 4. Regra Base para Acesso Geral Web (Tráfego Normal abaixo de 5 MB) -->
  <rule id="100350" level="5" ignore="60">
    <if_sid>35000</if_sid>
    <id>^200$|^301$|^302$|^304$</id>
    <match type="pcre2">\s+\d{1,6}\s+(?:CONNECT|GET|POST|HEAD)</match>
    <description>Squid-Acesso-autorizado: O [IP: $(srcip)] acedeu ao site $(url)</description>
  </rule>

  <!-- 5. Identificação por Navegador com Supressão de 60s por regra -->
  <rule id="100351" level="5" ignore="60">
    <if_sid>100350</if_sid>
    <match>Edg/|Edge/</match>
    <description>Squid-Acesso-autorizado: O [IP: $(srcip)] acedeu via Microsoft Edge ao site $(url)</description>
  </rule>

  <rule id="100352" level="5" ignore="60">
    <if_sid>100350</if_sid>
    <match>Firefox/</match>
    <description>Squid-Acesso-autorizado: O [IP: $(srcip)] acedeu via Mozilla Firefox ao site $(url)</description>
  </rule>

  <rule id="100353" level="5" ignore="60">
    <if_sid>100350</if_sid>
    <match>Chrome/</match>
    <description>Squid-Acesso-autorizado: O [IP: $(srcip)] acedeu via Google Chrome ao site $(url)</description>
  </rule>

  <rule id="100354" level="5" ignore="60">
    <if_sid>100350</if_sid>
    <match>Safari/</match>
    <description>Squid-Acesso-autorizado: O [IP: $(srcip)] acedeu via Apple Safari ao site $(url)</description>
  </rule>

  <!-- 6. Alerta de Verificação / Conectividade de Host -->
  <rule id="100359" level="8" ignore="300">
    <if_sid>100350</if_sid>
    <match>detectportal.firefox.com|connectivitycheck.gstatic.com|generate_204|msftconnecttest.com|captive.apple.com|connectivity-check.ubuntu.com</match>
    <description>Squid-Autenticação-bem-sucedida: O host [IP: $(srcip)] foi validado via $(url)</description>
    <group>authentication_success,pci_dss_10.2.5,</group>
  </rule>

</group>
```

---

## Validação no Dashboard

1. Aceda ao painel web do **Wazuh Dashboard**.
2. No menu principal, selecione **Threat Hunting**.
3. Aceda à aba **Events** e filtre por:
   - `rule.groups: squid`

## Validação no CLI
1. No servidor whazuh coloque o comando "/var/ossec/bin/wazuh-logtest"
2. Aceda o o acess.log do seruivor squid copiei uma linha de log e cole no servoidr wazuh com o comando a rodar.

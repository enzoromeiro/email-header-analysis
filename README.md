# 🛡️ Email Header Analysis: Autenticidade e Prevenção contra Phishing

Este projeto foca na análise de cabeçalhos de e-mail para verificar 
a autenticidade do remetente e identificar indicadores de phishing.

**Quem enviou?** (Rastreamento do servidor remetente via campo Received)

**É legítimo?** (Verificação de autenticidade via SPF, DKIM e DMARC)

**É confiável?** (Investigação de reputação via ferramentas OSINT)

A análise dessas camadas permite determinar se um e-mail é legítimo 
ou um potencial phishing, auxiliando a tomada de decisão em um SOC.

---

### 📨 Rastreando o Caminho do E-mail

O campo `Received` no cabeçalho registra cada salto que o e-mail 
percorreu até chegar ao destinatário.

- O **primeiro `Received` de baixo para cima** é o servidor de origem 
  — contém o domínio e o IP de quem enviou.
- O **primeiro `Received` de cima para baixo** é o servidor 
  destinatário — quem recebeu e entregou o e-mail.

No e-mail analisado:
- **Remetente:** `a48-56.smtp-out.amazonses.com` — IP `54.240.48.56`
- **Destinatário:** servidor Google — IPv6 
  `2002:a17:522:c1e3:b0:65e:8d1b:5887`

![cabeçalho received](/img/received.png)

---

### 🔐 SPF — Sender Policy Framework

O SPF verifica se o servidor que enviou o e-mail estava autorizado 
a usar o domínio presente no `Return-Path`.

No e-mail analisado, o domínio `amazonses.com` autoriza o IP 
`54.240.48.56` a enviar e-mails em seu nome — **SPF: pass**.

Caso o SPF falhasse, significaria que o domínio não autorizou aquele 
servidor a enviar mensagens usando seu domínio no `Return-Path`. 
O resultado do SPF é usado posteriormente na verificação do DMARC.

![resultado spf](/img/spf.png)

---

### ✍️ DKIM — DomainKeys Identified Mail

O DKIM garante a integridade do e-mail e a autenticidade do remetente 
através de criptografia assimétrica:

1. O servidor remetente calcula um hash do cabeçalho e do corpo do e-mail
2. Assina o hash com a **chave privada** do domínio
3. O servidor destinatário busca a **chave pública** no registro DNS 
   do domínio
4. Descriptografa a assinatura e obtém o hash original
5. Recalcula o hash do conteúdo recebido e compara — se batem, 
   integridade confirmada

No e-mail analisado, o DKIM passou para o domínio `americanas.com.br`, 
confirmando que o conteúdo não foi alterado e que o remetente tinha 
a chave privada desse domínio — **DKIM: pass**.

![resultado dkim](/img/dkim.png)

---

### 🏛️ DMARC — Domain-based Message Authentication

O DMARC confirma a identidade do domínio visível ao destinatário 
no campo `From`. Ele verifica se esse domínio bate com:
- O domínio do `Return-Path` que passou no SPF, **ou**
- O domínio que assinou o DKIM

No e-mail analisado, o domínio que assinou o DKIM (`americanas.com.br`) 
é o mesmo do `From` — alinhamento confirmado, **DMARC: pass**.

O DMARC define também a política para e-mails que falharem:
- `reject` → bloqueia o e-mail
- `quarantine` → envia para spam
- `none` → nenhuma ação, apenas monitoramento

![resultado dmarc](/img/dmarc.png)

---

### 🔍 Investigação OSINT

Além das verificações de autenticação, ferramentas OSINT foram 
utilizadas para investigar a reputação do IP e dos domínios envolvidos.

**IP do servidor remetente (`54.240.48.56`):**
- **AbuseIPDB** — verificação de histórico de abuso e reportes
- **VirusTotal** — reputação agregada de múltiplos engines

![osint ip](/img/osint-ip.png)

**Domínio do `From` e do DKIM (`americanas.com.br`):**
- **Whois** — data de criação do domínio (domínio recém-criado 
  é red flag)
- **URLScan.io** — IP do domínio, localização, provedor e 
  sub-requisições geradas ao carregar a página. Domínios ou IPs 
  estranhos nessas sub-requisições, especialmente de países usados 
  como *bullet-proof hosting*, são indicadores de alerta.

![osint dominio](/img/osint-dominio.png)

---

### ⚠️ Ações em Caso de E-mail Suspeito

- **Nunca clicar em links** nem realizar download ou executar 
  arquivos provenientes do e-mail
- **Consultar o IP** do servidor remetente no AbuseIPDB e VirusTotal
- **Investigar os domínios** do `From` e do DKIM no Whois, VirusTotal 
  e URLScan.io
- **Ficar atento ao conteúdo** — promessas milagrosas, promoções 
  absurdas, solicitação de dados, suposta invasão de conta ou débitos 
  não quitados são táticas clássicas de phishing. Na dúvida, acesse 
  o site ou aplicativo oficial diretamente, sem usar o link do e-mail

---

#### Considerações finais: Um campo isolado nem sempre indica algo, mas a correlação entre autenticação falhando, domínio recém-criado e conteúdo alarmante pode confirmar um phishing.
#### Não existe solução definitiva em segurança da informação — as ameaças evoluem, as ferramentas evoluem, e a capacidade de análise crítica é o que diferencia um bom analista SOC.

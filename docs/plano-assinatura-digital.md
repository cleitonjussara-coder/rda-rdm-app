# Plano – Assinatura Digital Própria (Petermann)

> Versão 1 · nov/2026 · Status: proposta para decisão
> Escopo: módulo de assinatura eletrônica dentro do sistema Petermann (RDA/RDM + RH), hospedado na Locaweb.

---

## 1. Objetivo

Permitir que a empresa e os colaboradores assinem documentos **dentro do próprio sistema**, sem depender de plataforma terceira (ZapSign, Clicksign, D4Sign, DocuSign), com:

- validade jurídica para documentos internos de RH e operação;
- comprovação de **quem** assinou, **quando**, **onde** e **o quê**;
- garantia de que o documento **não foi alterado** depois de assinado;
- verificação pública por QR Code.

**Documentos-alvo (fase inicial):** recibo de entrega de EPI, termos de responsabilidade (veículo, ferramentas, celular), termos de ciência de políticas internas, advertências, aditivos simples, aceite de RDA/RDM e repasses.

**Fora do escopo inicial:** documentos que exigem assinatura qualificada ICP-Brasil do colaborador ou de terceiros. Ver seção 2.3.

---

## 2. Base legal (validar com o jurídico/contador)

### 2.1 Fundamentos
| Norma | O que garante |
|---|---|
| **MP 2.200-2/2001, art. 10, §2º** | Assinaturas eletrônicas **fora da ICP-Brasil** são válidas quando **admitidas pelas partes** ou aceitas por quem recebe o documento. É a base da assinatura própria. |
| **Lei 14.063/2020** | Define os níveis **simples, avançada e qualificada**. É voltada principalmente a órgãos públicos, mas suas definições são a referência usada no mercado. |
| **CPC, art. 411** | Documento é autêntico quando a autoria é identificada por meio legal de certificação, inclusive eletrônico. |
| **LGPD (Lei 13.709/2018)** | Tratamento de dados pessoais e de evidências (IP, geolocalização, selfie). |

### 2.2 Âncora jurídica: Termo de Adesão
Cada colaborador assina **uma vez**, no papel ou na admissão, um **Termo de Adesão à Assinatura Eletrônica**, declarando que:
1. reconhece como sua a assinatura feita no sistema com seu login, senha e código de confirmação;
2. é responsável pelo sigilo das credenciais;
3. concorda com a coleta das evidências (data/hora, IP, dispositivo, localização);
4. pode pedir cópia de qualquer documento assinado.

> Sem esse termo, a assinatura própria fica juridicamente frágil. **É o item mais importante do plano.**

### 2.3 Quando NÃO usar a assinatura própria
- Documentos que a lei ou o destinatário exigem com **certificado ICP-Brasil** (alguns atos perante órgãos públicos, cartórios e bancos).
- Documentos assinados por **terceiros externos** que não aderiram ao termo (fornecedores, clientes). Nesses casos, aceitar **assinatura gov.br**, que é gratuita para o cidadão: a pessoa assina no gov.br, envia o PDF e o sistema registra a validação em validar.iti.gov.br.

---

## 3. Como funciona (modelo em duas camadas)

```
 Colaborador                           Empresa (servidor)
 ──────────────                        ─────────────────────────────
 Login + senha                         PDF congelado + hash SHA-256
 + código (e-mail/WhatsApp)    ──►     Registro de evidências
 + aceite explícito                    Trilha de auditoria encadeada
 (+ selfie opcional)                   Selo final com e-CNPJ ICP-Brasil
                                       QR Code de verificação pública
```

1. **Camada do signatário: assinatura eletrônica avançada.** Identifica a pessoa por vários fatores (login, código de uso único, dispositivo) e vincula a assinatura ao documento pelo hash.
2. **Camada da empresa: selo ICP-Brasil.** Ao final, o servidor assina o PDF com o **certificado e-CNPJ A1** da empresa, no padrão PAdES. Qualquer alteração posterior invalida o selo e aparece ao abrir no Adobe Reader ou em validar.iti.gov.br.

Essa combinação dá ao documento **autoria comprovada pelas evidências** e **integridade garantida por criptografia oficial**, sem custo por documento.

---

## 4. Fluxo de assinatura

| # | Etapa | Quem | Detalhes |
|---|---|---|---|
| 1 | Criar documento | RH/gestor | A partir de um modelo (EPI, termo, advertência…) com dados do colaborador preenchidos. |
| 2 | Definir signatários | RH/gestor | Ordem (sequencial ou paralela), prazo, tipo de autenticação. |
| 3 | Congelar | Sistema | Gera o PDF final, calcula o **SHA-256** e grava a versão. A partir daqui, o conteúdo não muda. |
| 4 | Notificar | Sistema | E-mail ou WhatsApp com link, além de aviso no app. |
| 5 | Autenticar | Signatário | Login + **código de 6 dígitos** com validade de 10 min (e-mail grátis; WhatsApp/SMS opcional e pago). |
| 6 | Ler e aceitar | Signatário | Visualiza o PDF inteiro, marca "Li e concordo" e confirma. Selfie ou rubrica desenhada são opcionais. |
| 7 | Registrar evidências | Sistema | Data/hora do servidor (UTC-3), IP, dispositivo, geolocalização (se autorizada), hash do documento, ID do código usado. |
| 8 | Concluir | Sistema | Quando todos assinam: anexa a **página de evidências** ao PDF, aplica o **selo e-CNPJ** e o **QR Code**, calcula o hash final. |
| 9 | Distribuir e arquivar | Sistema | Envia cópia a todos os signatários, guarda no servidor com backup no Drive da empresa. |
| 10 | Verificar | Qualquer pessoa | QR Code → página pública que mostra signatários, datas e confere o hash do arquivo enviado. |

**Recusa:** o signatário pode recusar com justificativa. Fica registrado e o RH é avisado.
**Cancelamento:** só antes da conclusão, com motivo, e também registrado.

---

## 5. Evidências e trilha de auditoria

Cada evento (criado, enviado, visualizado, código enviado, código validado, assinado, recusado, concluído, baixado, verificado) gera um registro **somente de inclusão**, nunca editado nem apagado:

```
evento_id | documento_id | tipo | usuario_id | data_hora | ip | user_agent | geo | dados_json | hash_anterior | hash_registro
```

- **Encadeamento:** `hash_registro = SHA-256(dados do evento + hash_anterior)`. Se alguém alterar um registro antigo direto no banco, a corrente quebra e a verificação acusa.
- **Página de evidências** anexada ao PDF: lista de signatários (nome, CPF parcialmente mascarado, e-mail), data/hora, IP, método de autenticação, hash do documento original e QR Code.

---

## 6. Arquitetura técnica (Locaweb, PHP + MySQL)

### 6.1 Componentes
| Componente | Opção sugerida | Observação |
|---|---|---|
| Banco | **MySQL** (já incluso no plano) | Pré-requisito: sair do SQLite. |
| Geração de PDF | mPDF ou TCPDF (PHP) | Funciona em hospedagem gerenciada. |
| Selo PAdES com e-CNPJ | TCPDF (`setSignature`) ou biblioteca de assinatura PAdES em PHP | Validar a compatibilidade em validar.iti.gov.br logo no início (prova de conceito). |
| Hash | `hash('sha256', ...)` nativo do PHP | |
| Código de uso único | E-mail (SMTP do plano Locaweb) | WhatsApp/SMS só se necessário. |
| Carimbo de tempo | Hora do servidor + trilha encadeada (fase 1); carimbo ICP-Brasil (opcional, fase 3) | O carimbo oficial reforça a prova de data, mas é pago. |
| Armazenamento | Pasta **fora da `public_html`** + backup diário no Drive da empresa | Arquivos servidos só por script com checagem de permissão. |

### 6.2 Tabelas novas
```
documentos            (id, modelo_id, titulo, status, criado_por, criado_em, hash_original, hash_final, caminho_pdf)
documento_signatarios (id, documento_id, colaborador_id, ordem, papel, status, prazo)
assinaturas           (id, signatario_id, data_hora, ip, user_agent, geo, metodo_auth, otp_id, aceite_texto, selfie_path)
otp_codigos           (id, signatario_id, canal, codigo_hash, expira_em, usado_em, tentativas)
eventos_auditoria     (encadeada – seção 5)
modelos_documento     (id, nome, conteudo_html, campos_json, versao, ativo)
termos_adesao         (colaborador_id, versao_termo, data_aceite, evidencia_path)
```

### 6.3 Proteção do certificado e-CNPJ (ponto crítico)
O arquivo `.pfx` do certificado A1 é a **chave da empresa**. Se vazar, alguém pode selar documentos em nome da Petermann.
- Guardar **fora da `public_html`**, com permissão de leitura só para o PHP.
- Senha do certificado em variável de ambiente ou config fora do repositório, **nunca no código nem no GitHub**.
- Registrar cada uso do certificado na trilha de auditoria.
- Na hospedagem compartilhada/gerenciada, o risco é maior que num VPS. Se o volume crescer ou houver documentos sensíveis, considerar mover a função de selar para um VPS.
- Renovar anualmente (A1 vale 1 ano) e manter um procedimento de troca.

---

## 7. Segurança e LGPD

- **Permissões:** colaborador vê só os próprios documentos; gestor vê o núcleo; RH/admin vê tudo. A regra fica no PHP, em **toda** consulta.
- **Código de uso único:** guardar só o hash; no máximo 5 tentativas; bloqueio temporário.
- **Selfie e geolocalização:** opcionais, com finalidade informada no Termo de Adesão; podem ser dados sensíveis (biometria).
- **Retenção:** definir prazos por tipo de documento com o contador ou o jurídico; alguns documentos trabalhistas exigem guarda longa.
- **Backup:** diário, com teste de restauração mensal.
- **Página pública de verificação:** não expor CPF completo, endereço nem conteúdo do documento sem autenticação.

---

## 8. Fases e estimativa

Valores com os mesmos parâmetros da planilha `docs/estimativa-custos-petermann-app.xlsx` (freelancer R$ 115/h, contingência 20%).

| Fase | Entregas | Horas | c/ contingência | Freelancer |
|---|---|---|---|---|
| **0 – Pré-requisitos** | Migrar SQLite → MySQL; código PHP atual no repositório; Termo de Adesão aprovado pelo jurídico | 8–20 h (já previsto na planilha) | – | – |
| **A – Núcleo** | Modelos → PDF, congelamento + hash, código por e-mail, tela de assinatura, evidências, trilha encadeada, página de evidências | 60–90 h | 90 h | **~R$ 10,4 mil** |
| **B – Selo e verificação** | Selo PAdES com e-CNPJ, QR Code, página pública de verificação, prova de conceito no validar.iti.gov.br | 30–50 h | 48 h | **~R$ 5,5 mil** |
| **C – Recursos extras** | Vários signatários com ordem, lembretes, assinatura em lote (vários EPIs de uma vez), recusa/cancelamento, carimbo de tempo, WhatsApp | 20–40 h | 36 h | **~R$ 4,1 mil** |
| **Total** | | **110–180 h** | **174 h** | **~R$ 20 mil** |

### Custos recorrentes
| Item | Estimativa | Obrigatório? |
|---|---|---|
| Certificado e-CNPJ A1 | ~R$ 200–300/ano | Sim (fase B) |
| E-mail para códigos | R$ 0 (SMTP do plano Locaweb) | Sim |
| WhatsApp/SMS | ~R$ 0,05–0,20 por mensagem | Não |
| Carimbo de tempo ICP-Brasil | pago por carimbo ou pacote | Não |

> Preços de referência aproximados; cotar antes de contratar.

---

## 9. Própria x plataforma de mercado

| Critério | Assinatura própria | Plataforma (ZapSign, Clicksign…) |
|---|---|---|
| Desenvolvimento | ~R$ 20 mil (174 h) | ~R$ 4,6 mil (integração, ~40 h) |
| Custo por documento | ~R$ 0 | ~R$ 1–2 (ou plano mensal) |
| Com 40 documentos/mês | ~R$ 250/ano (certificado) | ~R$ 720/ano + certificado opcional |
| Controle e independência | Total; dados ficam na empresa | Depende do fornecedor |
| Robustez jurídica | Boa **com** Termo de Adesão + selo e-CNPJ | Alta, já testada em tribunais |
| Responsabilidade técnica | Da empresa (segurança, certificado, backup) | Do fornecedor |

**Leitura honesta:** com o volume atual (cerca de 40 documentos por mês), a plataforma de mercado **custa menos**. A diferença de desenvolvimento (cerca de R$ 15 mil) levaria décadas para se pagar só com a economia por documento. A assinatura própria se justifica por:
- **controle e independência**, com dados e evidências dentro da empresa;
- **volume maior**: a conta se inverte a partir de algumas centenas de documentos por mês;
- **integração total** com RDA/RDM, EPI e RH, numa só tela e num só login;
- **desenvolvimento interno**: se é você quem desenvolve, o custo real é o seu tempo, não os R$ 20 mil.

**Caminho intermediário recomendado:** fazer **a Fase A** (núcleo próprio, sem custo por documento) para os documentos internos de baixo risco (EPI, termos de ciência, aceite de RDA/RDM) e usar **gov.br ou uma plataforma** só para os poucos documentos de maior risco (contratos, advertências, rescisões) até a Fase B estar madura.

---

## 10. Riscos e mitigação

| Risco | Impacto | Mitigação |
|---|---|---|
| Colaborador nega ter assinado | Alto | Termo de Adesão + código de uso único + trilha encadeada + selo e-CNPJ |
| Vazamento do certificado e-CNPJ | Alto | Seção 6.3; avaliar VPS para a função de selar |
| Selo PAdES feito em PHP não validar no ITI | Médio | Prova de conceito **antes** da Fase B |
| Perda de documentos | Alto | Backup diário no Drive + teste mensal de restauração |
| Hospedagem gerenciada limitar extensões PHP (OpenSSL, etc.) | Médio | Testar no plano atual na Fase 0 |
| Mudança de entendimento jurídico | Médio | Revisão anual com o jurídico; manter a opção gov.br/plataforma |

---

## 11. Próximos passos

1. **Jurídico/contador:** validar a lista de documentos-alvo, aprovar o Termo de Adesão e definir os prazos de guarda.
2. **Código:** subir a versão PHP/SQLite da Locaweb para este repositório (sem o arquivo `.db`).
3. **Fase 0:** migrar para MySQL e testar no plano Locaweb se as extensões `openssl` e `mbstring` e as bibliotecas de PDF funcionam.
4. **Certificado:** comprar o e-CNPJ A1 (só na Fase B).
5. **Prova de conceito (2–4 h):** gerar um PDF, selar com o e-CNPJ e validar em validar.iti.gov.br.
6. **Fase A** com piloto em um núcleo (ex.: Cristalina) usando recibo de EPI.

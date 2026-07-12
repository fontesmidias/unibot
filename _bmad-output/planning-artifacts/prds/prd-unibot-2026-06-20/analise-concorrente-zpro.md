# Análise do Concorrente — Z-PRO (changelog completo jul/2024 → jul/2026)

> Substitui a `RE_ANALISE.md` (que era engenharia reversa a partir de páginas de venda). Esta análise é destilada do **changelog completo do canal de updates do Z-PRO no Telegram** (`changelog-concorrente/result.json`, ~130 releases em 2 anos), incluindo os **fixes reportados por usuários** — que revelam onde o produto realmente sangra, não só o que o marketing diz.

---

## 1. O que o Z-PRO é

SaaS white-label **auto-hospedado**: o cliente/revendedor roda a plataforma na própria VPS. Cada atualização é um procedimento manual (backup obrigatório, substituir `dist`/`src`, `npm install`, `sequelize migrate`, rebuild, `pm2 restart`). Distribuído por autoinstalador com licença por chave/e-mail e **expiração forçada de APIs antigas** para coagir upgrades. Frontend migrou Vue2 → Vue3 → **React/Next.js** (reescrita total lançada em v4.0.0, mai/2026). Ritmo de release altíssimo: hotfix quase diário, updates oficiais a cada 2-4 semanas.

---

## 2. As 7 dores estruturais do Z-PRO (validadas por 2 anos de fixes)

### 2.1 Conexão WhatsApp não oficial = dor nº 1, crônica e nunca resolvida ⭐⭐⭐
O sinal mais forte de todo o changelog. **Quase toda release** (~130) carrega o mesmo aviso em negrito: *"Para usuários BAILEYS: atualize fora do expediente… pode ser necessário reler o QRCode ou recriar o canal."* Episódios graves:
- **Duplicação de mensagens** (out/2024): 3 posts no mesmo dia, incluindo trocar o pacote Baileys por um fork específico do GitHub à mão.
- Edição manual de `node_modules` em produção (WWebJS quebrou grupos → patch em `Utils.js`, depois downgrade forçado).
- **@lid** (mudança de identificador do WhatsApp, 2025): meses de retrabalho em ~5 versões.
- **Troca completa da biblioteca Baileys em produção** por instabilidade (set/2025), obrigando todos a reler QR.
- **Passkey do WhatsApp** (2026) quebrou vinculação de novos dispositivos em TODAS as libs não oficiais → tiveram que criar uma **extensão de navegador** paliativa para importar sessão.
- **"LIDs envenenados"** (2026): contatos duplicados faziam **mensagens irem para o destinatário errado** — falha grave de confiabilidade, exigiu ação destrutiva de consolidação.

**A "solução" do Z-PRO foi quantidade, não robustez:** hoje suporta **7 APIs de WhatsApp** (Baileys, WWebJS, MEOW/Wuzapi, UAZAPI, Z-API, Evolution, WABA) + contingência/failover manual. Isso **multiplica a superfície de bug**: o mesmo defeito (envio de mídia, áudio, mensagem rápida, nono dígito) reaparece corrigido API por API.

### 2.2 Escala derruba o sistema — Redis/filas/memória/banco ⭐⭐
Memory leak admitido no frontend; `pm2 restart all` agendado 1-2×/dia no cron como mitigação; opção de **desligar o Redis**; reescrita do Redis com beta-testers; campanhas processadas **fora do Redis**; sequência longa de correções de índices de banco; "filtro de tickets no socket recomendável >1000 atendimentos/dia". Modo **Cluster** e **S3/MinIO** só chegaram em 2026 para aliviar. Disparo em massa **exige manter a tela aberta** (não roda server-side) — limitação arquitetural.

### 2.3 Self-hosted transfere o ônus operacional ao cliente ⭐⭐
Toda a fricção de VPS, backup, update manual e fragmentação de versões é do cliente. **Incidente Hostinger (fev/2026):** o provedor de VPS bloqueou webhooks da Meta (tratados como DDoS); o Z-PRO **não tinha controle** e só pôde recomendar trocar de VPS — risco de churn admitido por eles próprios.

### 2.4 Multitenancy, isolamento e RBAC foram retrofit ⭐
Nasceu single-tenant. Vazamento de notificação cross-tenant (2×), pastas `public` compartilhadas entre empresas (separadas só em out/2024), **"Update Crítico" de permissão de superadmin** (nov/2024), RBAC quebrando acesso de admin ao ser introduzido (2025), regressões de visibilidade de tickets vai-e-volta.

### 2.5 Risco de plataforma junto à Meta ⭐
A **própria conta Meta da ZDG foi restringida** (mai/2026), derrubando os apps nativos de Facebook/Instagram dos clientes. OAuth compartilhado = ponto único de falha. Resposta: virar **Meta Business Partner / Tech Provider** com Embedded Signup e **Score de Qualidade Meta** por número (throttle/bloqueio automático de número amarelo/vermelho).

### 2.6 QA / processo de release imaturo ⭐
Hotfixes reemitidos no mesmo dia ("BAIXE NOVAMENTE") são frequentes (≥6 no bloco final); recompilações de emergência; TMA/TME retornando negativos; vaga de suporte citando **~40 chamados/dia**. Falta CI/CD e homolog confiável.

### 2.7 Identidade de contato brasileira é bug perene ⭐
Normalização do **nono dígito** reescrita ~6+ vezes em rotas distintas; combinada com @lid/LID e IGSID longos (Instagram) — a identidade do contato é uma fonte recorrente de erro de entrega.

---

## 3. Onde o Z-PRO é forte (não subestimar)

- **Amplitude de recursos** e **velocidade de entrega** brutais.
- **Multi-LLM nativo** (OpenAI, Gemini, Groq, Claude, DeepSeek, Grok, Qwen, Ollama, LM Studio, Dify) + **Copiloto de IA** (resumo, sugestão, **análise de sentimento**, **detecção de urgência**, tradução inline em 8 idiomas).
- **WABA oficial profundo**: Embedded Signup, templates, catálogo, flows, janela de 24h, Modo Híbrido (oficial + não oficial no mesmo número).
- **Chatflow visual estilo n8n** (VueFlow), com subfluxos, webhooks roteáveis, condições e captura de variável.
- **API externa forte** (+26 rotas por tenant, Bearer token, Postman, community nodes n8n).
- **Omnichannel agressivo**: Instagram, Messenger, Webmail, marketplaces (Mercado Livre, OLX, Nuvemshop, WooCommerce), TikTok/YouTube/LinkedIn.
- **Agendamento nativo** tipo Google Agenda com **turmas/vagas por horário** (clínicas, cursos).
- **Billing sofisticado**: 4 gateways (Asaas, Stripe, Pagar.me, Mercado Pago), cobrança preventiva, bloqueio automático por inadimplência, controle de features/canais por plano.
- **Voz**: Wavoip (ligações no WhatsApp) + SIP/WebRTC.

---

## 4. Implicações diretas para o unibot

| # | Achado no Z-PRO | Implicação para o unibot |
|---|---|---|
| I1 | Conexão não oficial quebra a cada update; "solução" é somar APIs frágeis | **Tese confirmada:** camada de canal unificada com contrato + **testes de contrato por provedor** + fallback + updates que **não derrubam sessão**. Poucas conexões estáveis > muitas frágeis. É o diferencial nº 1. |
| I2 | Nono dígito / @lid / LID / Passkey entregam mensagem errada | **Identidade de contato robusta** desde o dia 1, abstraída na camada de canal (ver FR-63). |
| I3 | WABA precisa de template + janela de 24h para msg iniciada pela empresa | Suporte a **template WABA + gestão da janela de 24h** é MVP-crítico para o canal oficial (ver FR-64/FR-65). |
| I4 | Redis/memória/banco não escalam; disparo exige tela aberta | NFR de **backpressure, filas server-side e observabilidade**; disparo em massa **assíncrono server-side** (não depender de tela aberta). |
| I5 | Self-hosted + incidente Hostinger | **SaaS gerenciado** elimina a dor inteira — o cliente nunca toca em VPS. Confirma o posicionamento. |
| I6 | Multitenancy/RBAC/segurança retrofit e com falhas críticas | **Isolamento + RBAC + auditoria + 2FA** desde o início (ver FR-67, NFR-4/9). |
| I7 | Risco de plataforma Meta (conta restringida, OAuth compartilhado) | **Risco R7**: caminho WABA oficial de primeira classe, gestão de saúde/qualidade de número, conformidade de template. |
| I8 | Debounce de mensagens antes da IA (diferencial deles) | Agrupar mensagens em sequência antes de acionar IA (ver FR-66) evita respostas fragmentadas. |
| I9 | Delay aleatório anti-banimento em campanhas | Reforça o throttling do disparo (FR-33). |
| I10 | Copiloto de IA (sentimento, urgência, resumo, tradução) | Tablestakes emergente — candidato forte ao **roadmap** próximo do unibot. |
| I11 | LGPD-like tarde (2FA, blur de dados, log de auditoria) | unibot entra com **governança de dados desde o início** (FR-68, NFR-4). |
| I12 | Direção deles: Híbrido WABA, omnichannel, marketplaces, API externa, agendamento com turmas | Informa o **roadmap** do unibot e o que é tablestakes vs. diferencial. |

---

## 5. Leitura estratégica de uma frase

> O Z-PRO ganhou em **amplitude e velocidade**, mas paga o preço em **fragilidade de conexão, dívida técnica, QA instável e um modelo self-hosted caro de suportar**. O unibot não compete em amplitude no MVP — compete **exatamente nos pontos onde eles tropeçaram por 2 anos**: conexão estável, operação gerenciada, isolamento robusto e onboarding sem atrito.

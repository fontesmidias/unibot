# Addendum — PRD unibot

Profundidade técnica e de contexto que não pertence ao corpo do PRD (capacidades), mas alimenta arquitetura, solução técnica e go-to-market. Decisões e alternativas rejeitadas registradas aqui.

---

## 1. Decisões técnicas de mecanismo

### 1.1 Camada de canal (WhatsApp) — provedores do MVP
Contrato único `enviar / receber / status` sobre **três drivers** (atualização do brief, confirmada por Bruno em 2026-06-20):

| Driver | Tipo | Papel |
|---|---|---|
| **WABA Cloud API (Meta)** | Oficial | Provedor principal/estável; sujeito a políticas, templates e preço por conversa da Meta. |
| **Evolution API** | Não oficial (self-hostável) | Driver não oficial primário. |
| **Baileys** | Não oficial (biblioteca) | Driver não oficial secundário / fallback adicional. |

Requisitos de engenharia derivados (lições do Z-PRO, ver `analise-concorrente-zpro.md`):
- Cada provedor isolado como **plugin/driver substituível** — atualizar/trocar um não quebra os demais.
- **Testes de contrato por provedor** obrigatórios (evitar os ciclos semanais de "correção geral" do ZPRO).
- Health-check + reconexão + **fallback transparente** entre drivers (FR-4, FR-5).

> Nota de risco: o brief original previa apenas "1 API não oficial". A expansão para 2 não oficiais aumenta a superfície de manutenção — é trade-off aceito por Bruno em troca de redundância de fallback. Registrado no `.decision-log.md`.

### 1.2 Camada de IA — provedores do MVP
Contrato único sobre múltiplos provedores (atualização do brief; brief dizia "1 provedor"):

| Provedor | Observação |
|---|---|
| **OpenAI** | Modelos de chat/qualificação. |
| **Gemini (Google)** | Alternativa/fallback. |
| **Groq** | Inferência de baixa latência. |
| **OpenRouter** | Agregador — provê acesso a vários modelos por uma única integração; reforça a abstração de "camada de IA plugável". |

Derivados: seleção de provedor/modelo por tenant (FR-36), contabilização de tokens por tenant para cota/billing (FR-39), fallback entre provedores (FR-40).

### 1.3 Multitenancy
- Modelo: **SaaS multitenant operado por Bruno** (não white-label revendido a desenvolvedores — esse é o modelo do ZPRO, rejeitado).
- Cada tenant = empresa-cliente final.
- Isolamento de dados por tenant; sessões de WhatsApp isoladas por tenant.

### 1.4 Monetização — tiers puros
- **Tiers puros** (assinatura fixa por faixa). Sem cobrança variável de excedente.
- Gating em 4 dimensões: nº de atendentes, nº de canais/conexões, cota de IA (tokens), volume de mensagens/contatos.
- Cobrança: cartão recorrente (gateway BR) + Pix.
- **Gateway (decisão técnica em aberto — Q6):** candidatos **Asaas** (cartão+Pix+boleto nativos, usado pelo próprio ZPRO), **Mercado Pago** (marca forte BR, Pix nativo), **Pagar.me/Stripe** (mais robustos para escala). A escolher antes de implementar billing.
- Trial gratuito por tempo determinado (7–14 dias) antes da conversão (FR-51).
- Cotas de IA/WABA por tier = mecanismo de proteção de margem.

---

## 2. Alternativas consideradas e rejeitadas

| Alternativa | Por que rejeitada |
|---|---|
| **Pricing híbrido** (assinatura + excedente) | Mais complexo de comunicar e faturar; previsibilidade não compensou a fricção de venda. |
| **Pricing puro consumo** | Imprevisível para a PME orçar; contraria a estratégia de venda simples. |
| **White-label para desenvolvedores** (modelo ZPRO) | unibot é operação direta B2B SaaS; Bruno é operador único. |
| **MVP com 1 API não oficial** | Substituído por 2 (Evolution + Baileys) para redundância de fallback. |
| **MVP com 1 provedor de IA** | Substituído por 4 (camada de IA plugável). |
| **White-label de tenant no MVP** | Posicionamento ainda evoluindo; diferido para pós-MVP. |

---

## 3. Inteligência competitiva (ZPRO) — inputs de design

> Fonte primária agora é **`analise-concorrente-zpro.md`** (destilada do changelog completo de ~130 releases, jul/2024→jul/2026). A `RE_ANALISE.md` está **superada** e pode ser desconsiderada.

Stack observada no ZPRO (matéria-prima, **não** prescrição): Frontend Vue 3 + Quasar → migrado para React/Next.js em 2026. Backend Node.js + Socket.IO. Redis. BD relacional. Nginx + Certbot. Docker/Portainer; modo Cluster e S3/MinIO chegaram em 2026 para escala.

**Aprendizados técnicos do changelog que viram requisitos de engenharia do unibot:**

| Achado (2 anos de fixes do ZPRO) | Requisito de design do unibot |
|---|---|
| Baileys quebra sessão a cada update; troca de fork à mão; Passkey/`@lid`/LID quebram vínculo e entregam msg errada | Camada de canal com **contrato único + testes de contrato por provedor**; **identidade de contato robusta** (FR-63); updates que não derrubam sessão (NFR-1). |
| WABA exige template aprovado + janela de 24h para msg iniciada pela empresa | Gestão de janela e templates WABA no MVP (FR-64, FR-65). |
| Redis/memória/banco não escalam; `pm2 restart` no cron; disparo exige tela aberta | Filas **server-side**, backpressure, índices e observabilidade (FR-70, NFR-2, NFR-7). |
| Isolamento/RBAC/segurança retrofit; "update crítico" de permissão de superadmin | Isolamento + RBAC + 2FA + auditoria desde o dia 1 (FR-47, FR-67, NFR-4, NFR-9). |
| Conta Meta da empresa do Z-PRO (ZDG) restringida derrubou clientes; OAuth compartilhado = ponto único de falha | Caminho WABA oficial de 1ª classe; abstração de identificador; roadmap BSP próprio (R7). |
| Debounce de mensagens antes da IA; delay aleatório anti-ban | FR-66 (debounce IA), FR-33 (delay aleatório). |
| QA imaturo: hotfixes reemitidos no mesmo dia | CI/CD, homolog, deploy zero-downtime (NFR-3). |

**Tablestakes emergentes observados (para o roadmap):** Copiloto de IA (sentimento/urgência/resumo/tradução), Modo Híbrido WABA, API externa por tenant + n8n, agendamento nativo com turmas, billing multi-gateway com cobrança preventiva.

---

## 4. Contexto qualitativo

- **Idioma:** Português BR.
- **Voz/tom:** não explicitado nos insumos; assumir tom executivo B2B SaaS, direto e sem jargão técnico para o cliente final (que não deve "ver o VPS").
- **Posicionamento do moat:** execução e operação, não tecnologia secreta. Comunicação comercial deve enfatizar estabilidade gerenciada e custo previsível.

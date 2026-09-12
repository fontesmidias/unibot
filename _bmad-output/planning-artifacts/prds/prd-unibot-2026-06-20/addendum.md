# Addendum — PRD unibot

> 🔒 **Versão anonimizada para versionamento.** As referências competitivas deste documento foram
> substituídas por designações neutras (Concorrente A, B, C). Os achados técnicos, datas e versões
> são preservados integralmente — apenas a identificação das fontes foi removida.
> A versão com as fontes nomeadas é mantida fora do controle de versão.


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

Requisitos de engenharia derivados (lições do Concorrente A, ver `analise-concorrente.md`):
- Cada provedor isolado como **plugin/driver substituível** — atualizar/trocar um não quebra os demais.
- **Testes de contrato por provedor** obrigatórios (evitar os ciclos semanais de "correção geral" do Concorrente A).
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
- Modelo: **SaaS multitenant operado por Bruno** (não white-label revendido a desenvolvedores — esse é o modelo do Concorrente A, rejeitado).
- Cada tenant = empresa-cliente final.
- Isolamento de dados por tenant; sessões de WhatsApp isoladas por tenant.

### 1.4 Monetização — tiers puros + implantação

> **Atualizado em 2026-09-12 (decisão D1):** modelo comercial passa de autoatendimento para **venda assistida** (referência: Concorrente B). Ver §1.5.

- **Tiers puros** (assinatura fixa por faixa). Sem cobrança variável de excedente.
- **Receita de implantação** cobrada uma vez na contratação, além da assinatura (referência Concorrente B: R$ 999–1.899 por 4–6 reuniões).
- Gating em 4 dimensões: nº de atendentes, nº de canais/conexões, cota de IA (tokens), volume de mensagens/contatos.
- Cobrança: cartão recorrente (gateway BR) + Pix.
- **Gateway — Q6 RESOLVIDA em 2026-09-12: ASAAS** (primário), com **Mercado Pago** como alternativa contratual. Análise completa em `../../decisao-gateway-pagamento.md`.
  - **Motivos:** (1) único candidato que atende FR-53 *e* FR-54 sem gate — o **Stripe está eliminado** porque seu Pix é *invite-only* e exige 60 dias de histórico de processamento com aprovação discricionária; (2) entrega mais do FR-71 (dunning) pronto — é o único que documenta a política de retry em detalhe operacional (3 tentativas no vencimento às 8h/14h/20h, 3 no dia seguinte, +2 a cada 24h) com eventos de webhook próprios; (3) menor risco operacional para operador único — KYC de menor atrito, sem mensalidade, sem taxa sobre recusa. O **Pagar.me/Stone** foi preterido por concentrar os relatos públicos mais graves de bloqueio de conta sem notificação prévia.
  - **Custo (30 tenants × R$ 700, mix 60/40):** Asaas R$ 409/mês (1,95% do MRR) · MP R$ 585 · Pagar.me R$ 611 · Stripe R$ 757. **A economia é o desempate, não o argumento** — a diferença entre o melhor e o pior é menor que um dia de trabalho construindo dunning que o gateway entrega pronto.
  - 🔴 **AÇÃO BLOQUEANTE ANTES DE IMPLEMENTAR BILLING:** a **tokenização de cartão em produção do Asaas exige aprovação do gerente de conta** (não é self-service). Sem ela **FR-53 não existe**. Confirmar antes de escrever código; se negada, virar para Mercado Pago.
  - ⚠️ **Condição que inverte a escolha:** a vantagem de custo vem quase toda do **Pix de taxa fixa (R$ 1,99)** — 0,28% num ticket de R$ 700, mas **1,3–2,0% num ticket de R$ 100–150**. Se o tier de entrada ficar nessa faixa (Q3), reavaliar: o Mercado Pago (Pix 0,99%) passa à frente.
  - **Mitigação de lock-in:** billing atrás de contrato próprio, espelhando a filosofia das camadas de canal e de IA (ver `decisao-gateway-pagamento.md` §6).
- ~~Trial gratuito por tempo determinado (7–14 dias)~~ **Revogado em 2026-09-12 (D1):** não há trial autônomo. O acesso é liberado na contratação, com período de implantação acompanhada (FR-51). Ver §5.
- Cotas de IA/WABA por tier = mecanismo de proteção de margem.

---

## 2. Alternativas consideradas e rejeitadas

| Alternativa | Por que rejeitada |
|---|---|
| **Pricing híbrido** (assinatura + excedente) | Mais complexo de comunicar e faturar; previsibilidade não compensou a fricção de venda. |
| **Pricing puro consumo** | Imprevisível para a PME orçar; contraria a estratégia de venda simples. |
| **White-label para desenvolvedores** (modelo Concorrente A) | unibot é operação direta B2B SaaS; Bruno é operador único. |
| **MVP com 1 API não oficial** | Substituído por 2 (Evolution + Baileys) para redundância de fallback. |
| **MVP com 1 provedor de IA** | Substituído por 4 (camada de IA plugável). |
| **White-label de tenant no MVP** | Posicionamento ainda evoluindo; diferido para pós-MVP. |

---

## 3. Inteligência competitiva (Concorrente A) — inputs de design

> Fonte primária agora é **`analise-concorrente.md`** (destilada do changelog completo de ~130 releases, jul/2024→jul/2026). A `RE_ANALISE.md` está **superada** e pode ser desconsiderada.

Stack observada no Concorrente A (matéria-prima, **não** prescrição): Frontend Vue 3 + Quasar → migrado para React/Next.js em 2026. Backend Node.js + Socket.IO. Redis. BD relacional. Nginx + Certbot. Docker/Portainer; modo Cluster e S3/MinIO chegaram em 2026 para escala.

**Aprendizados técnicos do changelog que viram requisitos de engenharia do unibot:**

| Achado (2 anos de fixes do Concorrente A) | Requisito de design do unibot |
|---|---|
| Baileys quebra sessão a cada update; troca de fork à mão; Passkey/`@lid`/LID quebram vínculo e entregam msg errada | Camada de canal com **contrato único + testes de contrato por provedor**; **identidade de contato robusta** (FR-63); updates que não derrubam sessão (NFR-1). |
| WABA exige template aprovado + janela de 24h para msg iniciada pela empresa | Gestão de janela e templates WABA no MVP (FR-64, FR-65). |
| Redis/memória/banco não escalam; `pm2 restart` no cron; disparo exige tela aberta | Filas **server-side**, backpressure, índices e observabilidade (FR-70, NFR-2, NFR-7). |
| Isolamento/RBAC/segurança retrofit; "update crítico" de permissão de superadmin | Isolamento + RBAC + 2FA + auditoria desde o dia 1 (FR-47, FR-67, NFR-4, NFR-9). |
| Conta Meta da empresa do Concorrente A (Concorrente A) restringida derrubou clientes; OAuth compartilhado = ponto único de falha | Caminho WABA oficial de 1ª classe; abstração de identificador; roadmap BSP próprio (R7). |
| Debounce de mensagens antes da IA; delay aleatório anti-ban | FR-66 (debounce IA), FR-33 (delay aleatório). |
| QA imaturo: hotfixes reemitidos no mesmo dia | CI/CD, homolog, deploy zero-downtime (NFR-3). |

**Tablestakes emergentes observados (para o roadmap):** Copiloto de IA (sentimento/urgência/resumo/tradução), Modo Híbrido WABA, API externa por tenant + n8n, agendamento nativo com turmas, billing multi-gateway com cobrança preventiva.

---

## 4. Contexto qualitativo

- **Idioma:** Português BR.
- **Voz/tom** *(definido em 2026-09-12 a partir dos benchmarks — substitui o "não explicitado nos insumos" anterior)*: **coloquial brasileiro, direto, sem jargão técnico**, na fronteira entre o acolhimento do Concorrente C e a objetividade comercial do Concorrente B.
  - **Vocabulário de resultado comercial, não de suporte técnico** (Concorrente B): fala-se de vender, atender, converter e não perder cliente — não de "tickets", "SLA de fila" ou "instâncias".
  - **CTAs em primeira pessoa** (Concorrente C): "Quero ver funcionando", "Quero conectar meu WhatsApp" — não imperativos da empresa.
  - **O cliente nunca deve "ver o VPS"**: nenhuma palavra de infraestrutura vaza para a interface do tenant.
  - **Honestidade operacional como traço de marca**: quando algo falha, o produto diz o que falhou e o que está sendo feito. É coerente com NFR-1/NFR-2 e é o contraste direto com as reclamações de suporte inacessível do concorrente.
  - Insumo obrigatório para `bmad-ux`.
- **Posicionamento do moat:** execução e operação, não tecnologia secreta. Comunicação comercial deve enfatizar estabilidade gerenciada e custo previsível.


---

## 5. Modelo comercial — venda assistida (2026-09-12)

### 5.1 A decisão

O unibot adota **venda assistida**: aquisição por demonstração agendada e implantação guiada e cobrada, sem autoatendimento nem trial autônomo. Substitui a premissa de onboarding self-service que vigorava desde o brief.

**Fundamento (benchmark, não preferência estética):** a Concorrente B entrega menos funcionalidade que o Concorrente A — 4 canais contra 10, IA como add-on de R$ 599 contra multi-LLM incluso, cota de disparo contra ilimitado — e cobra de 5 a 9 vezes mais (R$ 349–989/mês + implantação, contra ~R$ 166/mês equivalente). A diferença é embalagem, prova social, verticalização, onboarding e equipe comercial. Num mercado onde a amplitude funcional já é commodity, o prêmio está na condução da venda e da implantação.

### 5.2 Alternativas consideradas nesta rodada

| Alternativa | Por que rejeitada |
|---|---|
| **Autoatendimento puro** (modelo Concorrente C / Concorrente A, e o que o PRD assumia até 2026-07-09) | Ticket baixo (R$ 33–166/mês equivalentes) exige volume alto de clientes e investimento em marketing de entrada para sustentar receita. Incompatível com operador único sem capital de aquisição. |
| **Híbrido** (autoatendimento + venda assistida em paralelo) | Exigiria construir e manter *dois* caminhos de aquisição e dois modelos de suporte simultaneamente, com um só operador. Adiado — pode ser reavaliado quando houver equipe. |

### 5.3 Consequências assumidas

- **R4 (operador único) é agravado, não mitigado.** O Concorrente B sustenta esse modelo com ~70–90 pessoas. Cada demonstração, implantação e renovação passa pelo Bruno. Mitigações no produto: FR-81 (demonstração com dados fictícios), FR-82 (trilha de implantação registrada), FR-49/FR-50. Mitigação fora do produto, indispensável: **limitar deliberadamente o número de clientes novos por mês** à capacidade real de implantação (Q7).
- **Economia a validar (A8):** a receita de implantação precisa cobrir o custo do tempo do operador no onboarding. Se não cobrir, a principal justificativa econômica do modelo cai. **Cronometrar a primeira implantação real.**
- **Feature 8 deixa de ser caminho crítico do produto e passa a ser caminho crítico comercial:** FR-81 sobe de prioridade (não se demonstra sem ele), FR-48 (captura de lead) é trabalho de site/landing mais do que de produto.

### 5.4 Diferenciais contratuais derivados do benchmark

As reclamações graves do Concorrente B no plataformas públicas de reclamação concentram-se em suporte inacessível (média de 9 dias de resposta), instabilidade e **multa de 50% no cancelamento** — contra um site que promete 99,9% de uptime e suporte 24/7. Além disso, a categoria tem vácuo de avaliações (0 reviews no diretórios de software B2B e no diretórios de software B2B).

Cláusulas vendáveis derivadas, coerentes com NFR-1 e NFR-2: **ausência de fidelidade contratual**, **SLA explícito com crédito automático** e **página de status pública**. Registradas como `[ASSUMPTION]` A7 — dependem de a confiabilidade real sustentá-las.

---

## 6. Inteligência competitiva — atualização de set/2026

Fonte: `analise-concorrente-delta-set2026.md` (10/07→12/09/2026, 30 versões, 3 updates oficiais e 27 hotfixes — um a cada 2,4 dias).

**Nenhuma das 7 dores estruturais foi resolvida.** As dores 2.3 (self-hosted) e 2.5 (risco de plataforma Meta) agravaram; 2.6 (QA imaturo) permanece — v4.0.3.1 reemitida no mesmo dia, TPR/TTE corrigido 4 vezes em 4 semanas.

**Aprendizados que viraram requisito:**

| Achado (evidência) | Requisito derivado |
|---|---|
| Canal Meta "conectado" que parou de receber por dias (10/08) | **FR-73** — health-check por fluxo de dados, não por estado de sessão |
| Loop de reconexão Baileys elevando memória e reiniciando o servidor (27/07) | **FR-72** — circuit breaker por conexão |
| "Entregue" exibido para mensagem que nunca chegou (30/07, 04/08, 14/08) | **FR-74** — estados de entrega distintos |
| BSUID duplicado perdendo mensagem; contato WABA por username (09/09) | **FR-76** — identidade de contato com telefone opcional *(decisão de modelo de dados)* |
| Atendimento preso em robô mudo após exclusão de fila/fluxo (11/08, 29/08) | **FR-79** — proteção de referência em uso + rota de escape |
| Race conditions na ingestão: rajada perdia mensagem, evento simultâneo duplicava conversa (29/08) | **NFR-11** — idempotência por ID de mensagem e lock na criação de ticket *(aplicado ao texto do NFR-11 em 2026-09-12)* |
| Dois lock-ins de IA quebraram em 7 semanas: cutover forçado da OpenAI (24/07) e modelos Gemini desligados pelo Google (12/09) | **FR-40** estendido — fallback também para *modelo descontinuado*, não só para falha |
| Concorrente A sem 2FA em set/2026, após release dedicada a segurança | **FR-67** vira diferencial vendável; **FR-78** como complemento |

**Antipadrões a não repetir:** somar APIs sem convergir (o Concorrente A chegou a ~10 provedores de WhatsApp); monetizar a mitigação de risco da própria arquitetura ("Score Blindado"); publicar "planejamento concluído, execução pendente" como item de changelog.

**Sinal estratégico:** em 18/08 a Concorrente A moveu o painel de licença para servidor próprio, "sem depender da sua instalação estar no ar" — está migrando do self-hosted por comportamento. A vantagem do unibot em "operação gerenciada" pode estreitar; a vantagem em "o cliente nunca toca em VPS" permanece.

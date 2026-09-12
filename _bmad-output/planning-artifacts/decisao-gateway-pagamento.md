# Decisão Q6 — Gateway de Pagamento do unibot

> 🔒 **Versão anonimizada para versionamento.** As referências competitivas deste documento foram
> substituídas por designações neutras (Concorrente A, B, C). Os achados técnicos, datas e versões
> são preservados integralmente — apenas a identificação das fontes foi removida.
> A versão com as fontes nomeadas é mantida fora do controle de versão.


> **Status:** Recomendação técnica para decisão de Bruno.
> **Data da pesquisa:** 2026-09-12. Todas as taxas citadas foram consultadas nesta data.
> **Decisão que destrava:** Q6 do PRD (aberta desde 2026-06-20), bloqueando a implementação da Feature 9 (Billing & Assinatura), que está no primeiro corte da fatia vendável.
> **Candidatos:** Asaas · Mercado Pago · Pagar.me · Stripe.

> ⚠️ **Aviso de volatilidade:** taxas de gateway mudam sem aviso prévio e são frequentemente negociáveis por volume. Todo valor abaixo traz data e fonte. Valores que não puderam ser confirmados em fonte oficial estão marcados `[NÃO VERIFICADO]` e **não devem ser usados como base de contrato**.

---

## §1. Contexto da decisão e critérios

### 1.1 O que o unibot precisa cobrar

O unibot é um **SaaS B2B multitenant** vendido por **assinatura recorrente em tiers puros** (sem cobrança de excedente) a PMEs brasileiras, operado por **uma pessoa só** (Bruno). Isso define o perfil da decisão de forma bastante restritiva:

- Ticket médio esperado: **R$ 500 a R$ 700/mês por tenant**.
- Volume no horizonte de 12–18 meses: **dezenas de tenants**, não milhares.
- Transações: **recorrentes, de valor alto e baixa frequência** (1 cobrança/mês/tenant). Não é e-commerce de alto volume e ticket baixo.
- Operação: **sem equipe financeira**. Tudo que o gateway não automatizar vira trabalho manual de Bruno ou código que Bruno precisa escrever e manter.

### 1.2 Requisitos funcionais que a decisão precisa atender

| FR | Exigência | Implicação para a escolha |
|---|---|---|
| **FR-52** | Planos por tier com limites em 4 dimensões | Gateway precisa suportar **planos/assinaturas com valor fixo**; o enforcement de cota é do unibot, não do gateway. |
| **FR-53** | Assinatura recorrente por **cartão** (gateway brasileiro) | **Subscriptions nativas** com tokenização de cartão e cobrança automática mensal. Obrigatório. |
| **FR-54** | Pagamento por **Pix** | **Pix nativo**, com QR dinâmico e conciliação automática via webhook. Obrigatório. |
| **FR-55** | Enforcement de limites (aviso + bloqueio) | Depende de webhook confiável de status de pagamento. |
| **FR-56** | Medição de uso por ciclo | Interno ao unibot; gateway só precisa expor o ciclo de faturamento. |
| **FR-57** | **Histórico de faturas** visível ao tenant | Gateway precisa expor API de listagem de cobranças/faturas por cliente, com status e link/comprovante. |
| **FR-58** | **Suspensão por inadimplência** com fluxo de regularização | Webhooks de falha/atraso/recuperação, confiáveis e idempotentes. |
| **FR-71** | **Régua de cobrança preventiva (dunning)** | Aviso antes do vencimento + **retry automático de cartão recusado** + período de carência. *O quanto disso é nativo é o critério de maior peso desta decisão.* |
| **FR-51** | Trial gratuito 7–14 dias antes da conversão | Gateway precisa permitir assinatura com **primeira cobrança em data futura** (`nextDueDate`) ou trial nativo. |

### 1.3 Critérios de desempate (ponderação)

Para um **operador único** com **dezenas de tenants**, a ponderação não é a de um e-commerce:

1. **Peso alto — esforço de integração e de operação.** Cada hora que Bruno gasta construindo billing é uma hora que não vai para o produto. Dunning nativo vale mais do que 0,5% de taxa.
2. **Peso alto — cobertura Pix + cartão recorrente no mesmo gateway.** Dois gateways = duas integrações, duas conciliações, dois pontos de falha.
3. **Peso alto — risco de bloqueio de conta / KYC.** Operador único sem gerente de contas: uma conta bloqueada é a receita inteira parada.
4. **Peso médio — taxas.** Em ticket de R$ 500–700, a diferença percentual importa, mas o custo absoluto mensal ainda é pequeno frente ao custo de oportunidade de engenharia (ver §4).
5. **Peso médio — qualidade de API/SDK Node.js e sandbox.**
6. **Peso baixo — escala.** Nenhum dos quatro quebra em 100 tenants. Escala não é critério discriminante aqui.

### 1.4 Dado competitivo (registrado, não ponderado)

O **Concorrente A (concorrente) usa Asaas**. Isso é registrado como **inteligência competitiva**, não como argumento de mérito. O fato de o concorrente ter escolhido Asaas não é evidência de que Asaas seja tecnicamente superior — e tampouco de que seja inferior. A recomendação da §5 se sustenta sem esse dado; a coincidência é anotada em §7 como algo a *não* deixar influenciar.

Vale registrar, porém, um aprendizado **negativo** do Concorrente A que é diretamente relevante: o Concorrente A **só adicionou cobrança preventiva (dunning) tardiamente** (ver `addendum.md` §3 e FR-71). O unibot já nasce com esse requisito — o que eleva o peso do critério 5 (dunning nativo) na escolha.

---

## §2. Tabela comparativa (4 gateways × 13 critérios)

> Legenda: ✅ nativo/pronto · ⚠️ parcial ou com ressalva · ❌ ausente / precisa construir · `[NÃO VERIFICADO]` não confirmado em fonte oficial.

### 2.1 Critério 1 — Taxas

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Cartão à vista** | **R$ 0,49 + 2,99%** <br>*(promo 3 meses: 1,99%)* | **3,98% a 4,98%** conforme prazo de liberação <br>(4,98% na hora / 3,98% em 30 dias) | **4,19%** (plano Essencial) | **3,99% + R$ 0,39** |
| **Cartão recorrente/assinatura** | Mesma tabela do à vista (**R$ 0,49 + 2,99%**) — assinatura mensal é cobrança à vista recorrente | Mesma tabela de cartão | Mesma tabela de cartão | **3,99% + R$ 0,39** + **0,7% de Billing** sobre o volume recorrente |
| **Pix** | **R$ 1,99 por transação recebida** *(promo 3 meses: R$ 0,99)* — **valor fixo, não percentual** | **0,99%** | **0,99%** (site de ofertas) / 1,19% `[NÃO VERIFICADO]` — fontes divergem | **1,19% por Pix pago** — ⚠️ **acesso invite-only** (ver crit. 3) |
| **Boleto** | **R$ 1,99 por boleto recebido** *(promo 3 meses: R$ 0,99)* | `[NÃO VERIFICADO]` | `[NÃO VERIFICADO]` | **R$ 3,45 por boleto pago** |
| **Débito** | R$ 0,35 + 1,89% | 1,99% (Point) | `[NÃO VERIFICADO]` | n/a |
| **Negociação por volume** | `[NÃO VERIFICADO]` — não publicado; prática de mercado sugere possível | Tabela varia por faturamento mensal | **Sim** — plano "Flex" com **"TAXAS CUSTOMIZADAS"** (valores não divulgados) | Sim, para volume alto (não aplicável no horizonte do unibot) |
| **Fonte** | asaas.com/precos-e-taxas (2026-09-12) | calculadoradetaxas.com.br + sellsync.ai (2026-09-12) — ⚠️ **fontes de terceiros**, página oficial retornou 403 | pagar.me/ofertas (2026-09-12) | stripe.com/br/pricing (2026-09-12) |

> 🔴 **Ponto de atenção sobre o Asaas — este é o achado mais importante da §2.1:** a taxa de **Pix do Asaas é um valor FIXO de R$ 1,99**, não um percentual. Em ticket de R$ 500–700, isso equivale a **0,28%–0,40%** — contra 0,99%–1,19% percentuais dos concorrentes. Quanto maior o ticket, maior a vantagem. Para o perfil do unibot (ticket alto, baixo volume), essa é uma diferença estrutural, não marginal. Ver §4.

> 🔴 **Ponto de atenção sobre o Mercado Pago:** a taxa de cartão depende do **prazo de liberação escolhido**. Receber "na hora" custa ~4,98%; esperar 30 dias custa ~3,98%. Não existe um número único — o custo real é uma decisão de fluxo de caixa que Bruno toma. As fontes oficiais não puderam ser confirmadas (403); os números vêm de agregadores.

### 2.2 Critério 2 — Prazo de repasse (D+X) e antecipação

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Cartão** | **até 2 dias úteis** (à vista, sujeito a análise de crédito) | Configurável: **na hora / 14 dias / 30 dias** — a taxa varia com a escolha | **"a partir de 1 dia"**; ⚠️ novos vendedores podem ser retidos **até 15 dias** por análise | ⚠️ Transferências **contínuas, semanais ou mensais** configuráveis — D+X exato para BR `[NÃO VERIFICADO]` |
| **Pix** | Imediato na conta Asaas (liquidação instantânea) | Imediato | Imediato | `[NÃO VERIFICADO]` |
| **Boleto** | Antecipação **até 3 dias úteis** | `[NÃO VERIFICADO]` | `[NÃO VERIFICADO]` | `[NÃO VERIFICADO]` |
| **Antecipação** | Sim (boleto; cartão sujeito a análise) | Embutida na escolha do prazo (quanto antes, maior a taxa) | Sim `[NÃO VERIFICADO]` detalhes | ⚠️ Não é produto central no BR |
| **Avaliação** | ✅ Bom e previsível | ✅ Flexível, mas o custo é explícito | ⚠️ Retenção inicial de até 15 dias é risco de caixa no arranque | ⚠️ Menos transparente para BR |

### 2.3 Critério 3 — Pix nativo e qualidade da implementação

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Pix disponível** | ✅ Nativo, sem restrição de acesso | ✅ Nativo (Checkout Transparente e Pro) | ✅ Nativo | 🔴 **INVITE-ONLY** |
| **Restrição de acesso** | Nenhuma | Nenhuma | Nenhuma | **"Pix for Brazilian Stripe accounts is currently invite-only. Contact Stripe support to request access."** Exige conta **"in good standing"** e **"processed payments on Stripe for a minimum of 60 days"** |
| **QR dinâmico** | ✅ | ✅ | ✅ | ✅ (quando liberado) |
| **Conciliação automática** | ✅ Webhook `PAYMENT_RECEIVED` por cobrança | ✅ | ✅ | ✅ |
| **Pix em assinatura recorrente** | ✅ `billingType: PIX` em assinatura — gera cobrança Pix a cada ciclo | ✅ Pix suportado como meio em assinaturas | ⚠️ `[NÃO VERIFICADO]` se há Pix recorrente nativo em subscription | ✅ Desde **2026-04-22** (mandate options / Pix Automático) — **recurso muito recente** |
| **Avaliação** | ✅ **Melhor do grupo** — Pix de primeira classe, sem gate | ✅ Forte | ✅ Adequado | 🔴 **Bloqueante para o MVP** |

> 🔴 **Este critério, sozinho, elimina o Stripe do MVP.** FR-54 (Pix) é **obrigatório**. O Stripe exige 60 dias de histórico de processamento **antes** de poder sequer solicitar acesso ao Pix — ou seja, o unibot precisaria operar dois meses cobrando **só cartão**, e ainda assim sem garantia de aprovação ("contact our support team for a Pix risk evaluation"). Isso é incompatível com a meta de **primeiros pagantes em ~30 dias** (Q1 do PRD).

### 2.4 Critério 4 — Subscriptions/assinaturas nativas

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **API de assinatura** | ✅ `POST /v3/subscriptions` — objeto de 1ª classe | ✅ `POST /preapproval` (+ plano associado) | ✅ Plans + Subscriptions na Core API **v5** | ✅ Subscriptions — **referência do mercado** |
| **Ciclos** | ✅ `cycle: MONTHLY` (e outros) | ✅ semanal / mensal / anual | ✅ Intervalos configuráveis | ✅ Qualquer intervalo |
| **Primeira cobrança futura (trial FR-51)** | ✅ `nextDueDate` | ✅ | ✅ | ✅ Trial nativo (`trial_period_days`) |
| **Precisa orquestrar cobranças avulsas?** | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| **Avaliação** | ✅ Simples e direto | ✅ Adequado | ✅ Adequado | ✅ Melhor da categoria |

Os quatro entregam assinatura nativa. **Este critério não discrimina.**

### 2.5 Critério 5 — Dunning nativo *(critério de maior peso — FR-71)*

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Retry automático de cartão recusado** | ✅ **Sim, e documentado em detalhe:** 3 tentativas no dia do vencimento (8h, 14h, 20h — a cada 6h), mais 3 no dia seguinte, mais 2 a cada 24h — **totalizando 5 dias de tentativas** | ✅ Sim — *"o Mercado Pago cuida das cobranças e das novas tentativas em caso de pagamento recusado"* | ⚠️ `[NÃO VERIFICADO]` — política de retentativa não confirmada em doc oficial | ✅ **Smart Retries** — retry com **modelo de machine learning** que escolhe os melhores horários |
| **Régua de avisos ao cliente** | ✅ Notificações nativas ao cliente (e-mail/SMS/WhatsApp — **WhatsApp custa R$ 0,55/notificação**; pacote SMS/e-mail R$ 0,99) | ⚠️ `[NÃO VERIFICADO]` grau de configurabilidade | ⚠️ `[NÃO VERIFICADO]` | ✅ Configurável em Settings → Billing → Revenue Recovery |
| **Atualização de cartão expirado** | ⚠️ `[NÃO VERIFICADO]` se há account updater | ✅ *"atualização automática do status dos cartões pelas principais bandeiras"* | ⚠️ `[NÃO VERIFICADO]` | ✅ **Account Updater incluído sem custo adicional** na maioria dos planos (2026) |
| **Período de carência** | ⚠️ Construir no unibot (assinatura fica "vencida"; a decisão de suspender é do unibot) | ⚠️ Construir no unibot | ⚠️ Construir no unibot | ⚠️ Configurável no Stripe, mas a suspensão do serviço é sempre do unibot |
| **Negativação (extra)** | ✅ Serasa: R$ 9,90 por cobrança negativada | ❌ | ❌ | ❌ |
| **Avaliação** | ✅ **Muito forte para o custo** — retry e régua prontos | ✅ Bom | ⚠️ **Menos evidência pública** | ✅ **Melhor tecnicamente** (ML + Account Updater) |

> **Distinção nativo × construir (vale para os quatro):** nenhum gateway suspende o *serviço* do unibot. O que o gateway entrega é **retry de cobrança + avisos de fatura + eventos**. O unibot **sempre** precisa construir: (a) a máquina de estados do tenant (`ativo → em_atraso → carência → suspenso → reativado`), (b) o consumo idempotente de webhooks, (c) a decisão de quando suspender. O que muda entre gateways é **quanto da régua de cobrança vem pronto**, não se o unibot escreve código ou não.

### 2.6 Critério 6 — Webhooks

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Eventos** | ✅ Ricos: cobranças (`PAYMENT_RECEIVED`, `PAYMENT_OVERDUE`, `PAYMENT_DUNNING_REQUESTED`, `PAYMENT_DUNNING_RECEIVED`…), assinaturas (criação/alteração/inativação/remoção), **situação da conta** e **bloqueio de saldo** | ✅ Pagamentos e assinaturas (preapproval) | ✅ Eventos de cobrança, pedido e assinatura, **com escolha de quais eventos e para qual URL** | ✅ **Catálogo mais completo do mercado** |
| **Retry** | ✅ Sim. ⚠️ **Após 15 falhas consecutivas a fila é interrompida**; eventos ficam retidos e são guardados por **14 dias** — depois são **deletados permanentemente** | `[NÃO VERIFICADO]` política de retry | `[NÃO VERIFICADO]` | ✅ Retry com backoff exponencial, vários dias |
| **Idempotência** | ⚠️ **Modelo "at least once"** — a doc instrui explicitamente: *"Persista o `id` recebido e não execute novamente a regra de negócio caso esse identificador já tenha sido processado"* | ⚠️ Mesma necessidade | ⚠️ Mesma necessidade | ✅ Mesma necessidade, com boas ferramentas |
| **Autenticação/verificação** | ⚠️ **Token compartilhado** no header `asaas-access-token` (doc recomenda **não** usar a API key como token) — é mais fraco que assinatura HMAC | `[NÃO VERIFICADO]` | `[NÃO VERIFICADO]` | ✅ **Assinatura HMAC** (`Stripe-Signature`) com timestamp — padrão-ouro |
| **Ferramentas de debug** | ✅ Fila visível, penalidade removível via API/interface | ⚠️ | ⚠️ | ✅ **Stripe CLI** com forward local de eventos |
| **Avaliação** | ✅ Bom, com **2 armadilhas operacionais** (fila pausa em 15 falhas; janela de 14 dias) | ⚠️ Pouca evidência pública | ⚠️ Pouca evidência pública | ✅ **Melhor da categoria** |

> 🔴 **Armadilha operacional do Asaas que o unibot DEVE mitigar:** a fila de webhooks **pausa após 15 falhas consecutivas** e os eventos retidos **expiram em 14 dias**. Para um operador único, uma queda de VPS num fim de semana prolongado pode pausar a fila silenciosamente. **Mitigação obrigatória:** (a) endpoint de webhook que responde `200` **antes** de processar (enfileirar e processar async — o Asaas dá timeout em 10s), (b) alerta ativo se nenhum webhook chegar em X horas, (c) **job de reconciliação diária** que consulta a API de cobranças e corrige divergências independentemente do webhook. Isso vale como rede de segurança para qualquer gateway, mas é **especialmente necessário** no Asaas.

### 2.7 Critério 7 — Qualidade de API/SDK e documentação (backend Node.js)

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **SDK Node.js oficial** | ❌ **Não há SDK oficial.** Existem vários **não oficiais** no npm (`asaas`, `asaas-sdk`, `asaas-node-sdk`, `node-asaas-api`) — qualidade e manutenção variáveis | ✅ SDK oficial Node.js | ✅ SDKs oficiais — **Node.js**, Python, PHP, Ruby, Java, .NET, Go | ✅ SDK oficial Node.js, **excelente**, com tipagem TypeScript de primeira |
| **API** | ✅ REST limpa e previsível (`/v3`), **doc em português**, com `llms.txt` para agentes | ⚠️ REST; doc extensa mas **navegação confusa**, versões sobrepostas | ✅ Core API v5, HTTP Basic, `api.pagar.me/core/v5`. ⚠️ **v4 ainda na doc e sendo descontinuada** — risco de seguir tutorial errado | ✅ **Referência absoluta do setor** |
| **Documentação** | ✅ Boa, em PT-BR, orientada a caso de uso | ⚠️ Irregular | ⚠️ Fragmentada entre v4 e v5 | ✅ **Melhor do mercado** |
| **Avaliação** | ⚠️ API ótima, **SDK é o ponto fraco** | ⚠️ | ⚠️ | ✅ |

> **Mitigação do ponto fraco do Asaas:** a ausência de SDK oficial **não é bloqueante** — a API é REST simples e a arquitetura recomendada na §6 já prevê um **adapter próprio** escrito à mão. Depender de um SDK não oficial de terceiros seria, aliás, *pior* para o lock-in: acopla o unibot a um mantenedor voluntário. **Recomendação: escrever o client HTTP do Asaas à mão** (é pouco código: `fetch` + tipos TypeScript), sem SDK de terceiros.

### 2.8 Critério 8 — Sandbox / homologação

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Sandbox** | ✅ `https://sandbox.asaas.com/api/v3` — testa cobranças, pagamentos, transferências e **webhooks** | ✅ Credenciais de teste | ✅ Chaves de teste | ✅ **Test mode** completo + **Stripe CLI** para forward local de webhooks |
| **Ressalva** | ⚠️ Conta de sandbox é **independente** da de produção — precisa criar outra | ⚠️ Usuários de teste podem ser confusos | ⚠️ `[NÃO VERIFICADO]` maturidade | ✅ Nenhuma |
| **Avaliação** | ✅ Adequado | ✅ | ⚠️ | ✅ **Melhor da categoria** |

### 2.9 Critério 9 — KYC/onboarding do recebedor

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Abertura de conta** | ✅ Rápida e self-service; conta digital PJ gratuita | ✅ Muito rápida (self-service) | ⚠️ **KYC criterioso** — validação do responsável legal pelo CNPJ + análise documental | ⚠️ Onboarding rápido, **mas Pix exige 60 dias de histórico** |
| **Risco de bloqueio** | ⚠️ Existe (é instituição de pagamento; há eventos de bloqueio de saldo por disputa Pix / ordem judicial) — **mas o Asaas expõe webhooks específicos para isso**, o que é maturidade | ⚠️ **Relatos de recusa de transações em assinaturas via API** por análise antifraude — *"quando é identificada alguma inconsistência, a operação não é aprovada"* | 🔴 **Maior risco relatado do grupo** — reclamações públicas sobre dificuldade de aprovação de KYC, bloqueio de contas Stone *"sem fundamentação técnica, notificação prévia ou transparência"* e retenção de valores | ⚠️ Risco padrão Stripe; MCC de SaaS é bem aceito |
| **MCC / SaaS B2B** | ✅ Perfil comum na base (SaaS e serviços recorrentes) | ✅ Aceito | ✅ Aceito | ✅ Aceito |
| **Avaliação** | ✅ **Menor atrito** | ⚠️ Rápido, mas antifraude agressivo em assinaturas | 🔴 **Maior risco** | ⚠️ Bom, exceto o gate do Pix |

> **Por que este critério pesa tanto aqui:** com **um operador único**, não há equipe para brigar com o gateway. Uma conta bloqueada = **100% da receita recorrente parada**, com todos os tenants inadimplentes ao mesmo tempo. Os relatos públicos sobre Pagar.me/Stone (bloqueio sem notificação prévia, retenção de valores) são especialmente preocupantes nesse cenário. Note-se que reclamações em plataformas públicas têm **viés de seleção negativa** — clientes satisfeitos não escrevem — e por isso são tratadas aqui como **sinal direcional**, não como medida de taxa de incidência; ainda assim, o padrão relatado é consistente o bastante para pesar contra.

### 2.10 Critério 10 — Suporte ao desenvolvedor e reputação

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Suporte dev** | ✅ Suporte em PT-BR, doc ativa, changelog público | ⚠️ Suporte massificado, difícil escalar caso técnico | ⚠️ Suporte via Stone; relatos de dificuldade | ⚠️ Excelente em inglês; **suporte em PT-BR mais limitado** |
| **Comunidade BR** | ✅ Grande entre SaaS/PMEs brasileiros | ✅ Enorme | ✅ Boa em e-commerce | ✅ Enorme, mas majoritariamente internacional |
| **Estabilidade** | ✅ Changelog ativo, melhorias de webhook em 2026 | ✅ Grande escala | ✅ Infra Stone | ✅ Referência global |
| **Avaliação** | ✅ **Melhor ajuste ao operador BR solo** | ⚠️ | ⚠️ | ⚠️ Ótimo, mas em inglês |

### 2.11 Critério 11 — Custos ocultos

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Mensalidade** | ✅ **Zero** — conta e manutenção gratuitas | ✅ Zero | ✅ Zero (plano Essencial) | ✅ **"Sem taxas de configuração ou mensalidades"** |
| **Taxa por transação recusada** | ✅ **Não cobra** — modelo "pague apenas por cobrança recebida" | `[NÃO VERIFICADO]` | `[NÃO VERIFICADO]` | ✅ Não cobra por recusa |
| **Saque/transferência** | ⚠️ Pix PJ: **30 grátis/mês, depois R$ 2,00**; TED **R$ 5,00** | `[NÃO VERIFICADO]` | ⚠️ **R$ 3,67 por saque** `[NÃO VERIFICADO — fonte de terceiros]` | `[NÃO VERIFICADO]` para BR |
| **Chargeback** | `[NÃO VERIFICADO]` | `[NÃO VERIFICADO]` | `[NÃO VERIFICADO]` | 🔴 **R$ 55,00** de tarifa + **R$ 55,00** de refutação; Smart Disputes cobra **30% do valor contestado** em casos vencidos |
| **Notificações** | ⚠️ **WhatsApp R$ 0,55/notificação**; pacote SMS+e-mail **R$ 0,99** — ⚠️ *a régua de dunning por WhatsApp tem custo por mensagem* | `[NÃO VERIFICADO]` | `[NÃO VERIFICADO]` | Incluído |
| **Billing fee** | ✅ Nenhuma taxa extra sobre recorrência | ✅ Nenhuma | ✅ Nenhuma | 🔴 **+0,7% sobre o volume de Billing** — *taxa adicional só por usar assinaturas* |
| **Nota fiscal** | ✅ R$ 0,49/nota (opcional — útil para PME B2B) | ❌ | ❌ | ❌ |
| **Avaliação** | ✅ **Mais transparente** | ⚠️ Pouca informação pública | ⚠️ Pouca informação pública | 🔴 **Billing 0,7% + chargeback R$ 55 são pesados** |

> 🔴 **A taxa de Billing do Stripe (0,7%) é frequentemente esquecida** em comparações. Ela incide **sobre todo o volume de assinaturas**, *além* da taxa de cartão de 3,99% + R$ 0,39. O custo real de uma assinatura no cartão via Stripe é, portanto, **4,69% + R$ 0,39** — o mais caro do grupo por larga margem.

### 2.12 Critério 12 — Tokenização e portabilidade (anti-lock-in)

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Tokenização** | ✅ `creditCardToken` — token retornado após transação aprovada, ou via `POST /v3/creditCard/tokenizeCreditCard`. ⚠️ **Em produção exige aprovação do gerente de conta** | ✅ Card token | ✅ Card token | ✅ PaymentMethod / SetupIntent |
| **Escopo do token** | ⚠️ *"O token pertence ao cliente para o qual foi criado e não pode ser utilizado em cobranças de outro cliente"* | ⚠️ Escopo da conta | ⚠️ Escopo da conta | ⚠️ Escopo da conta |
| **Portabilidade dos cartões** | 🔴 **Não portável** — migrar exige recoletar cartões | 🔴 Não portável | 🔴 Não portável | ⚠️ **Melhor caso do grupo:** o Stripe tem processo documentado de **migração assistida de dados de cartão PCI** para sair (e entrar) |
| **Avaliação** | 🔴 Lock-in real | 🔴 Lock-in real | 🔴 Lock-in real | ⚠️ Menos pior |

> 🔴 **Verdade incômoda que a §6 precisa endereçar:** **nenhum** gateway brasileiro torna os cartões tokenizados portáveis na prática. A abstração de billing da §6 reduz o custo de *código* de uma migração, mas **não elimina o custo de negócio**: trocar de gateway significa **pedir a cada tenant que recadastre o cartão** — com perda de conversão. O Stripe é o único com processo formal de migração PCI, mas isso não o salva do problema do Pix. **Conclusão prática: o lock-in do cofre de cartão é inevitável e deve ser gerido pelo mix Pix/cartão, não só por arquitetura.** Quanto mais tenants pagarem por Pix, menor o custo de uma eventual migração.

### 2.13 Critério 13 — PCI-DSS

| | **Asaas** | **Mercado Pago** | **Pagar.me** | **Stripe** |
|---|---|---|---|---|
| **Caminho hospedado (menor escopo PCI)** | ✅ Redirecionar para `invoiceUrl` — **dados do cartão nunca tocam o servidor do unibot** | ✅ Checkout Pro | ✅ Checkout hospedado | ✅ **Stripe Checkout / Elements** (SAQ-A) |
| **Caminho transparente** | ⚠️ Enviar `creditCard` + `creditCardHolderInfo` via API — **coloca o unibot no escopo PCI (SAQ-D)** | ⚠️ Checkout Transparente (SDK client-side tokeniza — reduz escopo) | ⚠️ Idem | ✅ Elements tokeniza no client — **escopo mínimo com UI própria** |
| **Melhor experiência com escopo mínimo** | ⚠️ Hospedado é seguro, mas a UX é um redirect | ✅ SDK JS tokeniza no browser | ✅ SDK JS tokeniza no browser | ✅ **Melhor do grupo** — Elements dá UI embutida com SAQ-A |
| **Avaliação** | ⚠️ Adequado **se usar o caminho hospedado** | ✅ | ✅ | ✅ **Melhor** |

> **Decisão derivada, independente do gateway escolhido:** o unibot **NÃO deve trafegar dados de cartão pelo próprio backend**. Para um operador único, entrar no escopo **PCI-DSS SAQ-D** é um custo de conformidade e um risco de segurança injustificáveis. **Regra de arquitetura: usar sempre checkout hospedado ou tokenização client-side.** Isso é um requisito não-funcional que deve ser registrado independentemente de Q6.

---

## §3. Análise individual

### 3.1 Asaas

**Pontos fortes**
- **Pix com taxa FIXA (R$ 1,99)**, não percentual — vantagem estrutural crescente com o ticket. No ticket do unibot (R$ 500–700), equivale a 0,28%–0,40% vs. ~1% dos concorrentes.
- **Dunning nativo, documentado e granular**: 3 tentativas no dia do vencimento (8h/14h/20h), 3 no dia seguinte, mais 2 a cada 24h. É a política de retry mais explicitamente documentada do grupo — **FR-71 sai quase de graça**.
- **Suite completa em um só lugar**: cartão + Pix + boleto + conta digital PJ + nota fiscal (R$ 0,49) + negativação Serasa. Para uma operação solo, **um fornecedor em vez de três**.
- **Zero mensalidade, zero taxa sobre recusa** ("pague apenas por cobrança recebida") — alinhado ao risco de um SaaS em arranque.
- **Webhooks maduros** com eventos de assinatura, dunning, situação da conta e **bloqueio de saldo** — este último é um sinal de maturidade operacional raro.
- **Documentação em PT-BR** orientada a caso de uso, com `llms.txt`.
- **Onboarding self-service rápido**, menor atrito de KYC do grupo.
- Cobrança avulsa de implantação e boleto para empresas (venda assistida) são triviais.

**Pontos fracos**
- ❌ **Sem SDK Node.js oficial.** Só wrappers comunitários de qualidade variável.
- ⚠️ **Fila de webhooks pausa após 15 falhas consecutivas**; eventos retidos expiram em **14 dias**. Exige disciplina de engenharia (ver mitigação em §2.6).
- ⚠️ **Autenticação de webhook por token compartilhado** (`asaas-access-token`), não HMAC assinado — mais fraco que o padrão Stripe.
- ⚠️ **Tokenização em produção exige aprovação do gerente de conta** — não é auto-serviço; precisa ser solicitado com antecedência.
- ⚠️ **Régua de aviso por WhatsApp tem custo por mensagem** (R$ 0,55) — o dunning "grátis" não é 100% grátis se usar o canal mais eficaz. *(Ironia registrada: o unibot é uma plataforma de WhatsApp; pode mandar a própria régua pelos seus canais e pagar só o custo de WABA.)*
- ⚠️ Menor porte que Stripe/Stone — menos garantias de SLA.

**Risco principal**
> **Dependência de um fornecedor de médio porte para 100% da receita, com cofre de cartão não portável e webhooks que podem pausar silenciosamente.** Mitigável com: abstração de billing (§6), job de reconciliação diária, monitoramento ativo da fila de webhooks, e incentivo comercial ao Pix (que reduz o custo de uma eventual migração).

### 3.2 Mercado Pago

**Pontos fortes**
- **Marca fortíssima no Brasil** — reconhecimento do pagador reduz fricção na conversão.
- **Pix a 0,99%** com liquidação imediata; nenhum gate de acesso.
- **Assinaturas nativas** (`preapproval`) com retry de recusa e **atualização automática de status de cartão pelas bandeiras**.
- **SDK Node.js oficial.**
- Onboarding instantâneo, self-service.
- **Flexibilidade de fluxo de caixa**: escolher receber na hora (mais caro) ou em 30 dias (mais barato) é uma alavanca real.

**Pontos fracos**
- ⚠️ **Taxa de cartão é a mais alta ou quase** (3,98%–4,98%) e **depende do prazo de liberação** — não há número único, o que dificulta previsão de margem.
- 🔴 **Antifraude agressivo em assinaturas via API.** Há relatos públicos de recusas sistemáticas: *"quando é identificada alguma inconsistência, a operação não é aprovada"*. Para um SaaS B2B recorrente, recusas falso-positivas viram churn involuntário — **exatamente o que FR-71 quer evitar**.
- ⚠️ **Documentação irregular**, com versões sobrepostas e navegação confusa.
- ⚠️ **Não foi possível confirmar as taxas na fonte oficial** (página retornou 403) — todos os percentuais vêm de agregadores terceiros.
- ⚠️ Suporte massificado, difícil escalar um caso técnico sem gerente de conta.
- ⚠️ Sem conta digital PJ com nota fiscal integrada no mesmo nível do Asaas.

**Risco principal**
> **Recusa por antifraude em cobranças recorrentes legítimas.** O motor de risco do Mercado Pago é calibrado para marketplace/varejo, não para SaaS B2B recorrente de ticket alto. Um falso positivo recorrente no cartão de um tenant gera churn involuntário que o dunning não resolve, porque a transação nem chega a ser tentada de verdade.

### 3.3 Pagar.me

**Pontos fortes**
- **Infraestrutura Stone** — porte e robustez de processadora de verdade.
- **SDKs oficiais em 7 linguagens, incluindo Node.js** — o melhor suporte oficial de SDK entre os players nacionais.
- **Core API v5** bem estruturada, com Plans + Subscriptions nativos.
- **Webhooks configuráveis por evento e por URL** — granularidade útil.
- **Pix a 0,99%** (plano Essencial).
- **Plano Flex com taxas customizadas** — há espaço real de negociação por volume.
- Repasse **"a partir de 1 dia"** no melhor caso.

**Pontos fracos**
- 🔴 **KYC criterioso e com relatos públicos de atrito** — dificuldade de aprovação de cadastro, validação do responsável legal, análise documental demorada.
- 🔴 **Relatos de bloqueio de conta e retenção de valores** no ecossistema Stone *"sem fundamentação técnica, notificação prévia ou transparência"*.
- ⚠️ **Retenção de até 15 dias para novos vendedores** — problema de fluxo de caixa justamente no arranque, quando ele mais dói.
- ⚠️ **Documentação fragmentada entre v4 e v5**, com a v4 ainda visível e sendo descontinuada — risco real de implementar contra a versão errada.
- ⚠️ **Dunning nativo não confirmado** em documentação oficial — o critério de maior peso é justamente o menos evidenciado.
- ⚠️ **Taxa de cartão 4,19%**, das mais altas.
- ⚠️ Orientado a e-commerce; SaaS recorrente de operador único não é o perfil central de cliente.

**Risco principal**
> **Bloqueio/retenção de conta com pouca transparência e sem canal de escalação para um cliente pequeno.** O unibot seria um cliente minúsculo dentro da Stone — sem gerente de conta e sem poder de barganha no momento em que mais precisaria.

### 3.4 Stripe

**Pontos fortes**
- **A melhor API, SDK e documentação do mercado**, sem discussão. SDK Node.js com TypeScript de primeira linha.
- **Billing/Subscriptions é a referência da indústria** — trials, proração, upgrades/downgrades, faturas.
- **Dunning tecnicamente superior**: **Smart Retries** com modelo de ML que escolhe horários ótimos + **Account Updater gratuito** (recupera ~15% das falhas silenciosamente).
- **Webhooks com assinatura HMAC** (`Stripe-Signature`) — padrão-ouro de verificação.
- **Stripe CLI** com forward local de webhooks — a melhor DX de desenvolvimento e testes do grupo.
- **Portal do cliente pronto** — resolve FR-57 (histórico de faturas) praticamente sem código.
- **Único com processo formal de migração de dados PCI** — menor lock-in do cofre de cartão.

**Pontos fracos**
- 🔴 **PIX É INVITE-ONLY.** *"Pix for Brazilian Stripe accounts is currently invite-only."* Exige conta em boa situação e **mínimo de 60 dias processando pagamentos** antes de poder pedir acesso, com avaliação de risco discricionária. **Isto viola FR-54 no MVP.**
- 🔴 **Pix recorrente só existe desde 2026-04-22** (mandate options / Pix Automático) — **5 meses de maturidade**. Imaturo demais para ser a fundação do billing.
- 🔴 **Mais caro do grupo**: 3,99% + R$ 0,39 **mais 0,7% de Billing** = efetivamente **4,69% + R$ 0,39** em assinaturas de cartão.
- 🔴 **Chargeback R$ 55,00** + R$ 55,00 de refutação + Smart Disputes a 30% do valor contestado.
- ⚠️ **Suporte em PT-BR limitado**; doc majoritariamente em inglês.
- ⚠️ Complexidade de câmbio/fiscal e menor integração com o ecossistema financeiro BR (sem conta PJ, sem NF, sem boleto competitivo).

**Risco principal**
> **Não atender FR-54 no prazo.** O gate de 60 dias do Pix é incompatível com a meta de primeiros pagantes em ~30 dias. Adotar Stripe significaria lançar só com cartão e **torcer** por aprovação do Pix depois — transformando um requisito obrigatório do PRD em aposta com terceiro.

---

## §4. Simulação de custo

### 4.1 Premissas

- Mix: **60% do faturamento em cartão de crédito** / **40% em Pix** (conforme enunciado).
- **1 cobrança por tenant por mês** (assinatura mensal, tier puro, sem excedente).
- Interpretação do mix: **60% dos tenants pagam com cartão, 40% pagam com Pix** — isso importa porque as taxas do Asaas têm componente **fixo por transação**, e o número de transações muda o resultado.
- Cartão **à vista** (assinatura mensal não é parcelada).
- **Taxas padrão, não promocionais.** As promoções de 3 meses do Asaas (Pix/boleto a R$ 0,99 e cartão a 1,99%) foram **deliberadamente ignoradas** — servem só ao arranque e não devem sustentar uma decisão estrutural.
- Mercado Pago: usado **3,98%** (liberação em 30 dias, a taxa mais favorável). Se Bruno quiser receber na hora, some ~1 p.p.
- Stripe: incluída a taxa de **Billing de 0,7%** sobre o volume de assinaturas. *(Cenário hipotético — o Pix do Stripe é invite-only; simulado apenas para comparação.)*
- Pagar.me: Pix a **0,99%** (pagar.me/ofertas), cartão a **4,19%**.
- ❗ Não incluídos: custos de notificação (Asaas WhatsApp R$ 0,55), saques, chargebacks, nota fiscal. São variáveis de operação, não de tabela.

### 4.2 Fórmulas

```
Nº de tenants no cartão  = N × 0,60
Nº de tenants no Pix     = N × 0,40
MRR cartão               = N × 0,60 × ticket
MRR Pix                  = N × 0,40 × ticket

Asaas        = MRR_cartão × 2,99% + (nº cartão × R$ 0,49) + (nº Pix × R$ 1,99)
Mercado Pago = MRR_cartão × 3,98% + MRR_Pix × 0,99%
Pagar.me     = MRR_cartão × 4,19% + MRR_Pix × 0,99%
Stripe       = MRR_cartão × 3,99% + (nº cartão × R$ 0,39) + MRR_Pix × 1,19% + MRR_total × 0,70%
```

---

### 4.3 Cenário (a) — 10 tenants × R$ 500/mês · MRR R$ 5.000

**Composição:** 6 tenants no cartão (R$ 3.000) · 4 tenants no Pix (R$ 2.000)

| Gateway | Conta | **Custo/mês** | % do MRR |
|---|---|---|---|
| **Asaas** | `3.000 × 2,99% = 89,70` + `6 × 0,49 = 2,94` + `4 × 1,99 = 7,96` | **R$ 100,60** | **2,01%** |
| **Mercado Pago** | `3.000 × 3,98% = 119,40` + `2.000 × 0,99% = 19,80` | **R$ 139,20** | 2,78% |
| **Pagar.me** | `3.000 × 4,19% = 125,70` + `2.000 × 0,99% = 19,80` | **R$ 145,50** | 2,91% |
| **Stripe** | `3.000 × 3,99% = 119,70` + `6 × 0,39 = 2,34` + `2.000 × 1,19% = 23,80` + `5.000 × 0,70% = 35,00` | **R$ 180,84** | 3,62% |

🏆 **Asaas** — economia de **R$ 38,60/mês** vs. Mercado Pago e **R$ 80,24/mês** vs. Stripe.

---

### 4.4 Cenário (b) — 30 tenants × R$ 700/mês · MRR R$ 21.000 ⭐

**Composição:** 18 tenants no cartão (R$ 12.600) · 12 tenants no Pix (R$ 8.400)

| Gateway | Conta | **Custo/mês** | % do MRR | **Custo/ano** |
|---|---|---|---|---|
| **Asaas** | `12.600 × 2,99% = 376,74` + `18 × 0,49 = 8,82` + `12 × 1,99 = 23,88` | **R$ 409,44** | **1,95%** | **R$ 4.913,28** |
| **Mercado Pago** | `12.600 × 3,98% = 501,48` + `8.400 × 0,99% = 83,16` | **R$ 584,64** | 2,78% | R$ 7.015,68 |
| **Pagar.me** | `12.600 × 4,19% = 527,94` + `8.400 × 0,99% = 83,16` | **R$ 611,10** | 2,91% | R$ 7.333,20 |
| **Stripe** | `12.600 × 3,99% = 502,74` + `18 × 0,39 = 7,02` + `8.400 × 1,19% = 99,96` + `21.000 × 0,70% = 147,00` | **R$ 756,72** | 3,60% | R$ 9.080,64 |

🏆 **Asaas — R$ 409,44/mês (1,95% do MRR).**
Economia anual: **R$ 2.102** vs. Mercado Pago · **R$ 2.420** vs. Pagar.me · **R$ 4.167** vs. Stripe.

---

### 4.5 Cenário (c) — 100 tenants × R$ 700/mês · MRR R$ 70.000

**Composição:** 60 tenants no cartão (R$ 42.000) · 40 tenants no Pix (R$ 28.000)

| Gateway | Conta | **Custo/mês** | % do MRR | **Custo/ano** |
|---|---|---|---|---|
| **Asaas** | `42.000 × 2,99% = 1.255,80` + `60 × 0,49 = 29,40` + `40 × 1,99 = 79,60` | **R$ 1.364,80** | **1,95%** | **R$ 16.377,60** |
| **Mercado Pago** | `42.000 × 3,98% = 1.671,60` + `28.000 × 0,99% = 277,20` | **R$ 1.948,80** | 2,78% | R$ 23.385,60 |
| **Pagar.me** | `42.000 × 4,19% = 1.759,80` + `28.000 × 0,99% = 277,20` | **R$ 2.037,00** | 2,91% | R$ 24.444,00 |
| **Stripe** | `42.000 × 3,99% = 1.675,80` + `60 × 0,39 = 23,40` + `28.000 × 1,19% = 333,20` + `70.000 × 0,70% = 490,00` | **R$ 2.522,40** | 3,60% | R$ 30.268,80 |

🏆 **Asaas** — economia anual de **R$ 7.008** vs. Mercado Pago e **R$ 13.891** vs. Stripe.

---

### 4.6 Leitura dos resultados

| | (a) 10×R$500 | (b) 30×R$700 | (c) 100×R$700 |
|---|---|---|---|
| **Asaas** | **R$ 100,60** (2,01%) | **R$ 409,44** (1,95%) | **R$ 1.364,80** (1,95%) |
| Mercado Pago | R$ 139,20 (2,78%) | R$ 584,64 (2,78%) | R$ 1.948,80 (2,78%) |
| Pagar.me | R$ 145,50 (2,91%) | R$ 611,10 (2,91%) | R$ 2.037,00 (2,91%) |
| Stripe | R$ 180,84 (3,62%) | R$ 756,72 (3,60%) | R$ 2.522,40 (3,60%) |

**O que a simulação mostra:**

1. **O Asaas vence nos três cenários**, com margem crescente em valor absoluto. A vantagem é de **~0,8 p.p. do MRR** contra o segundo colocado e **~1,65 p.p.** contra o Stripe.
2. **A vantagem do Asaas vem quase inteiramente do Pix fixo.** No cartão, 2,99% vs. 3,98% já ajuda; mas no Pix, R$ 1,99 fixo num ticket de R$ 700 é **0,28%** contra 0,99%–1,19% — uma diferença de **3,5× a 4×**. Quanto maior o ticket e quanto mais Pix no mix, maior a vantagem.
3. **O Stripe é o mais caro**, e a razão é a soma frequentemente esquecida: 3,99% de cartão **+ 0,7% de Billing** sobre todo o volume recorrente.
4. **⚠️ Honestidade sobre a magnitude:** no cenário (b), a diferença entre o melhor e o pior é de **R$ 347/mês**. Isso é **real, mas não é o fator decisivo**. Um dia de trabalho de Bruno construindo dunning que o gateway já entrega pronto custa mais do que um ano dessa diferença. **A economia de taxa é o desempate, não o argumento principal** — o argumento principal está na §5.

---

## §5. Recomendação

### 🏆 PRIMÁRIO: **Asaas**

**Recomendação decisiva: adotar o Asaas como gateway de pagamento do unibot, e iniciar a implementação da Feature 9 sobre ele.** Q6 pode ser fechada.

#### Os três motivos, em ordem de peso

**1. É o único candidato que atende FR-53 e FR-54 sem gate, sem espera e sem aposta.**
Cartão recorrente e Pix, ambos nativos, ambos disponíveis no dia 1, num só fornecedor e numa só integração. O Stripe — tecnicamente o melhor produto do grupo — **está eliminado por FR-54**: seu Pix é *invite-only* e exige 60 dias de histórico de processamento antes de sequer poder ser solicitado, com aprovação discricionária. Isso é incompatível com a meta de **primeiros pagantes em ~30 dias**. Não se constrói o billing de um SaaS apostando que um terceiro vai liberar um requisito obrigatório depois.

**2. Entrega mais do FR-71 (dunning) pronto, por unidade de esforço de Bruno.**
Este é o requisito de maior peso, e o Asaas é o único que **documenta a política de retry em detalhe operacional**: 3 tentativas no dia do vencimento (8h, 14h, 20h), 3 no dia seguinte, mais 2 a cada 24h — 5 dias de recuperação automática — somadas a notificações nativas ao cliente e a eventos de webhook específicos de dunning (`PAYMENT_DUNNING_REQUESTED` / `PAYMENT_DUNNING_RECEIVED`). Para um **operador único**, isso é a diferença entre consumir webhooks e escrever um motor de cobrança. *(O Stripe tem dunning tecnicamente superior — Smart Retries com ML e Account Updater gratuito — mas dunning excelente sobre um Pix que não existe não resolve o problema do unibot.)*

**3. É o menor risco operacional para uma operação de uma pessoa só.**
Onboarding self-service rápido, KYC de menor atrito do grupo, zero mensalidade, zero taxa sobre cobrança recusada, documentação em português, e uma suite que cobre cartão + Pix + boleto + conta PJ + nota fiscal + negativação — **um fornecedor em vez de três**. Contrasta diretamente com o **Pagar.me/Stone**, que concentra os relatos públicos mais graves de bloqueio de conta e retenção de valores sem notificação prévia — risco inaceitável quando 100% da receita recorrente passa por ali e não há equipe para escalar o caso. E é mais barato nos três cenários (§4), o que serve de **desempate**, não de argumento principal.

#### Condições em que esta recomendação mudaria

| Se… | Então… |
|---|---|
| O Asaas **não liberar tokenização em produção** rapidamente (exige aprovação do gerente de conta) | 🔴 **Bloqueante.** Confirmar **antes** de escrever código (§7, item 1). Sem tokenização não há cartão recorrente. Se negado → **Mercado Pago**. |
| Ocorrerem **bloqueios de conta ou instabilidade material** do Asaas em produção | Ativar a alternativa (§5.2). A abstração da §6 existe exatamente para isso. |
| O unibot passar a vender **fora do Brasil** ou em múltiplas moedas | 🔄 **Migrar para Stripe.** É o cenário em que o Stripe ganha de forma inequívoca. |
| O **Pix do Stripe** deixar de ser invite-only **e** o Pix recorrente amadurecer (≥12 meses de produção estável) | Reavaliar, **mas apenas se o volume justificar** — a 0,7% de Billing + 3,99%, o Stripe continuaria ~85% mais caro. |
| O ticket médio **cair muito** (ex.: tier de entrada a R$ 100–150) | Reavaliar: o Pix fixo de R$ 1,99 vira **1,3%–2,0%** e a vantagem estrutural do Asaas **evapora**. Nesse caso, Mercado Pago (0,99%) passa à frente no Pix. |
| O volume crescer a ponto de dar poder de barganha (MRR ≫ R$ 100k) | Negociar tabela com Asaas **e** cotar o plano **Flex do Pagar.me** ("taxas customizadas"). |

### 🥈 ALTERNATIVA: **Mercado Pago**

**Por que é a alternativa e não o Pagar.me nem o Stripe:**
- ✅ Atende **FR-53 e FR-54 sem gate** (diferente do Stripe) — o requisito eliminatório.
- ✅ **SDK Node.js oficial** — cobre justamente a maior fraqueza do Asaas.
- ✅ Assinaturas nativas com retry e **atualização automática de cartão pelas bandeiras**.
- ✅ Onboarding instantâneo — **pode ser ativado em dias**, não semanas, se o primário falhar.
- ✅ Menor risco de KYC/bloqueio que o Pagar.me.
- ⚠️ **Ressalva conhecida:** antifraude agressivo em assinaturas via API pode gerar recusas falso-positivas — monitorar taxa de aprovação desde o primeiro dia.

**Por que o Pagar.me ficou em terceiro:** tecnicamente competente (SDK Node oficial, Core API v5 bem feita), mas concentra o **maior risco de KYC/bloqueio** do grupo, retém até 15 dias de novos vendedores, tem documentação fragmentada entre v4 e v5, e é o único cujo **dunning nativo não pôde ser confirmado** — justamente o critério de maior peso.

**Por que o Stripe ficou em quarto apesar de ser o melhor produto:** ele vence em API, SDK, documentação, webhooks, dunning e portabilidade. Perde por **dois fatos de negócio que anulam a superioridade técnica**: Pix invite-only (viola FR-54 no prazo) e custo ~85% maior. **O Stripe é a escolha certa para o unibot em outro momento** — se e quando houver venda internacional.

---

## §6. Plano de mitigação de lock-in

### 6.1 Princípio

O unibot já trata **canal (WhatsApp)** e **IA** como camadas plugáveis por trás de um contrato único (`addendum.md` §1.1 e §1.2). **Billing recebe o mesmo tratamento.** A escolha do Asaas é uma escolha de *implementação*, não de *arquitetura*.

### 6.2 Contrato único de billing

Espelhando `enviar / receber / status` da camada de canal, a camada de billing expõe um **port** estável:

```ts
// src/billing/ports/gateway.ts — contrato estável. Nenhum código de produto importa o Asaas.
export interface BillingGateway {
  readonly nome: 'asaas' | 'mercadopago' | 'pagarme' | 'stripe';

  // Recebedor (tenant como pagador)
  criarPagador(dados: DadosPagador): Promise<PagadorId>;
  atualizarPagador(id: PagadorId, dados: Partial<DadosPagador>): Promise<void>;

  // Assinatura (FR-52, FR-53, FR-54, FR-51)
  criarAssinatura(input: {
    pagador: PagadorId;
    tier: TierId;
    valor: Centavos;
    meio: 'CARTAO' | 'PIX' | 'BOLETO';
    primeiroVencimento: Date;   // FR-51: suporta trial via data futura
  }): Promise<AssinaturaId>;
  alterarAssinatura(id: AssinaturaId, mudanca: MudancaDeTier): Promise<void>;
  cancelarAssinatura(id: AssinaturaId): Promise<void>;

  // Cobrança avulsa (implantação da venda assistida)
  criarCobrancaAvulsa(input: CobrancaAvulsa): Promise<CobrancaId>;

  // Faturas (FR-57)
  listarFaturas(pagador: PagadorId, page: Paginacao): Promise<Fatura[]>;
  obterLinkFatura(id: CobrancaId): Promise<URL>;

  // Cartão — sempre via checkout hospedado ou token client-side (nunca PAN no backend)
  iniciarCadastroDeCartao(pagador: PagadorId): Promise<{ urlOuClientToken: string }>;

  // Webhooks (FR-58, FR-71)
  verificarAssinaturaWebhook(req: RequisicaoCrua): boolean;
  traduzirEvento(payloadCru: unknown): EventoBilling | null;  // ← tradução para o domínio
}
```

### 6.3 Eventos canônicos de domínio

O ponto crítico: **o unibot nunca reage a um evento do Asaas — reage a um evento do unibot.** `traduzirEvento` é a única fronteira que conhece vocabulário de gateway.

```ts
export type EventoBilling =
  | { tipo: 'PAGAMENTO_CONFIRMADO';  assinatura: AssinaturaId; fatura: CobrancaId; em: Date }
  | { tipo: 'PAGAMENTO_FALHOU';      assinatura: AssinaturaId; tentativa: number; motivo?: string }
  | { tipo: 'FATURA_VENCEU';         assinatura: AssinaturaId; diasEmAtraso: number }
  | { tipo: 'FATURA_A_VENCER';       assinatura: AssinaturaId; diasParaVencer: number }  // FR-71
  | { tipo: 'ASSINATURA_CANCELADA';  assinatura: AssinaturaId }
  | { tipo: 'REEMBOLSO' | 'CHARGEBACK'; fatura: CobrancaId };
```

Mapeamento de exemplo (Asaas → domínio): `PAYMENT_CONFIRMED`/`PAYMENT_RECEIVED` → `PAGAMENTO_CONFIRMADO` · `PAYMENT_OVERDUE` → `FATURA_VENCEU` · `PAYMENT_DUNNING_REQUESTED` → `PAGAMENTO_FALHOU`.

### 6.4 Máquina de estados do tenant — **do unibot, não do gateway**

Esta é a peça que **nenhum gateway entrega** e que, por isso mesmo, deve viver inteiramente no unibot. É também o que garante que trocar de gateway **não mexa em regra de negócio**:

```
ATIVO ──FATURA_A_VENCER──▶ ATIVO (dispara aviso — FR-71)
ATIVO ──FATURA_VENCEU───▶ EM_ATRASO (gateway faz retry; unibot avisa)
EM_ATRASO ──PAGAMENTO_CONFIRMADO──▶ ATIVO (reativação — FR-58)
EM_ATRASO ──(carência N dias expirada)──▶ SUSPENSO (FR-58)
SUSPENSO ──PAGAMENTO_CONFIRMADO──▶ ATIVO
```

O **período de carência (N dias) é parâmetro de configuração do unibot**, não do gateway. Trocar de gateway não altera a política comercial.

### 6.5 Regras de disciplina arquitetural

| # | Regra | Por quê |
|---|---|---|
| 1 | **Nenhum ID de gateway vaza para o domínio.** Persistir `gateway_ref` numa coluna dedicada, ligada à entidade do unibot | Migração não reescreve o domínio |
| 2 | **Nenhum `import` de gateway fora de `src/billing/adapters/<nome>/`** — garantir com regra de lint/dependency-cruiser no CI | A abstração só existe se for **executável**, não documental |
| 3 | **Testes de contrato por adapter** (espelhando a camada de canal — `addendum.md` §1.1) | Um segundo adapter é validável sem ir a produção |
| 4 | **Adapter `fake` em memória** implementando `BillingGateway` | Testar toda a régua de dunning e a máquina de estados **sem tocar em gateway** |
| 5 | **Webhook responde `200` antes de processar** (enfileirar + processar async) | Asaas dá timeout em 10s e **pausa a fila após 15 falhas** |
| 6 | **Job de reconciliação diária** — varre assinaturas via API e corrige divergências | Rede de segurança contra webhook perdido; **crítico** dada a janela de 14 dias do Asaas |
| 7 | **Nunca trafegar PAN pelo backend** — só checkout hospedado ou token client-side | Mantém o unibot fora do escopo PCI-DSS SAQ-D |
| 8 | **Persistir histórico de faturas no banco do unibot**, não só consultar o gateway | FR-57 continua funcionando após migração; sem isso o histórico fica refém |
| 9 | **Escrever o client HTTP do Asaas à mão** (sem SDK comunitário) | Evita acoplar o unibot a um mantenedor voluntário; a API REST é simples |

### 6.6 O limite honesto desta mitigação

> ⚠️ **A abstração resolve o custo de código. Não resolve o custo de negócio.**
> **Cartões tokenizados não são portáveis entre gateways brasileiros.** Uma migração real exigiria **pedir a cada tenant que recadastre o cartão**, com perda de conversão e atrito comercial. As mitigações de verdade para isso são três, e nenhuma é arquitetural:
> 1. **Incentivar Pix comercialmente** (ex.: desconto no anual via Pix). Pix não tem cofre — tenants no Pix migram **sem atrito nenhum**. Isso é, simultaneamente, o mais barato (§4) e o menos aprisionado.
> 2. **Concentrar renovações** para que uma eventual migração aconteça em janela previsível.
> 3. Aceitar que a migração de cartões é **evento de negócio**, planejado com comunicação ao cliente — não um deploy.

---

## §7. Lacunas de evidência — confirmar direto com o gateway

> Itens **1 a 3 são bloqueantes**: confirmar **antes** de escrever código de billing.

### 🔴 Bloqueantes

| # | Lacuna | Por quê é bloqueante | Como confirmar |
|---|---|---|---|
| **1** | **Tokenização em produção no Asaas exige aprovação do gerente de conta.** Prazo, critérios e probabilidade de aprovação para um SaaS B2B novo são desconhecidos | **Sem tokenização não há cartão recorrente → FR-53 não existe.** É o único item que pode derrubar a recomendação inteira | Abrir conta e **solicitar formalmente** a liberação de tokenização em produção **antes** de iniciar a Feature 9 |
| **2** | **Taxas reais aplicadas à conta do unibot.** As taxas públicas são de tabela; a taxa efetiva depende de MCC, perfil e análise | Toda a §4 assume tabela pública. Uma taxa diferente muda a simulação | Pedir **proposta formal por escrito** ao Asaas, com MCC de SaaS B2B recorrente declarado |
| **3** | **Existe régua de aviso ANTES do vencimento no Asaas** (FR-71 pede aviso preventivo), e ela é configurável? | O retry pós-vencimento está confirmado; o **aviso pré-vencimento não** | Perguntar ao suporte; testar em sandbox. **Plano B já disponível:** o unibot é uma plataforma de WhatsApp — pode disparar a própria régua preventiva pelos seus canais, sem pagar R$ 0,55/notificação ao Asaas |

### 🟡 Importantes

| # | Lacuna | Como confirmar |
|---|---|---|
| 4 | **Política de chargeback do Asaas** — custo e processo de contestação `[NÃO VERIFICADO]` | Suporte/contrato. Risco baixo em SaaS B2B recorrente, mas deve ser conhecido |
| 5 | **Asaas tem Account Updater** (atualização de cartão expirado pelas bandeiras)? | Se **não**, o unibot precisa de régua própria de "cartão vencendo em 30 dias". Mercado Pago e Stripe têm; **é a maior lacuna funcional do Asaas** |
| 6 | **SLA e uptime histórico** do Asaas — não publicado | Pedir SLA formal; monitorar status page |
| 7 | **Confirmar Pix recorrente nativo em assinatura Asaas** — se cada ciclo gera nova cobrança Pix com QR novo e como o tenant é avisado | Testar em **sandbox** — é o fluxo de 40% dos tenants na simulação |
| 8 | **Comportamento real da fila de webhooks** sob falha — validar as 15 falhas / 14 dias e testar a remoção de penalidade via API | Teste deliberado em sandbox, derrubando o endpoint |
| 9 | **Taxas oficiais do Mercado Pago para checkout online** — página oficial retornou **403**; todos os números da §2.1 vêm de agregadores terceiros | Consultar o painel do Mercado Pago com conta criada, antes de acionar a alternativa |
| 10 | **Dunning nativo do Pagar.me** — não confirmado em doc oficial | Só relevante se Asaas e Mercado Pago caírem |
| 11 | **Boleto no Mercado Pago e Pagar.me** — taxas `[NÃO VERIFICADO]` | Relevante para a venda assistida a empresas que exigem boleto |
| 12 | **Prazo de repasse do Stripe no Brasil** (D+X) — não especificado publicamente | Só relevante no cenário de internacionalização |

### 🔵 Viés a controlar

| # | Item |
|---|---|
| 13 | **O Concorrente A usa Asaas.** Registrado como inteligência competitiva. A recomendação da §5 se sustenta em FR-54 (Pix sem gate), FR-71 (dunning documentado) e risco operacional — **nenhum desses argumentos depende da escolha do concorrente**. O risco a evitar é o inverso: descartar o Asaas *por* ser do concorrente seria igualmente irracional. Gateway não é diferencial competitivo; **é infraestrutura**. |
| 14 | **Reclamações públicas têm viés de seleção negativa** (clientes satisfeitos não escrevem). Os relatos sobre Pagar.me/Stone foram tratados como **sinal direcional**, não como taxa de incidência. Se o Pagar.me voltar à mesa, buscar referências diretas de SaaS B2B brasileiros em operação. |

---

## Resumo executivo

| | |
|---|---|
| **Decisão** | **Asaas** como gateway primário do unibot |
| **Alternativa** | **Mercado Pago** (ativável em dias se o primário falhar) |
| **Eliminados** | **Stripe** — Pix invite-only viola FR-54 no prazo, e é ~85% mais caro · **Pagar.me** — maior risco de KYC/bloqueio, dunning não confirmado |
| **Custo estimado** | (a) R$ 100,60/mês · **(b) R$ 409,44/mês (1,95% do MRR)** · (c) R$ 1.364,80/mês |
| **Principal ressalva** | 🔴 A **tokenização de cartão em produção do Asaas exige aprovação do gerente de conta**. Confirmar **antes** de codar — sem ela, FR-53 não existe |
| **Próximo passo** | Abrir conta Asaas (produção + sandbox), solicitar tokenização e proposta de taxas por escrito, validar itens 1–3 da §7 |
| **Q6** | ✅ Pronta para fechamento |

---

*Documento produzido em 2026-09-12. Todas as taxas foram consultadas nesta data em fontes oficiais, salvo onde marcado `[NÃO VERIFICADO]` ou onde a fonte de terceiros está explicitada. **Taxas de gateway mudam sem aviso — revalidar antes de qualquer compromisso contratual.***

### Fontes consultadas (2026-09-12)

- Asaas — Preços e taxas: https://www.asaas.com/precos-e-taxas
- Asaas — Documentação API: https://docs.asaas.com/ (webhooks, assinaturas, cartão de crédito, sandbox)
- Asaas — Blog, cobrança recorrente: https://blog.asaas.com/cobranca-recorrente-no-asaas/
- Stripe — Preços Brasil: https://stripe.com/br/pricing
- Stripe — Pix invite-only: https://support.stripe.com/questions/how-to-enable-pix-as-a-payment-method-in-brazil
- Stripe — Changelog Pix recorrente (2026-04-22): https://docs.stripe.com/changelog/dahlia/2026-04-22/pix-recurring-payments-support
- Pagar.me — Ofertas: https://www.pagar.me/ofertas
- Pagar.me — Documentação: https://docs.pagar.me/
- Mercado Pago — Assinaturas/preapproval: https://www.mercadopago.com.br/developers/pt/docs/subscriptions/overview
- Mercado Pago — taxas via agregadores (⚠️ página oficial 403): calculadoradetaxas.com.br, sellsync.ai
- plataformas públicas de reclamação — relatos KYC/bloqueio Pagar.me/Stone e recusas Mercado Pago (⚠️ viés de seleção negativa)

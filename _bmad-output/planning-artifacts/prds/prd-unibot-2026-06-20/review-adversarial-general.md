---
title: "Review Adversarial — PRD unibot"
type: adversarial-review
target: prd-unibot-2026-06-20/prd.md + addendum.md
reviewer: revisor adversarial (cético)
date: 2026-07-09
---

# Review Adversarial — PRD unibot

> 🔒 **Versão anonimizada para versionamento.** As referências competitivas deste documento foram
> substituídas por designações neutras (Concorrente A, B, C). Os achados técnicos, datas e versões
> são preservados integralmente — apenas a identificação das fontes foi removida.
> A versão com as fontes nomeadas é mantida fora do controle de versão.


> **Postura:** este documento ataca. Não há elogios. Cada achado assume má-fé do redator e busca a contradição, a suposição não-examinada e o furo de escopo que vai explodir em epics/arquitetura. Onde o PRD se defende com uma mitigação, eu questiono se a mitigação é real ou se é só uma frase.

**Veredito adversarial:** o PRD promete estabilidade de nível operadora (uptime ≥99,5%, "não derruba a sessão") sobre três drivers dos quais dois são APIs não oficiais estruturalmente instáveis, com um operador único, e ao mesmo tempo confessa que o escopo (10 features / 71 FRs) não cabe no prazo de 30 dias — as três âncoras do produto (estabilidade, margem sob tiers puros, primeiros pagantes em 30 dias) estão em contradição direta entre si e nenhuma tem número verificável que a sustente.

---

## Sumário de severidade

| Severidade | Contagem |
|---|---|
| **Critical** | 4 |
| **High** | 7 |
| **Medium** | 6 |
| **Low** | 3 |
| **Total** | **20** |

---

## CRITICAL

### C1 — "Uptime ≥99,5%" sobre APIs não oficiais é uma promessa que a arquitetura não pode cumprir, e o PRD sabe disso
**Local:** §3 (métrica-âncora, "Uptime das conexões WhatsApp ≥ 99,5%"), NFR-1, addendum §1.1.

**Problema:** O PRD ergue 99,5% como **"âncora do diferencial"** (§3) e NFR-1 diz que "Atualizações da plataforma **não podem derrubar a sessão**". Mas o addendum §1.1 lista dois dos três drivers como não oficiais — **Evolution API** e **Baileys** — e o próprio addendum §3 admite: *"Baileys quebra sessão a cada update; Passkey/`@lid`/LID quebram vínculo e entregam msg errada"*. Uma conexão não oficial que o WhatsApp pode banir a qualquer momento não tem SLA; ninguém controla o denominador. 99,5% de uptime = ~3,6h de downtime/mês **por conexão**. Um único ciclo de ban+reconexão via QR estoura isso. O PRD nem define o que "uptime da conexão" significa: uptime do driver? do número do cliente? agregado do tenant? Sem denominador, a métrica-âncora é não-testável — e sendo testável, é provavelmente inatingível.

Adicionalmente, NFR-1 é **auto-contraditório com a existência do FR-63**: se a plataforma "não derruba a sessão", por que precisa de normalização de identidade contra "mudanças de endereçamento do WhatsApp" (FR-63) e gestão de reconexão automática (FR-4)? Porque a sessão *cai* — a promessa de NFR-1 é retórica de marketing colada num requisito.

**Correção sugerida:** (a) Segmentar o SLA: uptime ≥99,5% **apenas para o driver WABA oficial**; para drivers não oficiais, prometer "best-effort com fallback" e um alvo separado e honesto (ex.: ≥97% ou "reconexão automática < X min em Y% dos casos"). (b) Definir formalmente o denominador do uptime (por conexão, janela de medição, o que conta como downtime, quem exclui manutenção). (c) Reescrever NFR-1: "atualizações da *plataforma* não derrubam a sessão" é defensável; "a sessão nunca cai" não é — separar as duas.

---

### C2 — "Tiers puros sem excedente" + custo variável de IA/WABA = margem negativa garantida no pior tenant, e a "mitigação" é circular
**Local:** §3 (contra-métrica + "Margem de contribuição por tenant: Positiva"), §8 (3º bullet), addendum §1.4, R2.

**Problema:** O modelo é **tiers puros — "Sem cobrança variável de excedente"** (addendum §1.4). Mas dois custos do unibot são estritamente variáveis e fora do seu controle: (1) tokens de IA (OpenAI/Gemini/etc.) e (2) **preço por conversa do WABA cobrado pela Meta** (addendum §1.1: "preço por conversa da Meta"). O PRD reconhece em §3 que o custo de IA "pressiona a margem sob tiers puros" e afirma que a mitigação é "enforcement de cotas". Isso é **circular e insuficiente**:

- A cota protege o **teto**, não o **custo dentro da cota**. Se a cota de IA do plano Essencial for generosa o bastante para vender, ela já pode custar mais que a mensalidade quando totalmente consumida. Se for apertada o bastante para garantir margem, o produto perde para o Concorrente A na feature de IA.
- O **custo WABA por conversa nem aparece como dimensão de gating** (FR-52 lista atendentes, canais, tokens de IA, volume de mensagens — mas "volume de mensagens de campanha" ≠ conversas WABA faturáveis iniciadas pela empresa). Um tenant no tier baixo que dispara campanhas via WABA gera custo Meta que o preço fixo não cobre.
- "Margem positiva por tenant" (§3) é afirmada como **meta**, não demonstrada. Com todos os números de preço/cota em `[A DEFINIR]` (§11.1), a afirmação de margem positiva é **fé, não requisito** — e é justamente o número que decide se o negócio existe.

**Correção sugerida:** (a) Acrescentar **"conversas WABA faturáveis / mês"** como 5ª dimensão de cota (ou fundir explicitamente com "volume de mensagens"), senão o custo Meta fura o tier. (b) Exigir, como **gate de arquitetura/go-to-market**, uma planilha de margem por tier no pior caso (cota 100% consumida em IA + WABA) — mover Q3 de "não bloqueia arquitetura" para bloqueante da decisão de pricing. (c) Reconhecer no PRD que "tiers puros" transferem 100% do risco de custo variável para o unibot; ou aceitar isso conscientemente, ou reintroduzir um teto rígido de degradação (não só "aviso") que corte o custo antes da margem virar negativa.

---

### C3 — 30 dias vs. 10 features / 71 FRs: o PRD admite a contradição e a "resolve" empurrando o problema para epics
**Local:** R9, §11.1 ("⚠️ Tensão de escopo"), §3 ("Primeiros clientes pagantes: 10 — prazo a definir"), Q1b.

**Problema:** O PRD **admite explicitamente** que "30 dias não comportam as 10 features / 71 FRs" (§11.1) e responde com uma "fatia vendável". Mas a fatia vendável descrita **ainda é enorme** e internamente contém os itens mais difíceis do produto:

- Fatia de 30 dias = Feature 1 (camada de canal com WABA + 1 driver não oficial + **fallback transparente**) + Feature 2 (inbox: filas, transferência, histórico) + Features 7–9 completas (multitenancy, isolamento, RBAC, onboarding self-service, provisionamento automático, **billing cartão+Pix com gateway ainda não escolhido**, trial, inadimplência) + dashboard.
- Isso não é um MVP enxuto; é praticamente **metade das features**, incluindo as duas mais arriscadas (camada de canal com fallback + billing multitenant). O fallback transparente entre WABA e um driver não oficial (semânticas de sessão, template, janela de 24h e identidade completamente diferentes) é sozinho um projeto de semanas.
- **Contradição de metas:** §3 lista "Primeiros clientes pagantes: 10" mas com "prazo a definir"; §11.1 e Q1 dizem "primeiros pagantes em ~30 dias". São **10 pagantes em 30 dias** ou **primeiro pagante em 30 dias**? O PRD usa as duas leituras. Dez PMEs pagantes em 30 dias exige um funil de vendas que o PRD não menciona sequer existir (sem CAC, sem canal de aquisição, sem pipeline).
- O gateway de pagamento — pré-requisito absoluto de "cobrar" — está **em aberto (Q6)**. Não dá para prometer billing em 30 dias com o gateway indefinido.

**Correção sugerida:** (a) Desambiguar a meta: escrever "**1º cliente pagante** em 30 dias" e mover "10 pagantes" para um horizonte com data (ex.: 90 dias). (b) Cortar o fallback transparente da fatia de 30 dias — entregar **um único driver** primeiro (provavelmente Evolution *ou* WABA, não os dois com fallback) e adicionar o segundo driver + fallback logo depois. Fallback multi-driver é o item que mais ameaça o prazo. (c) Resolver Q6 (gateway) **antes** de comprometer a data de billing. (d) Adicionar seção de aquisição: sem plano de como 10 PMEs chegam e convertem, a meta de negócio é aspiração.

---

### C4 — Concentração no operador único (Bruno) é risco existencial, e a mitigação declarada não mitiga
**Local:** R4, §3 (contra-métrica "Carga operacional sobre o Bruno"), §8 (1º bullet), FR-42/45/46.

**Problema:** Bruno é **operador, superadmin, suporte, vendas (10 pagantes/30d) e provavelmente dev**. R4 nomeia "Concentração no operador" mas classifica como risco comum e mitiga com "onboarding self-service (Feature 8) + centro de operação consolidado (FR-46)". Isso é falha de raciocínio:

- Self-service reduz o trabalho **por tenant**, mas não remove o **single point of failure humano**. Se Bruno adoece na semana do lançamento, o negócio para: não há on-call secundário, sem runbook, sem cobertura. O PRD não menciona bus-factor, backup humano, nem SLA de resposta a incidente.
- A promessa de estabilidade (NFR-1/2/3) exige **resposta operacional a incidentes de canal** — bans de número, reconexões, restrição de conta Meta (R7). Esses incidentes chegam a qualquer hora e **não são self-service**; exigem o Bruno. Com dezenas de tenants, "~40 chamados/dia" (citado em R8 sobre o Concorrente A) sobre um humano só é matematicamente inviável.
- §3 lista como contra-métrica "carga operacional crescendo linearmente com nº de tenants" mas o único mecanismo contra isso é "automação operacional (ver NFRs)" — os NFRs falam de observabilidade e resiliência, não de **eliminar a intervenção humana obrigatória** em bans/reconexões, que é justamente o que não escala.

**Correção sugerida:** (a) Adicionar NFR de operação: runbooks para os N incidentes de canal mais comuns, automação de reconexão sem intervenção humana como requisito duro (não "health-check", mas "auto-remediação para X% dos incidentes"), e um limite explícito de tenants suportáveis por operador antes de contratar. (b) Definir um plano de continuidade mínimo (bus-factor ≥ 2 ou runbook + acesso de emergência). (c) Reconhecer que a meta de escala ("dezenas/centenas de tenants", FR-42/NFR-5) é incompatível com operador único e nomear o ponto em que a contratação vira pré-requisito, não opção.

---

## HIGH

### H1 — Onboarding "self-service em minutos" colide com a verificação obrigatória da Meta para WABA
**Local:** §3 ("Tempo de onboarding até primeiro atendimento real < 1 dia (ideal: minutos)"), FR-49 ("provisionamento automático do tenant, em minutos"), FR-50, FR-3.

**Problema:** Conectar o canal **WABA oficial** exige verificação de Business Manager pela Meta, registro/aprovação do número e, para mensagens iniciadas pela empresa, **aprovação de template** (o próprio FR-65 reconhece isso). Esse processo é da Meta, leva de horas a dias, e **não é self-service em minutos** — está fora do controle do unibot. O PRD promete "primeiro atendimento real < 1 dia (ideal minutos)" e "provisionamento em minutos" sem distinguir:
- provisionamento do **tenant** (a plataforma — pode ser em minutos), de
- provisionamento do **canal WABA** (depende da Meta — não é).

Para o driver não oficial (QR Code) o onboarding em minutos é plausível; para WABA, não. O PRD funde os dois e cria uma expectativa que a jornada oficial vai furar no primeiro cliente.

**Correção sugerida:** Separar a métrica: "provisionamento do tenant < X min" (controlável) vs. "ativação do canal WABA" (dependente de aprovação Meta, fora do SLA). Documentar explicitamente na Feature 8 que a rota WABA tem etapa de verificação externa; oferecer o driver não oficial como caminho de "primeiro atendimento em minutos" e o WABA como rota de estabilidade com onboarding assíncrono.

---

### H2 — "3 drivers estáveis" contradiz o próprio argumento de venda ("poucas conexões estáveis > muitas frágeis")
**Local:** §2 (bullet "Camada de canal unificada e estável": *"contrato único sobre poucas conexões realmente estáveis"* e *"O Concorrente A respondeu à instabilidade somando 7 APIs frágeis... Poucas conexões estáveis > muitas frágeis"*), addendum §1.1 + nota de risco.

**Problema:** O diferencial nº 1 de posicionamento é retórico: "poucas conexões **realmente estáveis**" contra as "7 APIs frágeis" do Concorrente A. Mas o addendum §1.1 adota **três** drivers, dois deles não oficiais (Evolution + Baileys) — exatamente a categoria "frágil" que o discurso condena — e a própria nota de risco do addendum admite: *"A expansão para 2 não oficiais aumenta a superfície de manutenção"*. O brief original previa **1** não oficial; subiu para 2 "por redundância de fallback". Então o produto está fazendo uma versão menor do que critica no concorrente (somar APIs não oficiais para compensar instabilidade), enquanto vende o oposto. "Baileys" (biblioteca) não é uma "conexão realmente estável" por nenhuma definição — o addendum §3 diz que ela "quebra sessão a cada update".

**Correção sugerida:** Alinhar discurso e realidade. Ou (a) reduzir para WABA + 1 não oficial no MVP (como o brief) e manter a narrativa "poucas e estáveis"; ou (b) manter os 3 mas reescrever o posicionamento honestamente: "1 conexão oficial estável + fallbacks não oficiais redundantes" — parar de afirmar que as conexões não oficiais são "realmente estáveis". A afirmação atual é falsa em relação à arquitetura escolhida.

---

### H3 — Fallback "transparente/automático" entre WABA e não oficial ignora que os canais não são intercambiáveis
**Local:** FR-5 ("fallback automático entre provedores... preservando a continuidade do atendimento"), §2, R1, addendum §1.1 ("fallback transparente entre drivers").

**Problema:** FR-5 trata os provedores como plugues intercambiáveis, mas WABA e drivers não oficiais têm **regras semânticas incompatíveis**:
- WABA impõe **janela de 24h + template aprovado** para mensagem iniciada pela empresa (FR-64/65). O driver não oficial não.
- O **número** é frequentemente diferente entre WABA e o número conectado por QR — fazer fallback pode significar **falar com o cliente de outro número**, quebrando a continuidade (o oposto do prometido).
- Formatos de mídia, recibos de leitura, presença (FR-7) diferem por provedor ("quando o provedor suportar" — já admite divergência).

"Fallback transparente preservando continuidade" é, portanto, subespecificado a ponto de ser não-implementável como escrito. O que acontece com uma mensagem template-only quando o fallback cai num driver sem template? E quando volta? Há risco real de mensagem entregue pelo número errado — exatamente o bug FR-63 tenta evitar em outra dimensão.

**Correção sugerida:** Especificar a **política de fallback**: (a) fallback só entre drivers que compartilham o mesmo número, ou tratar troca-de-número como evento explícito, não transparente; (b) definir o comportamento quando a janela/template diverge entre origem e destino do fallback; (c) rebaixar "transparente" para "com regras explícitas de compatibilidade" e listar as combinações suportadas. Isto é um furo de escopo que vai explodir na arquitetura da Feature 1.

---

### H4 — FR-71 "sessão única" marcado "opcional por tenant" — requisito com consequência anulada
**Local:** FR-71 ("Sessão única / forçar logout ao autenticar em novo dispositivo (opcional por tenant)").

**Problema:** Um controle de segurança marcado "opcional" não é testável como requisito de segurança — o comportamento padrão não está definido (liga ou desliga por default?), e "opcional" significa que a plataforma precisa suportar **os dois modos**, dobrando o custo sem que o PRD diga qual é o esperado. É requisito vago disfarçado de feature.

**Correção sugerida:** Definir o default (recomendo: ligado para o operador, opcional para tenant) e o critério de aceite verificável ("ao autenticar no dispositivo B, a sessão do dispositivo A é encerrada em < X s").

---

### H5 — NFRs sem limites numéricos = não-testáveis
**Local:** NFR-6 ("Latência baixa"), NFR-5 ("1 a centenas de tenants sem reescrita"), NFR-8 ("backup... testada"), NFR-2 ("detecta degradação antes do cliente"), §3 ("Redução mensurável de TPR/TMA").

**Problema:** Vários NFRs não têm número, logo não têm critério de falha:
- NFR-6 "latência baixa" — quanto? p95 de entrega de mensagem? de atualização de inbox? Sem número, sempre "passa".
- NFR-5 "centenas de tenants sem reescrita" — "sem reescrita" é inauditável; é aspiração, não NFR.
- NFR-8 "rotina testada" — testada com que frequência? RPO/RTO? Sem RPO/RTO, backup não é requisito, é intenção.
- NFR-2 "detecta antes do cliente" — sem definição de tempo de detecção / cobertura de alertas.
- §3 "redução mensurável de TPR/TMA" — mensurável contra qual baseline? "Mensurável" ≠ meta.

**Correção sugerida:** Anexar limites numéricos a cada NFR: NFR-6 → p95 entrega < X ms, inbox update < Y ms; NFR-8 → RPO ≤ X, RTO ≤ Y, teste de restore mensal; NFR-2 → detecção < X min, cobertura de alerta ≥ Y% dos incidentes; §3 → baseline capturado no onboarding + meta de redução %.

---

### H6 — Risco Meta (R7) tem probabilidade alta e mitigação empurrada para o roadmap
**Local:** R7, addendum §3 ("Conta Meta do Concorrente A restringida derrubou clientes; OAuth compartilhado = ponto único de falha"), roadmap item 3 (BSP próprio marcado ⚑).

**Problema:** O PRD reconhece que a conta Meta do **próprio concorrente foi restringida em 2026, derrubando clientes** — ou seja, é um risco **realizado no mercado**, não hipotético. A mitigação estrutural (ser **BSP/Meta Business Partner próprio**, Embedded Signup) está no **roadmap pós-MVP** (item 3). Isso significa que no MVP o unibot depende de uma configuração WABA que carrega o mesmo ponto único de falha que derrubou o Concorrente A. Se a estratégia de onboarding WABA no MVP usar um app/conta Meta compartilhado do unibot, uma restrição da Meta derruba **todos** os tenants simultaneamente — matando a métrica-âncora de uptime de forma correlacionada, não isolada. R1 promete "isolamento de cada provedor como driver substituível", mas isso não isola do risco de **conta/app Meta compartilhado**, que é ortogonal ao driver.

**Correção sugerida:** Decidir no MVP (não no roadmap) o modelo de conta WABA: cada tenant com seu próprio WABA/Business Manager (isolamento de risco Meta, mas onboarding mais pesado — ver H1) vs. app compartilhado do unibot (onboarding leve, risco correlacionado). Documentar a escolha e seu blast radius. Não é decisão adiável: define se um incidente Meta é de 1 tenant ou de todos.

---

### H7 — "Volume de mensagens/contatos" é uma dimensão de cota, mas mistura coisas de custo e risco diferentes
**Local:** FR-52 (dimensão "volume de mensagens/contatos"), §11.1 (tabela separa "Volume de mensagens de campanha/mês" e "Contatos no CRM"), FR-56.

**Problema:** FR-52 define a 4ª dimensão como "volume de mensagens/contatos" — juntando duas coisas que a tabela de §11.1 depois **separa em duas linhas** ("Volume de mensagens de campanha/mês" e "Contatos no CRM"). São 4 dimensões no FR-52 mas **5 linhas de gating** na tabela §11.1. Inconsistência interna. Pior: "mensagens" mistura mensagens de atendimento (baixo custo) com mensagens de campanha WABA (custo Meta por conversa — ver C2). Contar as duas na mesma cota mede a coisa errada para proteção de margem.

**Correção sugerida:** Reconciliar FR-52 com §11.1 (são 4 ou 5 dimensões?). Separar explicitamente "mensagens de atendimento" de "conversas de campanha faturáveis WABA" — só a segunda tem custo Meta variável e precisa de gating de margem.

---

## MEDIUM

### M1 — FR-51 trial: "bloqueio controlado do acesso" não define o destino dos dados/sessão do cliente
**Local:** FR-51, FR-58 (inadimplência).

**Problema:** "Ao fim do trial, conversão ou bloqueio controlado" e FR-58 "suspensão por falta de pagamento" não dizem: as **sessões de WhatsApp continuam conectadas** durante a suspensão? (Se sim, custo de infra num não-pagante; se não, o número do cliente cai — pesadelo de reputação e possível ban.) Os dados são retidos por quanto tempo antes de purgados (cruzar com NFR-4 retenção)? Requisito com "consequência" nominal mas sem comportamento verificável no ativo mais sensível (a conexão).

**Correção sugerida:** Especificar o estado de "suspenso": conexões pausadas graciosamente, janela de retenção de dados antes de purge, e o efeito no número do cliente. Definir critério de aceite.

---

### M2 — FR-63 (identidade robusta) é apresentado como resolvido, mas depende de comportamento não documentado do WhatsApp
**Local:** FR-63, R7, addendum §3.

**Problema:** FR-63 promete "normalizar de forma robusta" nono dígito, `@lid`/LID e "mudanças de endereçamento do WhatsApp" — e o PRD o vende como correção definitiva de um "bug perene do Concorrente A (corrigido 6+ vezes em 2 anos)". Mas se o Concorrente A precisou corrigir **6+ vezes**, é porque a Meta **muda o esquema de endereçamento** periodicamente — é um alvo móvel, não um bug de código a ser "resolvido de uma vez". Apresentar como requisito fechado subestima que isto é manutenção contínua e um vetor de quebra recorrente (o mesmo que R7 admite). FR-63 é honesto como *capacidade*, desonesto como *garantia*.

**Correção sugerida:** Reformular como requisito de **resiliência a mudanças** (camada de identidade isolada + testes de contrato + processo de resposta rápida a mudança de esquema Meta), não como "identidade robusta" fechada. Vincular ao mesmo processo de R7.

---

### M3 — Segurança/LGPD (2FA, mascaramento) empurrados para "logo após", mas trial expõe dados reais no dia 1
**Local:** §11.1 ("Logo após... FRs de governança (2FA, visibilidade de dados)"), FR-67, FR-68, NFR-4.

**Problema:** A fatia vendável de 30 dias **exclui** 2FA (FR-67) e mascaramento de dados sensíveis (FR-68), colocando-os em "logo após". Mas a fatia de 30 dias **inclui** onboarding, inbox e conversas reais — ou seja, dados pessoais de terceiros (LGPD) entram na plataforma **antes** dos controles de acesso que o próprio NFR-4 lista como requisito ("2FA e restrição de visibilidade de dados"). Isso é uma janela de exposição consciente: os primeiros 10 clientes operam sem 2FA e sem mascaramento, sobre dados de terceiros, sob responsabilidade LGPD onde o tenant é controlador e o unibot operador. Se vazar nessa janela, o DPA padrão (NFR-4) não protege o unibot da falha operacional.

**Correção sugerida:** Puxar 2FA (ao menos para o **operador/superadmin**, cuja conta compromete todos os tenants) para dentro da fatia de 30 dias. Mascaramento pode esperar, mas o acesso privilegiado do Bruno não pode ficar sem 2FA no dia 1.

---

### M4 — "Login como" tenant (FR-45) é o maior risco de privacidade e só tem "auditoria" como controle
**Local:** FR-45, NFR-9, NFR-4.

**Problema:** FR-45 permite ao operador "logar como" qualquer tenant e ler todas as conversas (dados de terceiros). O único controle é "com auditoria" (NFR-9). Auditoria é **detecção posterior**, não prevenção. Não há consentimento do tenant, escopo temporal, nem restrição de leitura de conteúdo sensível durante o impersonation. Sob LGPD, com o unibot como operador, acesso irrestrito ao conteúdo de conversas por um humano precisa de base legal e limitação de finalidade — "está auditado" não basta.

**Correção sugerida:** Adicionar controles preventivos: consentimento/notificação do tenant ao iniciar "login como", escopo temporal (sessão expira), aplicação do mascaramento FR-68 **também** ao operador impersonando, e finalidade registrada. Não deixar o controle mais sensível do sistema com só um log.

---

### M5 — "Redução mensurável de TPR/TMA" depende de baseline que o produto não captura
**Local:** §3 (métricas-âncora), FR-59.

**Problema:** A meta "redução mensurável de TPR/TMA nos clientes ativos" pressupõe conhecer o TPR/TMA do cliente **antes** do unibot — mas o cliente vinha de um WhatsApp caótico sem métricas (é a dor descrita em §1). Não há baseline. O dashboard (FR-59) mede TPR/TMA **dentro** do unibot, não a melhoria relativa. A métrica de sucesso é, na prática, inauditável.

**Correção sugerida:** Ou capturar um baseline autodeclarado no onboarding, ou trocar a métrica por uma absoluta e defensável (ex.: "TPR mediano < X min nos tenants ativos" em vez de "redução"). Como está, não serve de critério de sucesso.

---

### M6 — Fora-de-escopo empurra "webhooks/integrações" para o roadmap, mas FR-22 e billing os exigem no MVP
**Local:** §5.2 (fora do MVP: "webhooks bidirecionais, API pública", "n8n, Typebot"), FR-22 ("Webhook/integrações externas ficam fora do MVP"), FR-53/54 (gateway de pagamento).

**Problema:** §5.2 e FR-22 excluem webhooks/integrações do MVP. Mas o **billing** (FR-53 cartão, FR-54 Pix, FR-58 inadimplência) depende inteiramente de **webhooks de entrada do gateway de pagamento** (confirmação de pagamento, falha, chargeback, expiração de Pix). Ou seja, o MVP **precisa** de infraestrutura de webhook de gateway no dia 1 — a exclusão de "webhooks" em §5.2 é ambígua e, se interpretada literalmente, contradiz a fatia vendável que inclui billing. Um leitor de arquitetura pode dimensionar mal a Feature 9.

**Correção sugerida:** Esclarecer em §5.2 que a exclusão é de "webhooks **de tenant / integrações externas de automação**"; webhooks de gateway de pagamento são infraestrutura interna obrigatória do billing e estão **dentro** do MVP.

---

## LOW

### L1 — IDs de FR com buracos e "aposentados" espalhados criam risco de rastreabilidade
**Local:** FR-17 (aposentado), saltos FR-8→FR-63, FR-16→FR-18, FR-34→FR-70, FR-40→FR-66, FR-62 fim, numeração global até 71 com lacunas.

**Problema:** A numeração "global estável" tem FR-17 aposentado ("não reutilizar") e vários FRs de emenda inseridos fora de ordem (FR-63, 64, 65 dentro da Feature 1; FR-66 na Feature 6; FR-70 na Feature 5; FR-71 na Feature 7). Está gerenciado, mas a densidade de exceções aumenta a chance de um FR ser perdido no salto para epics (ex.: alguém conta "71 FRs" mas há lacunas — o número real de requisitos ativos não é 71). O PRD afirma "71 FRs" mas o maior ID é 71 **com** FR-17 morto e lacunas → contagem enganosa.

**Correção sugerida:** Adicionar uma tabela de índice FR→feature→status (ativo/aposentado) e declarar a **contagem real** de FRs ativos, não o maior ID.

---

### L2 — "Delay aleatório anti-banimento" (FR-33) é mitigação folclórica, não garantia
**Local:** FR-33, R5.

**Problema:** FR-33 vende "delay aleatório entre mensagens" como redução de risco de ban em disparo massivo por driver não oficial. Isso é uma heurística amplamente usada mas **sem eficácia garantida** — a Meta bane por padrão comportamental, reclamação de usuário e reputação do número, não só por cadência. O PRD trata como mitigação de R5 sem reconhecer que disparo massivo por API não oficial é **intrinsecamente** violação de ToS do WhatsApp e pode banir o número independentemente do delay.

**Correção sugerida:** Rebaixar a linguagem: "reduz, sem eliminar" o risco; adicionar aviso de que disparo massivo por driver não oficial viola ToS e recomendar campanhas via WABA/template como caminho conforme. Alinhar com o monitoramento de qualidade de número (roadmap item 4).

### L3 — Churn "< 5% após 3º mês" sem coorte definida e sem numerador/denominador
**Local:** §3 (métricas de negócio).

**Problema:** "Churn mensal (após 3º mês) < 5%" não define se é churn de logos ou de receita, nem a coorte. Com ~10 clientes, 1 cancelamento = 10% — a métrica é estatisticamente ruidosa demais para ser meta acionável no estágio do MVP.

**Correção sugerida:** Definir churn (logo vs. receita) e reconhecer que com N pequeno a métrica é indicativa; talvez substituir por retenção absoluta de logos nos primeiros meses.

---

## Padrões transversais (meta-observações)

1. **Marketing coladas em requisitos.** NFR-1 ("não derruba a sessão"), §2 ("conexões realmente estáveis"), FR-63 ("robusta"), FR-33 ("anti-banimento") usam linguagem de pitch onde deveria haver critério de aceite. Todo adjetivo forte sem número é um requisito não-testável.
2. **Três metas-âncora em tensão mútua não reconhecida.** Estabilidade (exige tempo e maturidade), margem sob tiers puros (exige cotas apertadas), e 30 dias (exige cortar escopo) puxam em direções opostas. O PRD reconhece a tensão de *escopo×prazo* (R9) mas não a de *tiers puros×custo variável* como bloqueante nem a de *estabilidade×APIs não oficiais*.
3. **Riscos mais graves têm mitigação diferida para o roadmap** (R7→BSP próprio; segurança→"logo após") enquanto a exposição começa no dia 1.
4. **Números que decidem o negócio estão todos em `[A DEFINIR]`/Q3** e classificados como "não bloqueiam arquitetura" — mas bloqueiam a viabilidade (margem). "Não bloqueia arquitetura" ≠ "não bloqueia a decisão de lançar".

---

## Recomendação de gate

Antes de avançar para epics/arquitetura, exigir: (a) resolução de C2 (planilha de margem por tier no pior caso) e C3 (desambiguar 1 vs. 10 pagantes + resolver gateway Q6); (b) decisão de C4/H6 (modelo de conta WABA + plano de continuidade do operador); (c) reescrita de C1/H5 (SLA segmentado por driver + números nos NFRs). Sem isso, os furos de H3 (fallback) e M6 (webhooks de billing) vão explodir a estimativa da Feature 1 e da Feature 9.

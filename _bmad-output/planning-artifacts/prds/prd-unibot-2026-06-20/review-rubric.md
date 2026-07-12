# PRD Quality Review — unibot (2026-06-20)

## Overall verdict

Este é um PRD forte no que mais importa para um chain-top de estágio LAUNCH: tem uma tese clara e defensável ("estabilidade gerenciada como produto" vs. self-hosted), escopo honesto, e substância *ganha* — cada diferencial e cada requisito de engenharia está ancorado em uma falha concreta do concorrente Z-PRO, não em template. A tensão real (30 dias vs. 71 FRs) é nomeada, não suavizada. O que está em risco é a camada de formalização downstream: **não há glossário e não há nenhuma User Journey**, apesar de o PRD alimentar explicitamente UX → arquitetura → epics e de o produto ter superfícies de UX genuínas (inbox do atendente, flowbuilder visual, onboarding self-service). Somado a alguns NFRs adjetivais sem bound e a um gap de ID (FR-69), isso não compromete a decisão de construir, mas cobra trabalho extra dos workflows a jusante.

## Decision-readiness — strong

Um decisor age sobre este PRD. As decisões estão declaradas *como* decisões, com o que foi abdicado explícito: o addendum §2 lista alternativas rejeitadas com motivo real ("Pricing híbrido → previsibilidade não compensou a fricção de venda"; "1 API não oficial → substituído por 2 para redundância de fallback"). O addendum §1.1 registra o trade-off ativo — "a expansão para 2 não oficiais aumenta a superfície de manutenção — é trade-off aceito por Bruno" — em vez de fingir que a escolha foi grátis. As Questões em aberto (Q3 valores de tier, Q1b recorte da fatia) são genuinamente abertas e marcadas como não-bloqueantes de arquitetura, com dono e condição de revisão; a lista de "Resolvidas (2026-06-20)" mostra que o PRD fechou o que dava para fechar. A tensão de escopo em §11.1 ("30 dias não comportam as 10 features / 71 FRs") e o R9 são o oposto de smoothing.

Nenhum finding material. O único ponto observável — preço 100% `[A DEFINIR]` na tabela de tiers (§11.1) — é honestamente classificado como go-to-market e não bloqueia downstream.

## Substance over theater — strong

Pouca mobília aqui. **Innovation:** os diferenciais de §2 não são novidade alegada — cada um cita uma dor que o Z-PRO "não resolveu ao longo de 2 anos de releases" (ex.: FR-63 identidade de contato "corrigido 6+ vezes"; o incidente Hostinger; a conta Meta restringida em 2026). Isso é diferenciação vinda de Discovery, não de template. **Vision:** a linha de abertura ("operada e gerenciada pelo unibot… o cliente nunca toca em VPS") é específica do produto, não trocável por outro PRD da categoria. **NFR:** a maioria tem threshold específico (uptime ≥99,5%, "updates que não derrubam a sessão", testes de contrato por provedor, backpressure) em vez de "deve ser escalável/seguro".

### Findings
- **low** Persona "Marketing" no teto e rasa (§4) — É a 4ª persona (o limite do rubric), marcada "eventual", e só reaparece difusamente em Campanhas (Feature 5). Não dirige nenhuma decisão que Atendente/Dono já não dirijam. *Fix:* rebaixá-la a menção inline dentro da persona Dono/gestor, ou dar-lhe uma decisão própria (ex.: quem aprova template WABA / cota de campanha).

## Strategic coherence — strong

O PRD tem tese e aposta nela de ponta a ponta. A tese — instabilidade de APIs externas convertida em uptime prometido pela camada de canal — é declarada em §1–2, consolidada em §8 ("a camada de canal é simultaneamente diferencial de produto e mecanismo operacional"), e governa a priorização: a fatia vendável de 30 dias começa pela camada de canal "porque é o diferencial; sem isso não há produto" (§11.1), não pelo que é fácil. As métricas validam a tese em vez de medir atividade: a âncora é *uptime ≥99,5%*, não DAU; e há contra-métricas nomeadas (custo de IA por tenant, tempo de conexão caída, carga operacional sobre o Bruno crescendo linearmente) — exatamente o "ganhar errado" que a tese teme. Scope kind coerente (platform/problem-solving) com lógica de escopo que casa. Sem findings.

## Done-ness clarity — adequate

Os FRs load-bearing têm consequência testável: FR-2 "respeitando o limite de canais do seu plano", FR-5 fallback "preservando a continuidade do atendimento", FR-33 delay aleatório anti-ban, FR-49 provisionamento "em minutos", FR-63 "evitando contatos duplicados e mensagens ao destinatário errado", FR-6 enumera os tipos de mensagem. A NFR-1 — a que mais importa — traz número duro (99,5%) e um critério verificável e incomum ("sem reler QRCode/recriar canal a cada update"). Isso é o suficiente para epics derivarem acceptance na maioria dos casos.

O que baixa a nota: não há seção de Acceptance, e uma minoria relevante de requisitos transversais recorre a adjetivo sem bound — o que o rubric manda flaggar sem piedade. Downstream, a story creation vai ter de inventar os números.

### Findings
- **medium** NFRs transversais sem bound (§7) — NFR-6 "latência baixa na entrega/recebimento" (sem alvo em ms/segundos), NFR-8 "rotina de backup e restauração testada" (sem RPO/RTO), NFR-5 "de 1 a centenas de tenants sem reescrita" (sem ponto de medição). São exatamente os "reasonable performance / user-friendly" que o rubric pede para marcar. *Fix:* dar bound à NFR-6 (ex.: p95 de entrega < Xs) e RPO/RTO à NFR-8; para NFR-5, nomear o teto validado do MVP.
- **low** FRs de tempo-real sem critério (§6) — FR-9 "conversas em tempo real" e FR-4 "health-check contínuo" não dizem quão real nem em que intervalo. Aceitável num PRD de capacidades, mas serão perguntas na story creation. *Fix:* uma linha de acceptance por FR ou remeter explicitamente à NFR-6 depois que ela ganhar bound.

## Scope honesty — strong

Omissões são explícitas, não inferidas. §5.2 "Explicitamente fora do MVP" lista canais, WABA avançado, integrações, IA avançada e white-label, cada um remetido ao roadmap §10. O de-scoping é feito à luz do dia: FR-17 (CSAT) "movido para o roadmap; ID aposentado, não reutilizar" — transparência de ID exemplar. A densidade de itens em aberto é baixa e adequada a um green-light (2 Open Questions não-bloqueantes + lista de resolvidas datada). A tensão de escopo dos 30 dias está sinalizada com ⚠️, não escondida.

### Findings
- **low** Convenção de callout não aplicada / Assumptions Index ausente (§11.1, rodapé) — O rodapé promete "Suposições marcadas com `[ASSUMPTION]` no corpo serão triadas no Finalize", mas só §11.1 carrega a tag e não existe índice de assumptions para o roundtrip; e as tensões usam ⚠️ em vez de `[NOTE FOR PM]`. Não muda a substância (as tensões *estão* lá), mas quebra a mecânica que o Finalize espera. *Fix:* indexar o único `[ASSUMPTION]` ou remover a promessa de triagem; opcionalmente marcar a tensão de 30 dias como `[NOTE FOR PM]`.

## Downstream usability — thin

Esta dimensão pesa mais porque o PRD é chain-top declarado (alimenta UX, arquitetura, epics). O material dos FRs é bom para extração: agrupados por feature, IDs únicos, addendum separa limpamente mecanismo de capacidade, e as cross-refs internas ("ver §5.2", "FR-5", "ver §11.1") resolvem. Mas faltam duas peças que o rubric nomeia explicitamente e que os workflows a jusante vão sentir:

- **Não há Glossário.** Substantivos de domínio de alta carga — *tenant, driver, provedor, camada de canal, fallback, janela WABA, transbordo, fatia vendável, tier, cota* — são usados de forma consistente no corpo, mas sem um glossário a arquitetura/epics não têm âncora para garantir que "canal", "conexão" e "driver" (usados de modo quase intercambiável em FR-2/FR-4 e no addendum §1.1) signifiquem a mesma coisa entrando na modelagem.
- **Continuidade de ID:** ver Mechanical notes — há um gap real (FR-69).

(A ausência de UJs é tratada em Shape fit, mas também degrada esta dimensão: o workflow de UX não terá nenhuma jornada para source-extract.)

### Findings
- **high** Glossário ausente num PRD chain-top (documento inteiro) — 71 FRs e 10 NFRs sem termos definidos; "canal/conexão/driver" oscilam de sentido entre FR-2 (conexões do plano), FR-4 (por conexão) e addendum §1.1 (drivers). *Fix:* adicionar Glossário curto (tenant, driver vs. conexão vs. canal, janela WABA, transbordo, tier, fatia vendável) antes de handoff para arquitetura.
- **low** FR-69 inexistente (§6) — a sequência salta FR-68 → FR-70; FR-69 não aparece em nenhum lugar e não há nota de aposentadoria (ao contrário do FR-17, que é explicitamente aposentado). *Fix:* preencher, ou registrar "FR-69 — aposentado/não usado" como no FR-17.

## Shape fit — adequate

O produto é híbrido: infraestrutura/operação pesada (camada de canal, multitenancy, billing, painel do operador único Bruno) — onde a forma de *capability spec* adotada é a certa e UJs seriam overhead — combinada com superfícies de UX genuínas do lado do tenant: o Atendente "vive na tela do dia a dia" (inbox), o flowbuilder é arrastar-e-soltar, e há onboarding self-service. Para a metade infra/operador, a forma casa bem. Para a metade tenant-facing, o PRD está **sub-formalizado**: nenhuma UJ, apesar de o rubric marcar "consumer/multi-stakeholder B2B/meaningful UX → UJs load-bearing" e de este ser um chain-top que alimenta UX. As personas de §4 carregam contexto, mas persona ≠ jornada. O Fast path justifica leveza, mas não a ausência total de jornada num fluxo que vai gerar telas.

### Findings
- **high** Zero User Journeys num chain-top com UX real (§4 e ausência geral) — o workflow de UX a jusante não tem jornada para extrair para as duas superfícies centrais (inbox do atendente com transbordo bot↔humano; onboarding self-service do dono). *Fix:* adicionar 2–3 UJs com protagonista nomeado — Atendente resolvendo uma conversa com handoff (FR-11/FR-12/FR-15), Dono no onboarding self-service (FR-48–FR-51), e Bruno reagindo a uma queda de conexão (FR-46/FR-8) — para dar tração ao chain UX→epics. Manter capability-spec para a camada de canal/billing.

## Mechanical notes

- **Glossário:** ausente (ver Downstream, high).
- **Continuidade de ID:** FR-1…FR-71 com dois pontos a notar — (1) **FR-69 é gap não explicado** (FR-68 → FR-70); (2) FR-17 aposentado e corretamente sinalizado. FR-63–FR-71 são acréscimos espalhados entre features — aceitável, pois o PRD declara "IDs globais estáveis" como escolha intencional, mas reduz a varredura visual.
- **Assumptions roundtrip:** só §11.1 traz `[ASSUMPTION]`; o rodapé promete triagem no Finalize sem índice — roundtrip incompleto (ver Scope honesty, low).
- **Cross-ref frouxo:** §3 contra-métrica diz "(ver §9, Feature 6 e 9)" — as Features 6 e 9 estão em §6 (Funcionalidades), não em §9 (Riscos). Ambíguo; provável intenção "§6, Features 6 e 9". *Fix:* corrigir para §6.
- **Numeração de Q:** Q1b como sub-questão de Q1 é levemente irregular, mas a lista de resolvidas reconcilia (Q1 timeline resolvida, Q1b ainda aberta) — consistente.
- **Personas:** 4 (Marketing "eventual") — no teto do rubric; ver Substance, low.
- **Seções exigidas** para o estágio/tipo presentes, exceto Glossário e UJs (tratados acima).

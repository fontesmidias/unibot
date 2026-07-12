# Reconciliação Brief → PRD — unibot

> Verificação de cobertura: o que o **Product Brief** (brief.md + addendum.md, 2026-06-20) afirma e que o **PRD** (prd.md + addendum.md) deixou cair ou representou mal. Foco em ideias qualitativas — posicionamento, personas, tom, métricas, restrições. Só o que está no brief; nada inventado.

## Resumo do veredito

O PRD tem **excelente cobertura funcional** (as 10 features/71 FRs traduzem fielmente o escopo do brief, e várias decisões do addendum foram incorporadas com rastreabilidade). As lacunas são quase todas de **camada qualitativa/estratégica** — posicionamento, personas de mercado e visão de longo prazo — que o brief carrega e o PRD, por ser mais operacional, deixou de fora. Nenhuma diverge por contradição; são **omissões**.

---

## Lacunas (o que o brief tem e o PRD não)

### L1 — Jetsales sumiu: o posicionamento perdeu um dos dois polos `[ALTA]`
O brief posiciona o unibot no **meio premium entre dois concorrentes opostos**: o **ZPRO** (amplo, mas self-hosted e frágil) e o **Jetsales** (simples e bem posicionado, mas raso, preso a um único canal). É uma tese de duas pontas — "a amplitude do ZPRO entregue com a simplicidade e estabilidade de um SaaS gerenciado" (Executive Summary; What Makes This Different: "Amplitude do ZPRO + simplicidade do Jetsales").

O PRD colapsou isso em **um único eixo contra o Z-PRO**. O §2 lista "amplitude com simplicidade", mas **nunca cita o Jetsales** nem o polo "simples-porém-raso". Consequência: perde-se o enquadramento de que o unibot também compete por baixo (contra ferramentas simples), não só por cima. Quem ler só o PRD acha que o único concorrente é o Z-PRO.

### L2 — A seção Vision inteira foi descartada `[ALTA]`
O brief fecha com uma **Vision** de 2-3 anos: unibot como "forma **padrão e confiável** de uma PME brasileira atender e vender por mensagem", evoluindo para um "**hub omnichannel gerenciado**", com "a confiabilidade operacional — não uma feature isolada — [virando] a **marca registrada** que sustenta a recorrência e o **boca a boca**".

O PRD **não tem nenhuma seção de visão/norte de longo prazo**. O roadmap (§10) lista features pós-MVP, mas é uma lista tática, não a narrativa de para onde o produto vai nem por quê. Perdas específicas: a ambição "padrão/default do mercado", o conceito de **hub omnichannel gerenciado** como destino, e a ideia de **boca a boca / confiabilidade como marca registrada** como motor de crescimento (palavra "boca a boca" não aparece no PRD).

### L3 — Personas perderam os segmentos-alvo (verticais) `[MÉDIA]`
O brief "Who This Serves" nomeia os **setores-alvo**: "comércio/varejo, serviços, educação, saúde e similares", com equipe de "**1 a ~15 atendentes**". O §4 do PRD descreve bem os **papéis dentro do tenant** (Bruno, dono/gestor, atendente, marketing), mas **omite completamente as verticais** e a faixa de tamanho da equipe. Isso importa para priorização de features, marketing e definição de tiers — e reaparece indiretamente no roadmap (item 6: "clínicas, cursos"), mostrando que os nichos são relevantes mas não estão ancorados nas personas.

### L4 — O "por que agora" (timing/janela de mercado) sumiu `[MÉDIA]`
O brief justifica o timing: "**Agora é o momento**, com a maturação da API Oficial do WhatsApp (WABA) e a IA generativa acessível, que barateia a automação de atendimento" (Executive Summary). O PRD **não registra o why-now**. É um argumento estratégico central (janela de oportunidade) que orienta urgência e é insumo natural de go-to-market; ficou de fora do §1 (Contexto e problema).

### L5 — Nuances qualitativas do posicionamento diluídas `[BAIXA]`
Três framings do brief perderam força ou desapareceram no PRD:
- **"Meio premium"** — o brief nomeia explicitamente a categoria de preço/posição ("O unibot ocupa o meio premium"). O PRD não usa esse enquadramento; fala em amplitude+simplicidade sem ancorar o *premium*.
- **"Estabilidade como produto" / "vantagem invisível, mas sentida"** — o PRD tem "estabilidade como produto" (§2) mas perde a nuance de que é uma vantagem *invisível ao cliente porém percebida no uptime*, que o brief destaca duas vezes (What Makes This Different; e "O cliente nunca vê uma VPS").
- **Operação "idealmente 24/7"** — o brief (Riscos & Dependências) diz que a estabilidade gerenciada exige capacidade de operar "**idealmente 24/7**". O PRD trata operação como produto (NFR-3, R3) mas **não menciona o compromisso/ambição 24/7**, que é uma restrição operacional concreta.

---

## Itens verificados que NÃO são lacuna (cobertos ou intencionalmente evoluídos)

- **Monetização (tiers puros, 4 dimensões, trade-off de margem, mitigação por cotas):** totalmente coberto (§3 contra-métricas, Feature 9, §8, addendum §1.4). ✓
- **Camada de canal unificada + fallback + isolamento de driver + testes de contrato:** coberto e ampliado (Feature 1, NFR-1/3, addendum). ✓
- **Métricas de sucesso (uptime ≥99,5%, onboarding <1 dia, TPR/TMA, 10 pagantes, churn <5%, margem positiva):** transpostas 1:1 no §3. ✓
- **Modelo de negócio (não revende licença; Bruno operador único; multitenant para servir empresas):** coberto (§4, addendum §1.3). ✓
- **Escopo MVP e roadmap parkeado:** cobertos com fidelidade (§5, §10). ✓
- **Divergências intencionais (1→2 APIs não oficiais; 1→4 provedores de IA):** são **atualizações aprovadas por Bruno**, registradas no addendum do PRD com nota de trade-off — não são lacunas, são evolução documentada. ✓
- **Tom/voz:** o brief não especificou; o addendum do PRD (§4) reconhece isso e assume tom executivo B2B. Consistente. ✓

---

## Recomendação

Lacunas L1–L4 merecem correção antes de epics/arquitetura, pois afetam posicionamento e priorização. Sugestão mínima:
1. Reintroduzir o **Jetsales** no §2 como o polo "simples-porém-raso" (L1).
2. Adicionar uma seção curta de **Visão/Norte** ao PRD (ou nota no topo do §10) capturando hub omnichannel gerenciado + confiabilidade como marca/boca a boca (L2).
3. Incluir **verticais-alvo e faixa de 1–15 atendentes** no §4 (L3).
4. Acrescentar o **why-now** (maturação WABA + IA acessível) ao §1 (L4).
5. Opcional: reancorar "meio premium" e o compromisso "24/7" (L5).

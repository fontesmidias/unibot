# Reconciliação — Inteligência competitiva (Z-PRO) × PRD unibot

> Verifica se a análise `analise-concorrente-zpro.md` (destilada do changelog completo do Z-PRO, ~130 releases, jul/2024→jul/2026) foi absorvida pelo `prd.md` + `addendum.md`. Método: rastrear cada implicação (I1–I12) e cada dor estrutural (2.1–2.7) até um artefato concreto do PRD (FR / NFR / risco / diferencial / roadmap), marcando o que ficou órfão ou sub-explorado.

Data: 2026-07-09

---

## 1. Veredito

Cobertura **alta**. Todas as 12 implicações e as 7 dores estruturais têm rastreio para pelo menos um artefato do PRD. A tese central do produto ("competir exatamente onde o Z-PRO tropeçou por 2 anos: conexão estável, operação gerenciada, isolamento robusto, onboarding sem atrito") está fielmente traduzida em FRs, NFRs e riscos. Não há implicação totalmente órfã.

As lacunas abaixo são **de profundidade/segundo grau**, não de ausência: pontos onde a análise revela uma nuance que o PRD trata de forma genérica, mal-categorizada, ou empurra para o roadmap sendo que a dor é de MVP.

---

## 2. Rastreio das implicações (I1–I12)

| # | Implicação | Absorvido em | Status |
|---|---|---|---|
| I1 | Camada de canal + testes de contrato + fallback + updates que não derrubam sessão | Diferencial §2; FR-1, FR-4, FR-5; NFR-1, NFR-3; addendum §1.1 | ✅ Completo |
| I2 | Identidade de contato robusta (nono dígito, @lid/LID) | FR-63; R7; addendum §3 | ✅ Completo (ver ressalva L3) |
| I3 | Template WABA + janela de 24h | FR-64, FR-65 | ✅ Completo |
| I4 | Backpressure, filas server-side, disparo assíncrono | FR-70; NFR-2, NFR-7 | ✅ Completo |
| I5 | SaaS gerenciado elimina dor de VPS | Contexto §1; Diferencial §2 ("Gerenciado, não self-hosted") | ✅ Completo (ver ressalva L5) |
| I6 | Isolamento + RBAC + auditoria + 2FA desde o início | FR-41, FR-47, FR-67; NFR-4, NFR-9 | ✅ Completo |
| I7 | Risco de plataforma Meta | R7; roadmap 3–4 (BSP próprio) | ✅ Completo (ver ressalva L2) |
| I8 | Debounce antes da IA | FR-66 | ✅ Completo |
| I9 | Delay aleatório anti-ban | FR-33; R5 | ✅ Completo |
| I10 | Copiloto de IA (sentimento/urgência/resumo/tradução) | Roadmap item 1 (⚑ tablestakes) | ✅ Endereçado (roadmap) |
| I11 | LGPD/2FA/blur/auditoria desde o início | FR-67, FR-68; NFR-4, NFR-9, NFR-10 | ✅ Completo |
| I12 | Direção do concorrente (Híbrido, omnichannel, marketplaces, API, agendamento) | Roadmap 2–9 | ✅ Endereçado (roadmap) |

## 3. Rastreio das 7 dores estruturais

| Dor | Absorvido em | Status |
|---|---|---|
| 2.1 Conexão não oficial (dor nº 1) | Diferencial §2; Feature 1 inteira; NFR-1/3; R1 | ✅ Núcleo do produto (ver L1) |
| 2.2 Escala derruba o sistema (Redis/filas/memória/banco) | FR-70; NFR-5, NFR-7; roadmap 10 (cluster/S3) | ✅ Completo |
| 2.3 Self-hosted transfere ônus ao cliente (incidente Hostinger) | Diferencial "Gerenciado"; NFR-1/3/8 | ✅ Completo (ver L5) |
| 2.4 Multitenancy/RBAC/segurança retrofit | FR-41, FR-47; NFR-4; addendum §3 | ✅ Completo |
| 2.5 Risco de plataforma Meta (conta ZDG restringida) | R7; roadmap 3–4 | ✅ Completo (ver L2) |
| 2.6 QA/release imaturo (~40 chamados/dia, hotfix diário) | NFR-3 (CI/CD, deploy zero-downtime); R8 | ✅ Completo |
| 2.7 Identidade de contato BR (nono dígito perene) | FR-63 | ✅ Completo (ver L3) |

---

## 4. Lacunas (por ordem de importância)

### L1 — Falha de modo comum entre os dois drivers não oficiais NÃO está nomeada `[IMPORTANTE]`
A análise mostra que mudanças do lado da Meta atingem **todas as bibliotecas não oficiais ao mesmo tempo**, não uma de cada vez: o **Passkey (2026) quebrou a vinculação em TODAS as libs não oficiais**; `@lid`/"LIDs envenenados" atingiram o ecossistema inteiro. O PRD (addendum §1.1) escolhe **Evolution + Baileys** como driver primário e fallback não oficial, e o diferencial §2 vende "fallback automático" e "poucas conexões estáveis". Mas **fallback Evolution↔Baileys não protege contra falha de modo comum** — quando a causa é uma mudança da Meta, ambos caem juntos; o único driver verdadeiramente independente é o **WABA oficial**.

Isso não está registrado como risco nem como premissa de arquitetura. R1 fala de "isolamento de cada provedor como driver substituível" (protege contra bug de *um* provedor), mas não contra o cenário de todos os não oficiais quebrando simultaneamente. **Recomendação:** nomear explicitamente que o WABA oficial é a âncora de resiliência de modo comum, e que a estratégia de fallback assume falhas independentes — condição que a própria análise mostra ser falsa para eventos da Meta. Sem isso, a promessa de uptime ≥99,5% (métrica-âncora) fica exposta ao mesmo evento que derrubou o Z-PRO.

### L2 — Monitoramento de saúde/qualidade do número é roadmap, mas o risco (R5) é de MVP
A análise destaca o **Score de Qualidade Meta por número** (throttle/bloqueio automático de número amarelo/vermelho) como resposta do Z-PRO ao banimento de números. No PRD isso está **apenas no roadmap (item 4)**. Porém **R5 (bloqueio de número) é um risco real desde o dia 1** — o MVP usa dois drivers não oficiais, exatamente os mais suscetíveis a banimento. A mitigação atual de R5 (delay aleatório FR-33 + disparo server-side FR-70 + fallback FR-5) é **reativa**; falta o guarda **proativo** (detectar degradação de reputação do número antes do bloqueio). Candidato a puxar um subconjunto mínimo (ao menos monitorar/alertar sobre saúde do número) para o MVP, ou registrar como risco explicitamente aceito.

### L3 — Passkey está mal-categorizado como problema de identidade, quando é de conexão/vínculo de dispositivo
FR-63 e R7 agrupam **Passkey** junto a nono dígito/@lid/LID como problema de **normalização de identidade de contato**. Mas na análise (2.1) o Passkey quebrou a **vinculação de novos dispositivos** — é um problema de **estabelecimento de sessão/conexão**, não de endereçamento de contato. O Z-PRO precisou de uma **extensão de navegador paliativa** para importar sessão. Nenhum FR trata a hipótese de que **conectar um novo número via driver não oficial simplesmente pare de funcionar**. FR-3 (conexão por QR) e FR-4 (health-check/reconexão) assumem que a vinculação funciona; não há requisito nem risco cobrindo a quebra do próprio mecanismo de vínculo. Órfão parcial — a menção existe, mas endereça a dor errada.

### L4 — Cobrança preventiva / dunning ausente (Z-PRO tem, PRD pula direto para suspensão)
A análise cita billing do Z-PRO com **"cobrança preventiva, bloqueio automático por inadimplência"**. O PRD tem FR-58 (suspensão por falta de pagamento + fluxo de regularização) e FR-55 (enforcement de cota), mas **não há fluxo de dunning** (lembrete/retentativa de cobrança antes do vencimento/bloqueio). Salto direto para suspensão aumenta churn involuntário — atrito evitável num produto cujo diferencial é operação suave. Lacuna menor, mas barata de fechar e diretamente inspirada no concorrente.

### L5 — Robustez de ingestão de webhooks (incidente Hostinger) sem requisito nomeado
O incidente Hostinger (2.3) foi um provedor de VPS **bloqueando webhooks da Meta como se fossem DDoS** — o Z-PRO não tinha controle. Para o unibot o posicionamento "gerenciado" **implica** a solução, mas **nenhum FR/NFR nomeia a capacidade** de ingestão resiliente/redundante de webhooks WABA — que é infraestrutura crítica do canal oficial (a própria âncora de resiliência apontada em L1). Está no roadmap como "webhooks keep-alive" (item 3, WABA avançado), o que o adia. Lacuna menor: vale um NFR ou uma linha em NFR-1/NFR-2 garantindo que a recepção de webhooks do canal oficial é monitorada e resiliente desde o MVP.

---

## 5. Itens verificados como corretamente endereçados (não são lacunas)

- **Disparo exige tela aberta** (limitação arquitetural do Z-PRO) → FR-70 resolve explicitamente. ✅
- **Escala Redis/cluster/S3** → NFR-5/7 + roadmap 10. ✅
- **Copiloto de IA / sentimento / urgência** → roadmap 1, corretamente marcado como tablestake emergente ⚑. ✅
- **Modo Híbrido WABA, marketplaces, agendamento com turmas, API externa/n8n, voz SIP/Wavoip** → roadmap 2–9, coerente com a leitura estratégica de "não competir em amplitude no MVP". ✅
- **~40 chamados/dia / custo de suporte** → R8. ✅
- **DPA / papéis controlador-operador** → NFR-4 (vai além do que a análise pedia). ✅

---

## 6. Conclusão

O PRD absorveu a inteligência competitiva de forma quase completa e disciplinada — as forças do Z-PRO viraram roadmap com priorização honesta, e as dores viraram diferencial/requisito no MVP. As lacunas remanescentes concentram-se num único tema fino porém crítico: **a fragilidade estrutural dos drivers não oficiais é tratada como falha independente, quando a análise prova que é frequentemente de modo comum (eventos da Meta atingem todos de uma vez)**. L1, L2 e L3 são três faces desse mesmo ponto cego e, juntas, tocam a métrica-âncora (uptime ≥99,5%) e o diferencial nº 1. Recomenda-se endereçá-las antes de `bmad-create-epics-and-stories`. L4 e L5 são refinamentos de menor porte.

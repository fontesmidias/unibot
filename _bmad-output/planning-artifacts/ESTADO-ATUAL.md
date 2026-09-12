# Estado atual do projeto — retomada rápida

> Atualizado em **2026-09-12**. Este documento existe para que qualquer sessão nova (sua ou de um agente)
> saiba em 2 minutos onde o projeto está, sem reconstruir o contexto. **Mantenha-o atualizado ao fim de cada fase.**

---

## Onde estamos

| Fase BMad | Artefato | Status |
|---|---|---|
| 1 — Análise | `briefs/brief-unibot-2026-06-20/brief.md` | ✅ pronto |
| 2 — Planning | `prds/prd-unibot-2026-06-20/prd.md` | ✅ **`final`** — 10 features, **79 FRs** (1–82, com 17 aposentado), 11 NFRs, 9 riscos, 2 jornadas |
| 2 — Planning | UX (`bmad-ux`) | ⬅️ **PRÓXIMO PASSO** — nunca rodou; `design-artifacts/` está vazio |
| 3 — Solutioning | `architecture.md` | ⚠️ travado no step 1 desde 12/07 — só o frontmatter de constraints |
| 3 — Solutioning | Épicos e stories | ❌ não existem |
| 4 — Implementação | Código | ❌ nada escrito |

---

## O próximo passo

**Rode `bmad-ux` em contexto novo.** Por que ele vem antes da arquitetura: o padrão visual de referência
(mimetismo do WhatsApp Web) precisa virar especificação antes de o front ser desenhado, senão a
arquitetura é feita duas vezes.

Insumos que o `bmad-ux` deve consumir:
- `prds/prd-unibot-2026-06-20/prd.md` — **§11.2** traz as diretrizes de experiência e embalagem
- `prds/prd-unibot-2026-06-20/addendum.md` — **§4** traz voz e tom já definidos
- o benchmark de UI/UX do **Concorrente C** em `.private-research/` — **fora do Git** (ver §3 e §5 daquele documento)

Depois do UX: `bmad-create-architecture` → `bmad-create-epics-and-stories` → `bmad-check-implementation-readiness` → `bmad-sprint-planning`.

---

## Decisões vigentes (não reabrir sem motivo)

| # | Decisão | Data |
|---|---|---|
| **D1** | **Venda assistida** (demo agendada + implantação guiada e cobrada), não autoatendimento. Feature 8 reescrita | 2026-09-12 |
| **D2** | Sequência: PRD → UX → Arquitetura → Épicos | 2026-09-12 |
| **D4** | Prazo **derivado da estimativa nos épicos**, não fixado antes dela | 2026-09-12 |
| **Q6** | Gateway = **Asaas** (alternativa: Mercado Pago; Stripe eliminado pelo gate de Pix) | 2026-09-12 |
| — | Canal: WABA + Evolution + Baileys · IA: OpenAI/Gemini/Groq/OpenRouter · tiers puros | 2026-06-20 |

Trilha completa em `prds/prd-unibot-2026-06-20/.decision-log.md`.

---

## ⚠️ Ações que dependem do Bruno

1. 🔴 **Confirmar com o Asaas a liberação de tokenização de cartão em produção** — exige aprovação de
   gerente de conta, não é self-service. **Sem ela o FR-53 não existe.** Fazer isso *antes* de escrever
   qualquer código de billing; se negada, virar para Mercado Pago.
2. ⏱️ **Cronometrar a primeira implantação real.** Destrava três coisas: **Q7** (quantos clientes novos
   por mês o operador suporta), **Q8** (preço da implantação) e a validação de **A8** — se a receita de
   implantação cobre o tempo do operador. *Se A8 for falsa, a justificativa econômica da venda assistida cai.*
3. **Q3** — valores e cotas dos tiers. Atenção: se o tier de entrada ficar em R$ 100–150, a vantagem de
   custo do Asaas evapora (o Pix fixo de R$ 1,99 vira 1,3–2,0%) e o Mercado Pago passa à frente.

---

## Riscos que merecem vigilância

- **R4 — operador único.** É o risco mais severo do projeto, **agravado** pela venda assistida: cada demo,
  implantação e renovação passa pelo Bruno. A mitigação que mais importa está fora do produto —
  **limitar deliberadamente quantos clientes novos por mês aceitar**.
- **NFR-2 — falha silenciosa.** Princípio adotado: medir *efeito observável* (evento recebido, mensagem
  confirmada, atendimento com destino válido), nunca estado declarado de componente. Sustentado por
  FR-73, FR-74 e FR-79. É o diferencial de confiabilidade e o que mais distingue o produto do concorrente.

---

## Convenção de privacidade (obrigatória)

Inteligência competitiva **não é versionada** — ver `.gitignore`. Nos documentos que vão ao Git, os
concorrentes são **Concorrente A / B / C**; fatos, datas e versões são preservados, nomes não.
Fornecedores (Asaas, Baileys, Evolution, OpenAI…) **não** são anonimizados — são a stack do produto.

Material completo, com as fontes nomeadas, em `.private-research/` (fora do Git).

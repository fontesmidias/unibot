# unibot

**SaaS multitenant gerenciado de atendimento e automação no WhatsApp para PMEs brasileiras.**

O cliente vende e atende; a estabilidade da conexão, a infraestrutura e os updates são responsabilidade da plataforma. Diferencial central: **camada de canal unificada** (contrato único sobre WABA oficial + APIs não oficiais, com fallback) entregue como serviço gerenciado — o cliente nunca toca em VPS, update ou biblioteca de conexão.

## Estado do projeto

Em fase de planejamento pelo método **BMad**. Este repositório preserva, desde o início, o histórico do que foi pensado e o **porquê** de cada decisão.

## Artefatos de planejamento

Em [`_bmad-output/planning-artifacts/`](_bmad-output/planning-artifacts/):

| Artefato | Descrição |
|---|---|
| `briefs/brief-unibot-2026-06-20/` | Product Brief (modelo de negócio, escopo) |
| `prds/prd-unibot-2026-06-20/prd.md` | **PRD** — 10 features, FRs 1–71, NFRs, riscos |
| `prds/.../addendum.md` | Decisões técnicas (drivers de canal, provedores de IA) |
| `prds/.../analise-concorrente-zpro.md` | Inteligência competitiva do Z-PRO (changelog de ~130 releases) |
| `prds/.../.decision-log.md` | Trilha de decisões e o porquê de cada uma |
| `architecture.md` | Documento de arquitetura (em construção) |

Insumo bruto do concorrente em [`changelog-concorrente/`](changelog-concorrente/).

## Convenções de versionamento

- **Conventional Commits** (`feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`...).
- **ADRs** (Architecture Decision Records) preservam o porquê das decisões técnicas.
- `CHANGELOG.md` versionado a cada release.
- **Segurança:** o `.env` real nunca é versionado (ver `.gitignore`); apenas `.env.example` com placeholders. Segredos via Docker Swarm secrets / secret manager.

## Deploy (previsto na arquitetura)

- Docker local (desenvolvimento)
- Docker Swarm + Traefik (produção, TLS via Traefik)
- Docker Swarm + Certbot na VPS (produção, TLS no host)
- Topologia flexível: co-locado (uma VPS) ou distribuído (VPS por serviço)

---
stepsCompleted: [1]
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-unibot-2026-06-20/prd.md
  - _bmad-output/planning-artifacts/prds/prd-unibot-2026-06-20/addendum.md
  - _bmad-output/planning-artifacts/prds/prd-unibot-2026-06-20/analise-concorrente-zpro.md
  - _bmad-output/planning-artifacts/briefs/brief-unibot-2026-06-20/brief.md
workflowType: 'architecture'
project_name: 'unibot'
user_name: 'Bruno'
date: '2026-07-12'
architecturalConstraints:
  deploymentScenarios:
    - 'Docker local (desenvolvimento)'
    - 'Docker Swarm + Traefik (produção, TLS via Traefik/Let''s Encrypt)'
    - 'Docker Swarm + Certbot na VPS (produção, TLS via certbot no host)'
  requirements:
    - 'Gerar .env com todas as variáveis de ambiente necessárias, respeitando princípios de segurança (secrets fora do código, separação por ambiente, rotação)'
    - 'Definir recursos mínimos e desejáveis por microserviço (CPU/RAM: reservations e limits)'
    - 'Topologia flexível: co-locado (tudo numa VPS) ou distribuído (microserviços em VPS diferentes)'
  direction: 'Arquitetura de microserviços'
  gatewayDecisionOpen: 'Gateway de pagamento BR (cartão+Pix): Asaas / Mercado Pago / Pagar.me / Stripe'
  sellableSliceGoal: 'Primeiros pagantes em ~30 dias: canal estável + inbox + billing/onboarding'
  versionControl:
    - 'Git/GitHub desde o início do projeto (repo inicializado antes do primeiro código)'
    - 'Conventional Commits (feat/fix/chore/docs...) para versionar features e fixes com rastreabilidade'
    - 'Preservar o PORQUÊ das decisões via ADRs (Architecture Decision Records) + manter o .decision-log do PRD e um CHANGELOG versionado'
    - 'SEGURANÇA: .env real jamais versionado (.gitignore) — só .env.example com placeholders; secrets via Docker Swarm secrets / secret manager'
    - 'CI/CD acoplado aos cenários de deploy (Swarm+Traefik / Swarm+Certbot), com deploy zero-downtime (NFR-3)'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

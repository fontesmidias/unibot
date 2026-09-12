# Sumário de Insumos — unibot (referência: Concorrente A · concorrente: Concorrente B)

> 🔒 **Versão anonimizada para versionamento.** As referências competitivas deste documento foram
> substituídas por designações neutras (Concorrente A, B, C). Os achados técnicos, datas e versões
> são preservados integralmente — apenas a identificação das fontes foi removida.
> A versão com as fontes nomeadas é mantida fora do controle de versão.


> Fase 1 da engenharia reversa. Sintetiza os insumos públicos antes da análise.
> Status: **completo** — páginas de vendas + changelog ingeridos.

## Fontes analisadas

- ✅ Página de vendas Concorrente A — (site do Concorrente A) (sistema de referência)
- ✅ Página de vendas Concorrente B — https://(site do Concorrente B)/ (concorrente)
- ✅ Changelog Concorrente A — `(fonte fora do versionamento)` (conversas 21/07–17/10/2025, v3.1.3.0 → v3.1.4.1)

---

## Entidades identificadas

| Entidade | Descrição | Origem |
|----------|-----------|--------|
| Tenant / Conta | Conta isolada de cliente (multi-tenant) | Concorrente A |
| Usuário / Agente | Atendente que responde tickets | ambos |
| Ticket / Conversa | Atendimento individual com um contato | ambos |
| Fila / Setor | Agrupamento de conversas (Vendas, Suporte, Financeiro) | Concorrente A |
| Contato | Registro de CRM com tags e histórico | ambos |
| Canal / Conexão | Número/conta conectada (WhatsApp, IG, etc.) | ambos |
| Flow / Chatbot | Fluxo de automação visual | ambos |
| Oportunidade | Card de Kanban/funil de vendas | Concorrente A |
| Campanha | Disparo em massa segmentado | ambos |

## Ações / Operações principais

- Multi-atendimento simultâneo no mesmo número, com filas e transferência
- Construção visual de chatbots (flowbuilder) com gatilhos e condições
- Disparo em massa com personalização por variáveis e agendamento
- Gestão de CRM: tags, histórico, Kanban/funil de oportunidades
- Roteamento por setor e transferência humano↔bot
- Relatórios: TMA, TME, desempenho por agente
- (Concorrente A) Revenda multi-tenant: criar contas ilimitadas, gateway de pagamento, painel SuperAdmin

## Regras de negócio observadas

- **Concorrente A:** licença anual única, sem limites de usuários/números/mensagens
- **Concorrente A:** revendedor fica com 100% da receita (sem royalties) no plano Revenda
- **Concorrente A:** masterkey dá acesso administrativo sem senha do cliente; Force Logout
- **Concorrente A:** modo coexistência (QR Code) **e** API Oficial (BSP)
- **Concorrente A:** motor Socket otimizado para 1.500+ tickets/dia
- **Concorrente B:** disparo "respeitando práticas do WhatsApp" (anti-ban implícito)

## Integrações e dependências externas

| Categoria | Concorrente A | Concorrente B |
|-----------|------|----------|
| IA | ChatGPT, Claude, Gemini, Dify | IA própria (não especifica) |
| Automação externa | n8n (400+ nós), Typebot, Webhooks | não especificado |
| Canais | WhatsApp, Instagram, Messenger, Telegram, E-mail, Webchat, TikTok, YouTube, Mercado Livre, OLX | WhatsApp apenas |
| WhatsApp | API Oficial (BSP) + QR Code | QR Code (gerador de link/QR) |
| Infra | Ubuntu VPS (DigitalOcean, Vultr, AWS, Hostinger) | SaaS gerenciado |
| Pagamento | Gateway integrado (margem livre) | — |

## Terminologia do sistema (Concorrente A)

- **Super-Admin** — painel central de gestão de revendas
- **Tenants** — contas isoladas de clientes
- **Flowbuilder** — editor visual de fluxos
- **Filas** — conversas por setor
- **Masterkey** — acesso admin sem senha do cliente
- **Force Logout** — saída forçada de usuários
- **White-Label** — identidade visual personalizável (8 temas de login, domínio próprio)

## Planos e preços

| Produto | Plano | Preço | Limites |
|---------|-------|-------|---------|
| Concorrente A | Uso Próprio | R$ 1.997/ano (12x R$ 205,32) | 1 conta, ilimitado interno |
| Concorrente A | Revenda | R$ 2.797/ano (12x R$ 287,57) | contas/servidores ilimitados |
| Concorrente B | — | não divulgado | SaaS por assinatura (presumido) |

- Concorrente A: garantia 7 dias, reembolso integral; infra ~R$ 50-150/mês; 13 idiomas
- Concorrente B: 50k+ usuários ativos, 10M+ msgs/mês, 99,9% uptime

## Público-alvo

- **Concorrente A:** agências, empreendedores, revendedores SaaS, e-commerce, turismo
- **Concorrente B:** educação, varejo, serviços profissionais, saúde

## Posicionamento comparado (insight inicial)

| Eixo | Concorrente A | Concorrente B |
|------|------|----------|
| Modelo | Self-hosted, white-label, licença anual, revenda | SaaS gerenciado por assinatura |
| Amplitude de canais | Muito ampla (10 canais) | WhatsApp puro |
| Foco | Controle/custo/revenda | Simplicidade + IA de qualificação |
| Escala provada | 1.500+ tickets/dia (alegado) | 10M msgs/mês, 99,9% uptime |

## Stack técnica do Concorrente A (confirmada pelo changelog)

| Camada | Tecnologia | Evidência no changelog |
|--------|-----------|------------------------|
| Backend | Node.js (npm + pm2), arquitetura por socket | `npm install`, `pm2 restart`, "motor Socket" |
| Frontend | Vue 2 → **Vue 3** (Quasar, PWA) | migração ago–out/2025, `npx quasar build -m pwa` |
| Banco | Relacional (indexações, performance) | "novas indexações e melhoria de performance no banco" |
| Cache/filas | Redis (5.6 → 6.8) | "consumo excessivo de Redis", `redis-parser` |
| Proxy/TLS | Nginx + Certbot (Let's Encrypt) | `sites-available`, `certbot --nginx` |
| Orquestração | Docker + Portainer (sem k8s) | "recriação do Redis via Portainer" |
| Deploy | VPS Ubuntu, autoinstalador shell, `(usuário de deploy do concorrente)` | `(script de instalação e usuário de deploy do concorrente)` |
| Multi-tenant | Pastas por tenant | `backend/sessions/tenant1/1` |

### Canais/APIs de WhatsApp suportados (camada fragmentada)

Baileys (não oficial), whatsapp-web.js (wwebjs), **Wuzapi/MEOW** (Go/whatsmeow self-hosted), **Evolution API** (self-hosted), **Z-API** e **Uazapi** (cloud pagas), **WABA/Cloud API** (oficial Meta, com templates, catálogo, SIP/ligações).

### Features entregues no período (jul–out/2025)

Superadmin (export empresas, masterkey, limitar canais por tenant), convite/transbordo em atendimento, templates+catálogo WABA, Google Calendar (OAuth2, ligado a oportunidades), Gmail nativo (OAuth2), chatbot interno com ações (criar oportunidade/evento/nota, agendar msg, webhook, SMS, vapi) e condições (igual/contém/regex) + captura de variáveis `{var}`, transbordo humano + ChatGPT, funil/Kanban, SMS (livison), recebimento de ligações (SIP/WAVOIP), reescrita de IA por idioma, campanhas/disparo em massa, avaliação pós-atendimento (TMA/TME), webhooks n8n + Typebot, webchat, presence (digitando/gravando).

## Gaps / incertezas restantes

1. Preços e limites reais do **Concorrente B** (página de planos não lida) — secundário.
2. Modelo de dados detalhado e contratos de API exatos — inferíveis, não capturados via tráfego.
3. Definição final do **unibot**: escopo do MVP e modelo de negócio — ✅ direção definida: **self-hosted/revenda** melhorando custo e onboarding.

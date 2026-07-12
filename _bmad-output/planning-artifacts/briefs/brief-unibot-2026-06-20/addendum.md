# Addendum — Product Brief: unibot

Profundidade que pertence a documentos downstream (PRD, arquitetura) ou que ganhou lugar mas não cabe no corpo enxuto do brief.

## Monetização — opções consideradas

> **Decisão (2026-06-20): Tiers puros.** Bruno priorizou previsibilidade e simplicidade de venda.

| Modelo | Como funciona | Prós | Contras |
|--------|---------------|------|---------|
| **Tiers puros (ESCOLHIDO)** | Assinatura fixa por faixa (atendentes/canais/volume) | Simples de vender e entender; conta previsível | Margem exposta quando uso de IA dispara → mitigar via cotas por tier |
| Híbrido | Assinatura por tiers + excedente de IA/mensagens | Receita previsível + margem protegida | Mais complexo de comunicar e faturar |
| Puro consumo | Cobrança por mensagem/conversa/IA | Alinha custo↔receita | Imprevisível para a PME; difícil de orçar |

Custos variáveis reais a cobrir no pricing: tokens de IA generativa; cobrança por conversa da Meta na API Oficial (WABA); infra por tenant (compute, storage, Redis).

## Roadmap parkeado (pós-MVP)

Derivado das forças do ZPRO mapeadas em `RE_ANALISE.md`, mantidas fora do MVP mas no horizonte:

- **Expansão de canais:** Instagram, Messenger, Telegram, e-mail/Gmail, webchat, TikTok, Mercado Livre, OLX.
- **WABA avançado:** templates, catálogo/produtos, ligações (SIP/WAVOIP), webhooks keep-alive.
- **Integrações:** Google Calendar (ligado a oportunidades), n8n, Typebot, webhooks bidirecionais, API pública.
- **IA avançada:** múltiplos provedores simultâneos, reescrita por idioma, agentes mais autônomos.
- **White-label do tenant:** cores, logo, domínio próprio por empresa-cliente (se o posicionamento evoluir nessa direção).

## Diferencial técnico a detalhar no PRD/arquitetura

**Camada de canal unificada** — contrato único (`send` / `receive` / `status`) sobre múltiplas APIs de WhatsApp, com:
- health-check por conexão e reconexão automática;
- fallback entre provedores quando uma API não-oficial cai;
- isolamento da API não-oficial como plugin substituível (lição direta das trocas de fork da Baileys no ZPRO);
- testes de contrato por provedor para evitar o ciclo de "correções gerais uazapi/zapi toda semana" observado no ZPRO.

## Notas de modelo de negócio

- unibot **não** revende licença a desenvolvedores (modelo ZPRO). Bruno é o operador único; tenants = empresas-clientes finais.
- O multitenant existe para o operador servir muitas empresas — não para terceiros revenderem.

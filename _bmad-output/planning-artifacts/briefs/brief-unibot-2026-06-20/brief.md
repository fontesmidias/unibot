---
title: "Product Brief: unibot"
status: ready
created: 2026-06-20
updated: 2026-06-20
---

# Product Brief: unibot

> Insumos-base: `../SUMARIO_INSUMOS.md`, `../RE_ANALISE.md`. Detalhe estendido em `addendum.md`.

## Executive Summary

O **unibot** é uma plataforma SaaS de **atendimento e automação multicanal** para pequenas e médias empresas que vendem e atendem pelo WhatsApp. Reúne multi-atendimento, chatbot visual, CRM/funil e IA numa única plataforma **gerenciada e multitenant** — o cliente apenas usa, sem nunca tocar em servidor, atualização ou API de WhatsApp.

O mercado brasileiro hoje força uma escolha ruim. De um lado, plataformas amplas como o **ZPRO** entregam muitos recursos, mas no modelo de **licença self-hosted**, em que quem compra herda toda a operação. De outro, soluções como o **Jetsales** são simples e bem posicionadas, mas rasas — presas a um único canal. O unibot ocupa o meio premium: **a amplitude do ZPRO entregue com a simplicidade e estabilidade de um SaaS gerenciado**.

A aposta é direta: transformar tudo o que hoje é dor do operador (estabilidade de conexão, deploy, manutenção) em **responsabilidade do unibot** — e cobrar por isso de forma recorrente. Quem opera a plataforma é o Bruno; cada empresa-cliente é um tenant que consome o serviço pronto. Agora é o momento, com a maturação da API Oficial do WhatsApp (WABA) e a IA generativa acessível, que barateia a automação de atendimento.

## The Problem

PMEs que dependem do WhatsApp para vender e atender vivem três dores simultâneas:

1. **Atendimento caótico e não-escalável.** Um número, vários atendentes brigando pelo mesmo celular, conversas perdidas, sem fila, sem histórico, sem métricas. Quando a empresa cresce, o WhatsApp "na unha" trava o crescimento.
2. **Automação fora de alcance.** Montar chatbot, qualificar lead e integrar com CRM exige ou contratar agência, ou comprar uma plataforma técnica demais para o time da PME operar.
3. **As alternativas atuais transferem o problema errado.** Plataformas self-hosted como o ZPRO resolvem recursos, mas entregam à empresa uma operação de TI que ela não quer nem sabe tocar: VPS, atualização manual com "faça backup antes", e a instabilidade recorrente das APIs não oficiais (reconectar, reler QR Code, breaking changes). A PME quer **vender mais**, não administrar servidor. As alternativas simples, por sua vez, prendem a empresa a um único canal e pouca profundidade.

O custo do status quo: leads perdidos por demora na resposta, equipe afogada, e dependência de uma solução frágil que pode cair no pior momento — sem ninguém claramente responsável por levantá-la.

## The Solution

Uma plataforma de atendimento que a PME **liga e usa no mesmo dia**:

- **Multi-atendimento organizado** — vários agentes no mesmo número, filas por setor (vendas, suporte, financeiro), transferência e transbordo bot→humano.
- **Chatbot/flowbuilder visual** — automações com gatilhos, condições e captura de variáveis, sem código; qualifica e filtra leads 24/7.
- **CRM e funil integrados** — contatos com tags e histórico, Kanban de oportunidades, agendamento e campanhas/disparos com personalização.
- **IA plugável** — resposta, qualificação e reescrita por IA (múltiplos provedores), como parte nativa do fluxo.
- **100% gerenciado e multitenant** — onboarding self-service, provisionamento do tenant em minutos, e toda a camada de infraestrutura, conexão de WhatsApp e atualizações por conta do unibot. O cliente nunca vê uma VPS.

O coração técnico é uma **camada de canal unificada**: um contrato único sobre as várias APIs de WhatsApp (oficial WABA + alternativas), com health-check, reconexão automática e fallback entre provedores — exatamente a fragilidade que derruba o ZPRO, aqui resolvida na infraestrutura e invisível para o cliente.

## What Makes This Different

- **Gerenciado, não self-hosted.** O diferencial nº 1 não é uma feature, é o **modelo de entrega**: o cliente consome um serviço estável; a operação é nossa. É o que separa o unibot da categoria "compre a licença e se vire".
- **Amplitude do ZPRO + simplicidade do Jetsales.** Múltiplos recursos sem a complexidade técnica; um canal forte (WhatsApp) com profundidade real e caminho para outros.
- **Estabilidade como produto.** A camada de canal unificada com fallback transforma a maior dor do mercado (a conexão caindo) em uptime que o concorrente self-hosted não consegue prometer — uma vantagem invisível, mas sentida.
- **Honestidade sobre o moat.** Não há tecnologia secreta intransponível aqui. A vantagem real é **execução e operação**: produto bem-feito, onboarding sem atrito e confiabilidade que sustenta a assinatura recorrente. O fosso se cava com velocidade e qualidade de serviço, não com patente.

## Who This Serves

**Usuário-cliente primário:** PMEs brasileiras que atendem e vendem por WhatsApp — comércio/varejo, serviços, educação, saúde e similares — com uma equipe pequena de atendimento (de 1 a ~15 atendentes) que hoje opera no WhatsApp comum ou numa ferramenta frágil. Para elas, sucesso é **responder rápido, não perder lead e enxergar o atendimento** sem virar área de TI.

**Dentro do tenant:** dono/gestor (decide a assinatura; quer métricas e custo previsível), atendente (quer uma tela simples que não trave e organize as conversas) e, eventualmente, marketing (campanhas e fluxos de qualificação).

**Operador da plataforma:** Bruno (e equipe), que administra todos os tenants via painel central — provisionamento, billing, saúde das conexões.

## Success Criteria

**Sinais de produto/usuário**
- Tempo de onboarding de um novo tenant até o primeiro atendimento real < 1 dia (idealmente minutos).
- Uptime das conexões de WhatsApp ≥ 99,5% — métrica-âncora do diferencial.
- Redução do tempo de primeira resposta (TPR) e do tempo médio de atendimento (TMA) nos clientes ativos.

**Sinais de negócio**
- Primeiros 10 clientes pagantes (meta de prazo a definir conforme go-to-market).
- Churn mensal < 5% após o 3º mês.
- Margem de contribuição positiva por tenant (receita da assinatura > custo de infra/IA por cliente).

## Scope

**Dentro do MVP (validar e vender):**
- Canal **WhatsApp**: API Oficial (WABA) **+ uma API não oficial** com a camada unificada e fallback.
- Multi-atendimento: filas/setores, transferência, transbordo, histórico.
- Chatbot/flowbuilder visual básico (gatilhos, condições, captura de variáveis, transbordo humano).
- CRM: contatos, tags, funil/Kanban de oportunidades.
- Campanhas/disparo em massa com personalização e agendamento.
- IA plugável (1 provedor no MVP) para resposta/qualificação.
- **Multitenant + painel de operador (superadmin)** + onboarding self-service + billing de assinatura.

**Explicitamente fora do MVP (roadmap):**
- Outros canais (Instagram, Messenger, Telegram, e-mail/Gmail, webchat, TikTok, ML, OLX).
- Ligações via WABA (SIP/WAVOIP), catálogo/produtos WABA, templates avançados.
- Google Calendar, integrações n8n/Typebot, webhooks avançados.
- White-label profundo para o cliente final (cores/domínio próprio do tenant) — MVP mantém a marca unibot.

## Monetização

**Assinatura por planos fixos por faixa (tiers puros)** — planos mensais/anuais por faixa (ex.: número de atendentes, canais e/ou volume incluído). Sem cobrança variável de excedente: a conta é previsível para a PME e simples de vender.

- **Trade-off assumido:** a simplicidade e a previsibilidade favorecem a conversão da PME; em contrapartida, a margem fica exposta quando o uso de IA generativa e a cobrança por conversa da WABA crescem. **Mitigação:** dimensionar cotas e custos de IA/WABA dentro de cada tier para proteger a margem, e revisar o pricing conforme os dados reais de consumo.
- **Alternativas consideradas e descartadas** (híbrido, puro consumo): registradas no `addendum.md`.

## Riscos & Dependências

- **Dependência de plataformas externas.** A API Oficial (WABA) está sujeita às políticas, preços por conversa e mudanças da Meta; as APIs não oficiais, a instabilidade e a termos de uso do WhatsApp. A camada de canal unificada mitiga, mas não elimina, esse risco.
- **Custo variável de IA.** Sob assinatura de tiers puros, picos de uso de IA generativa pressionam a margem — exige cotas bem dimensionadas por plano e monitoramento de consumo real.
- **Operação como produto.** O diferencial (estabilidade gerenciada) exige capacidade real de operar conexões e infraestrutura de forma confiável, idealmente 24/7. Confiabilidade é promessa de venda — uma queda prolongada fere o posicionamento.
- **Concentração no operador.** No início, a operação depende fortemente do Bruno. Escalar de dezenas para centenas de tenants exige automação operacional e, eventualmente, time.
- **Horizonte/timeline:** prazos do MVP e do piloto ainda **a definir** conforme o go-to-market (ver Success Criteria).

## Vision

Em 2-3 anos, o unibot é a forma **padrão e confiável** de uma PME brasileira atender e vender por mensagem — começando no WhatsApp e expandindo para um verdadeiro **hub omnichannel gerenciado** (Instagram, e-mail, webchat e mais), com automação por IA cada vez mais autônoma na qualificação e no atendimento. O operador escala de dezenas para centenas/milhares de tenants sobre a mesma base estável, e a confiabilidade operacional — não uma feature isolada — vira a marca registrada que sustenta a recorrência e o boca a boca.

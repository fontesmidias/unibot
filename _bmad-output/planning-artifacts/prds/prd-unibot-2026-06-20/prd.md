---
title: "PRD — unibot"
status: final
created: 2026-06-20
updated: 2026-07-09
---

# PRD — unibot

> Plataforma SaaS multitenant de atendimento e automação no WhatsApp para PMEs brasileiras, **operada e gerenciada** pelo unibot. O cliente vende e atende; a estabilidade da conexão, a infraestrutura e os updates são responsabilidade da plataforma.

---

## 1. Contexto e problema

PMEs brasileiras que vendem e atendem pelo WhatsApp enfrentam três dores simultâneas:

1. **Atendimento caótico e não escalável** — vários atendentes no mesmo número, sem fila, sem histórico, sem métricas.
2. **Automação fora de alcance** — montar chatbot, qualificar lead e integrar CRM exige agência ou plataforma técnica demais.
3. **Alternativas inadequadas** — soluções self-hosted (ex.: Z-PRO) transferem operação de TI para a empresa (VPS, manutenção manual, instabilidade de APIs não oficiais); soluções simples e verticais (estilo Jetsales) prendem-se a um único canal raso, sem profundidade de automação/CRM. O unibot ataca **os dois flancos**: amplitude do primeiro, simplicidade do segundo.

**Oportunidade:** entregar a amplitude de recursos das plataformas self-hosted (multicanal, chatbot, CRM, multitenant) como **SaaS gerenciado e estável**, eliminando a dor operacional. O cliente nunca toca em VPS, update ou biblioteca de conexão. Essa estabilidade operacional é o que justifica a assinatura recorrente.

**Por que agora:** a **API Oficial do WhatsApp (WABA)** amadureceu e se tornou acessível a PMEs, e a **IA generativa** ficou barata e boa o suficiente para qualificar e responder — a janela para um SaaS gerenciado que combina os dois está aberta.

---

## 2. Proposta de valor e diferencial

O unibot combina **amplitude de funcionalidades** com **simplicidade de uso** e **estabilidade como produto**. Os diferenciais abaixo não são teóricos: cada um ataca uma dor que o concorrente Z-PRO **não resolveu ao longo de 2 anos de releases** (ver `analise-concorrente-zpro.md`).

- **Gerenciado, não self-hosted** — diferencial nº 1. Operação é responsabilidade do unibot. *(No Z-PRO, cada update é procedimento manual de VPS; o incidente Hostinger de 2026 derrubou clientes por infra de terceiros fora do controle deles.)*
- **Camada de canal gerida, não multiplicada** — o **WABA oficial** é a espinha dorsal estável; **poucos** drivers não oficiais (2, não 7) entram como fallback, cada um isolado e coberto por **testes de contrato**, com updates que **não derrubam a sessão**. O diferencial não é "a API não oficial é estável" (nenhuma é) — é **gerir poucas conexões com disciplina** em vez de empilhar sete frágeis. *(O Z-PRO respondeu à instabilidade somando 7 APIs; o aviso "reler QRCode/recriar canal" aparece em quase toda release.)*
- **Amplitude com simplicidade** — multi-atendimento, chatbot visual, CRM/funil, campanhas e IA sem complexidade técnica.
- **Isolamento e governança desde o dia 1** — multitenancy, RBAC, auditoria e conformidade LGPD como fundação, não retrofit. *(No Z-PRO, isolamento cross-tenant e permissão de superadmin geraram "updates críticos" de segurança.)*
- **Honestidade sobre o moat** — a vantagem é execução, operação confiável e onboarding sem atrito; não há tecnologia secreta.

### Visão de longo prazo

Tornar-se o **hub de atendimento omnichannel gerenciado** padrão para PMEs brasileiras — onde WhatsApp é a porta de entrada e a **confiabilidade operacional vira marca registrada**, sustentada por boca a boca de clientes que "nunca mais tiveram o número caindo". O MVP é a cabeça de ponte no WhatsApp; a camada de canal já é desenhada para absorver os demais canais sem reescrita.

---

## 3. Objetivos e métricas de sucesso

### Métricas-âncora (produto/usuário)
| Métrica | Meta MVP |
|---|---|
| Uptime das conexões WhatsApp | **≥ 99,5%** (âncora do diferencial) |
| Tempo de onboarding até primeiro atendimento real | **< 1 dia** (ideal: minutos) |
| Tempo de primeira resposta (TPR) e tempo médio de atendimento (TMA) | Redução mensurável nos clientes ativos |

### Métricas de negócio
| Métrica | Meta MVP |
|---|---|
| Primeiro(s) cliente(s) pagante(s) | em **~30 dias** (via piloto — ver §11.1) `[ASSUMPTION: "primeiros pagantes" = 1–3 no piloto, não os 10 — confirmar com Bruno]` |
| 10 clientes pagantes | marco subsequente (prazo a definir — Q3/go-to-market) |
| Churn mensal (após 3º mês) | **< 5%** |
| Margem de contribuição por tenant | **Positiva** — depende do pricing (Q3): a assinatura de cada tier precisa cobrir infra + **IA (tokens)** + **custo de conversa WABA da Meta**. Sob tiers puros, o risco de margem é real (ver R2). |

### Contra-métricas (vigiar para não "ganhar errado")
- **Custo de IA + conversa WABA por tenant** pressionando a margem sob tiers puros → exige enforcement de cotas (ver R2, Features 6 e 9).
- **Tempo de conexão caída** prolongado → fere diretamente o posicionamento de estabilidade.
- **Carga operacional sobre o Bruno** crescendo linearmente com nº de tenants → exige automação operacional (ver NFR-2/3 e R4).

---

## 4. Usuários

**Segmento-alvo do MVP:** PMEs brasileiras que atendem e vendem pelo WhatsApp — com destaque para **varejo/comércio, serviços, educação e saúde** — com equipe de **1 a ~15 atendentes**. Mercado amplo, sem segmentação restrita no MVP.

Personas com contexto inline (sem seção isolada):

- **Bruno — operador da plataforma.** Administra todos os tenants por um painel central (superadmin). Quer escalar para dezenas/centenas de tenants sem que a operação dependa dele a cada incidente.
- **Dono/gestor da PME (tenant).** Decide a assinatura. Quer métricas, custo previsível e a sensação de que "funciona sozinho".
- **Atendente (tenant).** Vive na tela do dia a dia. Quer uma caixa de entrada simples que organize as conversas e não trave.
- **Marketing (tenant, eventual).** Presente sobretudo em varejo e educação; roda campanhas de disparo, cria os fluxos de qualificação e acompanha conversão do funil. Depende de segmentação por tag e relatórios de campanha (FR-30–34).

### 4.1 Jornadas de usuário

**UJ-1 — Marina, dona de uma clínica de estética (onboarding self-service).** Marina assina o unibot num sábado à noite pelo site, sem falar com vendedor. Cria a conta, é levada pelo assistente de configuração: lê o QR Code para conectar o número do WhatsApp da clínica, cria o setor "Agendamentos", monta um primeiro fluxo simples ("Oi! Quer agendar? Responda 1"). Em minutos recebe a primeira mensagem de cliente já dentro da plataforma. No domingo já está no trial, sem ter tocado em nada técnico. *(Exercita FR-48→51, FR-3, FR-10, FR-18.)*

**UJ-2 — Diego, atendente de uma loja de materiais de construção (dia de operação).** Diego chega e abre a caixa de entrada compartilhada: 12 conversas na fila do setor "Vendas". Ele assume três, responde com mensagens rápidas de preço, transfere uma dúvida técnica para o colega do setor "Pós-venda" e marca um contato como oportunidade no funil. Quando o volume aperta, o bot tria as novas conversas e só passa para ele as que pedem humano. Ao fim do dia, o dono vê no dashboard que o TPR caiu. *(Exercita FR-9→16, FR-12, FR-27, FR-59.)*

---

## 5. Escopo

### 5.1 Dentro do MVP
Canal WhatsApp (oficial + não oficiais com fallback), multi-atendimento, chatbot/flowbuilder visual, CRM/funil, campanhas, IA plugável, multitenancy + painel do operador, onboarding self-service, billing por assinatura (cartão + Pix) e dashboard básico de métricas.

### 5.2 Explicitamente fora do MVP (roadmap — ver §10)
- **Outros canais:** Instagram, Messenger, Telegram, e-mail/Gmail, webchat, TikTok, Mercado Livre, OLX.
- **WABA avançado:** templates avançados, catálogo/produtos, ligações (SIP/WAVOIP), webhooks keep-alive.
- **Integrações:** Google Calendar, n8n, Typebot, webhooks bidirecionais, API pública.
- **IA avançada:** reescrita por idioma, agentes autônomos.
- **White-label de tenant:** cores/logo/domínio próprio do cliente. MVP mantém a marca unibot.

---

## 6. Funcionalidades e requisitos

> Requisitos funcionais com **IDs globais estáveis** (FR-N), agrupados por feature. Descrevem **capacidades**, não implementação — escolhas técnicas (provedores concretos, transporte, stack) estão no `addendum.md`.

### Feature 1 — Camada de Canal Unificada (WhatsApp)
*O coração do diferencial. Abstrai múltiplos provedores de WhatsApp sob um contrato único com fallback.*

- **FR-1** — Expor um contrato único de canal (enviar / receber / status) sobre múltiplos provedores de WhatsApp (oficial e não oficiais), de modo que o restante do sistema não saiba qual provedor está em uso.
- **FR-2** — Permitir que um tenant configure uma ou mais conexões de WhatsApp, respeitando o limite de canais do seu plano.
- **FR-3** — Suportar a conexão por QR Code (provedores não oficiais) e por credenciais da conta oficial (WABA).
- **FR-4** — Executar health-check contínuo por conexão e reconexão automática quando uma conexão cai.
- **FR-5** — Fazer **fallback automático** entre provedores quando a conexão primária degrada ou cai, preservando a continuidade do atendimento.
- **FR-6** — Enviar e receber os tipos de mensagem essenciais: texto, imagem, áudio, vídeo, documento e localização.
- **FR-7** — Refletir indicadores de presença ("digitando"/"gravando") e status de entrega/leitura quando o provedor suportar.
- **FR-8** — Registrar e auditar eventos de conexão (queda, troca de provedor, reconexão) de forma visível ao operador e ao tenant.
- **FR-63** — Normalizar de forma robusta a **identidade do contato** (nono dígito brasileiro, identificadores `@lid`/LID e mudanças de endereçamento do WhatsApp), evitando contatos duplicados e mensagens entregues ao destinatário errado. *(Bug perene do Z-PRO — corrigido 6+ vezes ao longo de 2 anos.)*
- **FR-64** — Gerir a **janela de conversa de 24h do WABA**: sinalizar quando a janela está aberta/fechada e exigir template aprovado para mensagens iniciadas pela empresa fora da janela.
- **FR-65** — Cadastrar e usar **templates WABA aprovados** (com variáveis e mídia) para mensagens iniciadas pela empresa no canal oficial (pré-requisito para campanhas e notificações via WABA).

### Feature 2 — Multi-atendimento (Inbox unificado)
- **FR-9** — Caixa de entrada compartilhada exibindo conversas em tempo real.
- **FR-10** — Filas/setores configuráveis e roteamento de conversas para o setor correto.
- **FR-11** — Atribuir e transferir conversas entre atendentes.
- **FR-12** — Transbordo bot→humano e humano→bot dentro da mesma conversa/número.
- **FR-13** — Histórico completo da conversa por contato, persistente entre sessões.
- **FR-14** — Notas internas e tags na conversa.
- **FR-15** — Estados de conversa (aberta, pendente, resolvida) com reabertura.
- **FR-16** — Respostas rápidas / mensagens prontas reutilizáveis.
- *(FR-17 — CSAT pós-conversa — movido para o roadmap; ID aposentado, não reutilizar.)*

### Feature 3 — Chatbot / Flowbuilder visual
- **FR-18** — Editor visual de fluxos (arrastar-e-soltar) por tenant, sem código.
- **FR-19** — Gatilhos: palavra-chave, início de conversa e horário/agenda.
- **FR-20** — Condições: igual, contém, começa com, termina com e regex.
- **FR-21** — Captura de variáveis `{var}` a partir das mensagens, persistidas no contato.
- **FR-22** — Ações de fluxo: enviar mensagem, atribuir tag, mover no funil, acionar IA e transbordo para humano. *(Webhook/integrações externas ficam fora do MVP — ver §5.2.)*
- **FR-23** — Mensagens por inatividade / agendadas dentro do fluxo.
- **FR-24** — Testar/pré-visualizar o fluxo antes de publicar.

### Feature 4 — CRM & Funil
- **FR-25** — Cadastro de contatos com campos e tags.
- **FR-26** — Funil de vendas em Kanban com etapas configuráveis.
- **FR-27** — Oportunidades/cards vinculados a contato e à conversa de origem.
- **FR-28** — Busca e filtros de contatos (por tag, etapa, atributo).
- **FR-29** — Importação de contatos por arquivo (CSV) para uso em campanhas.

### Feature 5 — Campanhas / Disparo em massa
- **FR-30** — Criar campanhas com segmentação por tag/filtro.
- **FR-31** — Personalização da mensagem por variáveis do contato.
- **FR-32** — Agendamento de envio.
- **FR-33** — Controle de ritmo do disparo com **delay aleatório entre mensagens** (anti-banimento), reduzindo risco de bloqueio do número.
- **FR-34** — Relatório de entrega da campanha (enviadas, entregues, falhas).
- **FR-70** — Executar o disparo em massa **de forma assíncrona no servidor** (não depender de manter uma tela/aba aberta). *(Limitação arquitetural do Z-PRO: o disparo exige a tela aberta.)*

### Feature 6 — IA plugável (camada de IA unificada)
*Espelha a camada de canal: contrato único sobre múltiplos provedores de IA, com seleção e fallback.*

- **FR-35** — Expor uma camada de IA com **contrato único** sobre múltiplos provedores, de modo que fluxos e atendentes invoquem "IA" sem acoplamento ao provedor.
- **FR-36** — Selecionar provedor/modelo de IA por tenant (ou definido pelo operador).
- **FR-37** — Usar IA como ação de fluxo: resposta automática e qualificação de lead.
- **FR-38** — Sugerir/reescrever resposta para o atendente (assistência humana no atendimento).
- **FR-39** — Contabilizar o consumo de IA por tenant (tokens/uso) para enforcement de cota e billing.
- **FR-40** — Fazer fallback entre provedores de IA em caso de falha ou limite de um provedor.
- **FR-66** — Agrupar mensagens em sequência (**debounce**) antes de acionar a IA, evitando respostas fragmentadas quando o cliente envia várias mensagens seguidas. *(Diferencial que o Z-PRO precisou introduzir.)*

### Feature 7 — Multitenancy & Painel do Operador (Superadmin)
- **FR-41** — Isolar dados entre tenants (um tenant nunca acessa dados de outro).
- **FR-42** — Painel do operador (Bruno) com visão de todos os tenants.
- **FR-43** — Criar, suspender e reativar tenants.
- **FR-44** — Definir e alterar o plano (tier) e os limites de cada tenant.
- **FR-45** — "Login como" tenant para suporte, **com auditoria** do acesso.
- **FR-46** — Visão consolidada da saúde das conexões de todos os tenants (centro de operação).
- **FR-47** — Gestão de papéis e usuários dentro do tenant (gestor, atendente, marketing) com **RBAC confiável** desde o início.
- **FR-67** — **Autenticação de dois fatores (2FA)** para usuários do tenant e do operador.
- **FR-68** — Restrição de visibilidade de dados sensíveis do contato por perfil (ocultar/mascarar telefone, CPF, e-mail; blur de foto), alinhada à LGPD.
- **FR-69** — Sessão única / forçar logout ao autenticar em novo dispositivo (opcional por tenant).

### Feature 8 — Onboarding self-service
- **FR-48** — Cadastro e criação de conta self-service.
- **FR-49** — Provisionamento automático do tenant (em minutos, sem intervenção manual).
- **FR-50** — Assistente de configuração inicial: conectar WhatsApp, criar primeiro setor e primeiro fluxo.
- **FR-51** — Ativação de plano com **trial gratuito por tempo determinado (7–14 dias)**; ao fim, conversão para plano pago ou bloqueio controlado do acesso.

### Feature 9 — Billing & Assinatura
- **FR-52** — Planos por **tier** com limites em quatro dimensões: nº de atendentes, nº de canais/conexões, cota de IA (tokens) e volume de mensagens/contatos.
- **FR-53** — Assinatura recorrente por **cartão** (gateway brasileiro).
- **FR-54** — Pagamento por **Pix**.
- **FR-55** — Enforcement dos limites do tier: aviso ao se aproximar e bloqueio/degradação controlada ao atingir a cota.
- **FR-56** — Medição de uso por ciclo (atendentes ativos, canais, tokens de IA, mensagens) para billing e exibição.
- **FR-57** — Histórico de faturas e pagamentos visível ao tenant.
- **FR-58** — Gestão de inadimplência: suspensão por falta de pagamento, com fluxo de regularização.
- **FR-71** — **Régua de cobrança preventiva (dunning)** antes da suspensão: avisos de fatura próxima do vencimento, retry automático de pagamento e período de carência — reduzindo churn involuntário. *(O Z-PRO só adicionou cobrança preventiva tardiamente.)*

### Feature 10 — Relatórios & Dashboard básico
- **FR-59** — Dashboard com TPR, TMA e volume de atendimentos.
- **FR-60** — Métricas por atendente e por setor.
- **FR-61** — Status e uptime das conexões de WhatsApp do tenant.
- **FR-62** — Indicadores de uso vs. limites do plano (consumo de IA, mensagens, canais, atendentes).

---

## 7. Requisitos não funcionais (transversais)

- **NFR-1 — Disponibilidade.** Uptime **≥ 99,5%** por conexão, medido em janela mensal como *% de tempo em que a conexão está apta a enviar/receber*, **excluindo** indisponibilidade da própria Meta/WhatsApp (fora do controle do unibot — reportada à parte). Sustentado por fallback (FR-5) e health-check (FR-4); atualizações da plataforma **não podem derrubar a sessão** do cliente. **Premissa/limite honesto:** o fallback entre drivers **não** protege contra **falha de modo comum** — mudanças na plataforma WhatsApp (ex.: `@lid`, Passkey) já quebraram *todas* as libs não oficiais de uma vez; nesses eventos, **só o WABA oficial é caminho independente**. Por isso o WABA é a espinha dorsal (ver §2). Métrica-âncora do produto.
- **NFR-2 — Observabilidade desde o dia 1.** Métricas, logs e alertas por conexão, tenant e fila; o operador detecta degradação antes do cliente.
- **NFR-3 — Operação como produto e maturidade de release.** Deploy sem downtime, migrations versionadas, updates reversíveis, **CI/CD** e **testes de contrato por provedor de canal** (evitam o padrão Z-PRO de "correção geral" semanal por API e hotfixes reemitidos no mesmo dia). Cliente nunca edita dependências em produção.
- **NFR-4 — Segurança e LGPD.** Conversas contêm dados pessoais de terceiros. **Papéis:** o tenant é o **controlador** e o unibot, **operador**. Requisitos: **DPA (contrato de tratamento) padrão** oferecido a todo tenant no onboarding; isolamento por tenant; criptografia em trânsito e em repouso; **retenção configurável**; atendimento a direitos do titular (**exclusão e exportação a pedido**); restrição de visibilidade de dados sensíveis por perfil (FR-68) e 2FA (FR-67).
- **NFR-5 — Escalabilidade multitenant.** Arquitetura que cresça de 1 a **centenas de tenants** sem reescrita, com provisionamento automático (FR-49). `[ASSUMPTION]` alvo de referência do MVP: suportar ~200 tenants e picos de disparo sem degradar o tempo real; números finos a validar em arquitetura.
- **NFR-6 — Tempo real.** Entrega/recebimento de mensagem e atualização da inbox com latência-alvo **p95 < 2s** em condições normais. `[ASSUMPTION]` a confirmar em arquitetura.
- **NFR-7 — Resiliência de recursos.** Limites de recurso e backpressure nas filas para evitar vazamentos de memória/CPU/cache. (Lição direta do Z-PRO: `pm2 restart` agendado, disparo travando a tela.)
- **NFR-8 — Backup e recuperação.** Backup por tenant com restauração testada; alvos **RPO ≤ 24h e RTO ≤ 4h**. `[ASSUMPTION]` a confirmar.
- **NFR-9 — Auditoria.** Trilha de ações sensíveis: "login como" (FR-45), mudança de plano, suspensão, exclusão de dados.
- **NFR-10 — Portabilidade.** Exportação dos dados do tenant (contatos, conversas) sob demanda.
- **NFR-11 — Ingestão resiliente de eventos.** Recebimento de webhooks/eventos do WhatsApp com buffer e retry, de modo que instabilidade de rede/infra (ex.: incidente Hostinger bloqueando webhooks da Meta, no Z-PRO) não perca mensagens.

---

## 8. Multitenancy, operação e governança

*Cluster de concerns que atravessa várias features — consolidado para o leitor.*

- O **operador (Bruno)** é o centro de controle: provisiona, precifica, suporta e monitora todos os tenants (Feature 7). O design precisa reduzir a carga operacional por tenant para que a escala não dependa de esforço linear do operador (contra-métrica de §3).
- A **camada de canal** (Feature 1) é simultaneamente diferencial de produto e mecanismo operacional: é ela que converte instabilidade de APIs externas em uptime prometido.
- **Cotas e enforcement** (Features 6 e 9) são o mecanismo de proteção de margem sob tiers puros — sem eles, o custo variável de IA corrói a margem.

---

## 9. Riscos

| # | Risco | Mitigação no produto |
|---|---|---|
| R1 | **Dependência de plataformas externas** (WABA + APIs não oficiais fora do controle) | Camada de canal unificada com fallback (FR-1, FR-5) e isolamento de cada provedor como driver substituível. |
| R2 | **Custo variável de IA** pressionando margem sob tiers puros | Contabilização de uso (FR-39) + cotas por tier e enforcement (FR-52, FR-55). |
| R3 | **Operação como produto** — promessa de estabilidade exige confiabilidade real | Observabilidade (NFR-2), operação como produto (NFR-3), resiliência (NFR-7). |
| R4 | **Operador único como ponto de falha existencial** — dependência do Bruno (bus-factor) para incidentes, bans e reconexões | Onboarding self-service (Feature 8) + centro de operação (FR-46) reduzem a intervenção por tenant, mas **não removem o bus-factor**. Residual aceito no MVP; escala exige runbooks, automação de recuperação e, adiante, um segundo operador. |
| R5 | **Bloqueio de número** em disparos/uso de APIs não oficiais | Throttling com delay aleatório (FR-33); disparo server-side (FR-70); fallback de canal (FR-5); conformidade de template/janela WABA (FR-64, FR-65). **No MVP a mitigação é reativa**; monitoramento proativo de saúde/qualidade do número é roadmap. |
| R6 | **Conformidade LGPD** com dados de terceiros | Isolamento, retenção e direitos do titular (NFR-4, NFR-10); 2FA e restrição de visibilidade de dados (FR-67, FR-68). |
| R7 | **Risco de plataforma Meta** — mudança de política/preço do WABA, restrição de conta/app, e quebras que atingem **todas** as libs não oficiais de uma vez: identificador (`@lid`) e **vínculo de dispositivo (Passkey)** | WABA oficial como caminho **independente** de 1ª classe; camada de canal que abstrai identificador (FR-63) e reconecta (FR-4); isolamento por conta. **Mitigação estrutural (BSP/Tech Provider próprio) é roadmap — risco residual permanece no MVP.** *(No Z-PRO, a conta Meta da própria empresa foi restringida em 2026, derrubando clientes.)* |
| R8 | **Custo de suporte por instabilidade** crescendo com a base (o Z-PRO citou ~40 chamados/dia) | Estabilidade estrutural (NFR-1/3), observabilidade proativa (NFR-2) e onboarding self-service (Feature 8) reduzem volume de chamado. |
| R9 | **Prazo agressivo (30 dias)** vs. escopo do MVP (10 features / 71 FRs) | Entregar por **fatia vendável** (ver §11.1). Princípio inegociável: não comprometer a estabilidade de canal (o diferencial) para cumprir prazo. |

---

## 10. Roadmap pós-MVP

Sequência indicativa (prioridade a refinar). Itens marcados ⚑ são **tablestakes emergentes** — o Z-PRO já os tem e o mercado passará a esperá-los; os demais são expansão.

1. **Copiloto de IA no atendimento** ⚑ — resumo de conversa, sugestão de resposta, **análise de sentimento**, detecção de urgência, tradução inline. *(Já é diferencial ativo do Z-PRO; forte candidato a subir de prioridade.)*
   - **CSAT** — avaliação de atendimento pós-conversa, alimentando o dashboard.
2. **Expansão de canais** — Instagram, Messenger, Telegram, e-mail/Gmail, webchat (a camada de canal já está desenhada para isso).
3. **WABA avançado** ⚑ — Embedded Signup / caminho **Meta Business Partner (BSP) próprio** (blindagem contra risco R7), catálogo/produtos, flows, ligações (SIP/WAVOIP), webhooks keep-alive, **Modo Híbrido** (oficial + não oficial no mesmo número).
4. **Monitoramento de saúde/qualidade do número** ⚑ — score de qualidade Meta, throttle preventivo (mitiga R5/R7).
5. **Integrações e API externa** ⚑ — API pública por tenant, n8n, Google Calendar, Typebot, webhooks bidirecionais.
6. **Agendamento nativo** — agenda tipo Google, com **turmas/vagas por horário** (clínicas, cursos) — nicho relevante no BR.
7. **IA avançada** — agentes autônomos, orquestração multi-provedor mais rica.
8. **White-label de tenant** — cores, logo e domínio próprio.
9. **Canais marketplace** — Mercado Livre, OLX, Nuvemshop/WooCommerce, TikTok.
10. **Escala** — modo cluster e storage externo (S3/MinIO) conforme o volume exigir.

---

## 11. Questões em aberto

| # | Questão | Dono | Condição de revisão |
|---|---|---|---|
| Q1b | **Recorte fino da fatia vendável de 30 dias** — quais FRs entram no primeiro corte | Bruno + Arquitetura | No `bmad-create-epics-and-stories` |
| Q3 | **Valores e cotas exatas dos tiers** — preços por plano e limites por dimensão | Bruno | Antes do go-to-market (não bloqueia arquitetura) |

**Resolvidas (2026-06-20):** Q2 CSAT → roadmap · Q4 Trial → gratuito 7–14 dias (FR-51) · Q5 LGPD → unibot operador + DPA padrão (NFR-4) · Q6 Gateway → decisão técnica no addendum · Estrutura de tiers (3 planos, 4 dimensões) → §11.1 · **Q1 Timeline → meta de primeiros pagantes em ~30 dias, via fatia vendável enxuta (§11.1)**.

### 11.1 Straw-man de tiers e fases `[ASSUMPTION — aprovar/ajustar]`

**Planos (3 tiers puros — sem cobrança de excedente; ao atingir o limite, aviso e bloqueio/degradação controlada). Estrutura confirmada; números `[A DEFINIR]` (Q3, go-to-market):**

| Dimensão | Essencial | Profissional | Business |
|---|---|---|---|
| Atendentes | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| Canais/conexões WhatsApp | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| Cota de IA (tokens/mês) | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| Volume de mensagens de campanha/mês | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| Contatos no CRM | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| Preço mensal (R$) | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |

> Confirmado: **3 planos** com gating nas **4 dimensões** (refletido em FR-52/55/56). Valores exatos são go-to-market e não bloqueiam arquitetura.

**Timeline (Q1) — meta: primeiros pagantes em ~30 dias.**

⚠️ **Tensão de escopo:** 30 dias não comportam as 10 features / 71 FRs. A meta é atingível apenas com uma **primeira fatia vendável** enxuta; o restante do MVP segue logo após, com clientes já pagando. Recomendação para arquitetura/epics:

- **Fatia vendável (≈30 dias) — o mínimo que um cliente paga por:**
  - **Camada de canal estável** (Feature 1) com pelo menos WABA + 1 driver não oficial e fallback — *é o diferencial; sem isso não há produto.*
  - **Inbox multi-atendimento** essencial (Feature 2): filas, transferência, histórico.
  - **Multitenancy + onboarding self-service + billing** (Features 7–9): trial, cartão/Pix, provisionamento automático — *sem isso não há como vender/cobrar.* **Bloqueio de partida:** a escolha do gateway (Q6) precisa ser resolvida logo, pois billing está no primeiro corte.
  - **Dashboard mínimo** de status de conexão e volume (subconjunto da Feature 10).
- **Logo após (semanas seguintes):** flowbuilder + IA plugável (Features 3, 6), campanhas (Feature 5), CRM/funil completo (Feature 4), dashboard completo, FRs de governança (2FA, visibilidade de dados).

> A sequência exata e o recorte fino da fatia vendável são trabalho de **epics/arquitetura**; o PRD registra a meta de 30 dias e o princípio ("estabilidade de canal + capacidade de cobrar primeiro").

---

## 12. Glossário

| Termo | Definição |
|---|---|
| **unibot** | A plataforma SaaS multitenant descrita neste PRD. |
| **Tenant** | Uma empresa-cliente do unibot. Cada tenant tem seus dados isolados. |
| **Operador / Superadmin** | O unibot (Bruno) operando a plataforma sobre todos os tenants. |
| **Camada de canal unificada** | Abstração que expõe um **contrato de canal** único (enviar/receber/status) sobre vários provedores de WhatsApp. |
| **Driver** | Implementação da camada de canal para um provedor específico (WABA, Evolution, Baileys). |
| **Fallback** | Troca automática para outro driver quando o primário degrada/cai. |
| **WABA** | WhatsApp Business Platform (API Oficial da Meta). Caminho independente e estável. |
| **API não oficial** | Biblioteca/serviço não homologado pela Meta (Baileys, Evolution). Sujeita a quebra. |
| **Janela de 24h** | Período do WABA em que a empresa pode responder livremente; fora dela, exige **template** aprovado. |
| **Template WABA** | Mensagem pré-aprovada pela Meta para contato iniciado pela empresa. |
| **Transbordo** | Passagem de uma conversa entre bot e humano (nos dois sentidos). |
| **Flowbuilder** | Editor visual de fluxos de automação/chatbot. |
| **Funil / Kanban** | Visualização de oportunidades por etapa de venda. |
| **Camada de IA plugável** | Abstração com contrato único sobre múltiplos provedores de IA, com seleção e fallback. |
| **Debounce (IA)** | Agrupar mensagens em sequência antes de acionar a IA (FR-66). |
| **Tier** | Plano de assinatura com limites fixos (tiers puros = sem cobrança de excedente). |
| **Cota** | Limite de uso por tier (atendentes, canais, tokens de IA, mensagens/contatos). |
| **Dunning** | Régua de cobrança preventiva antes da suspensão por inadimplência (FR-71). |
| **DPA** | Acordo de tratamento de dados entre controlador (tenant) e operador (unibot). |

---

## 13. Índice de suposições

Suposições inferidas (não confirmadas diretamente) — a validar antes de arquitetura/go-to-market:

- **A1** — "Primeiro(s) pagante(s) em ~30 dias" = 1–3 clientes no piloto, não os 10 (§3). **Aguardando confirmação do Bruno.**
- **A2** — Estrutura e valores dos tiers do straw-man (§11.1) são ponto de partida; números `[A DEFINIR]`.
- **A3** — Alvo de escala do MVP ~200 tenants (NFR-5).
- **A4** — Latência tempo real p95 < 2s (NFR-6).
- **A5** — RPO ≤ 24h / RTO ≤ 4h (NFR-8).

---

> Decisões técnicas (provedores de canal/IA, gateway, stack, transporte) estão em `addendum.md`. Análise competitiva completa em `analise-concorrente-zpro.md`.

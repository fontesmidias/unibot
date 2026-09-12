---
title: "PRD — unibot"
status: final
created: 2026-06-20
updated: 2026-09-12
---

# PRD — unibot

> 🔒 **Versão anonimizada para versionamento.** As referências competitivas deste documento foram
> substituídas por designações neutras (Concorrente A, B, C). Os achados técnicos, datas e versões
> são preservados integralmente — apenas a identificação das fontes foi removida.
> A versão com as fontes nomeadas é mantida fora do controle de versão.


> Plataforma SaaS multitenant de atendimento e automação no WhatsApp para PMEs brasileiras, **operada e gerenciada** pelo unibot. O cliente vende e atende; a estabilidade da conexão, a infraestrutura e os updates são responsabilidade da plataforma.

---

## 1. Contexto e problema

PMEs brasileiras que vendem e atendem pelo WhatsApp enfrentam três dores simultâneas:

1. **Atendimento caótico e não escalável** — vários atendentes no mesmo número, sem fila, sem histórico, sem métricas.
2. **Automação fora de alcance** — montar chatbot, qualificar lead e integrar CRM exige agência ou plataforma técnica demais.
3. **Alternativas inadequadas** — soluções self-hosted (ex.: Concorrente A) transferem operação de TI para a empresa (VPS, manutenção manual, instabilidade de APIs não oficiais); soluções simples e verticais (estilo Concorrente B) prendem-se a um único canal raso, sem profundidade de automação/CRM. O unibot ataca **os dois flancos**: amplitude do primeiro, simplicidade do segundo.

**Oportunidade:** entregar a amplitude de recursos das plataformas self-hosted (multicanal, chatbot, CRM, multitenant) como **SaaS gerenciado e estável**, eliminando a dor operacional. O cliente nunca toca em VPS, update ou biblioteca de conexão. Essa estabilidade operacional é o que justifica a assinatura recorrente.

**Por que agora:** a **API Oficial do WhatsApp (WABA)** amadureceu e se tornou acessível a PMEs, e a **IA generativa** ficou barata e boa o suficiente para qualificar e responder — a janela para um SaaS gerenciado que combina os dois está aberta.

---

## 2. Proposta de valor e diferencial

O unibot combina **amplitude de funcionalidades** com **simplicidade de uso** e **estabilidade como produto**. Os diferenciais abaixo não são teóricos: cada um ataca uma dor que o concorrente Concorrente A **não resolveu ao longo de 2 anos de releases** (ver `analise-concorrente.md`).

- **Gerenciado, não self-hosted** — diferencial nº 1. Operação é responsabilidade do unibot. *(No Concorrente A, cada update é procedimento manual de VPS; o incidente Hostinger de 2026 derrubou clientes por infra de terceiros fora do controle deles.)*
- **Camada de canal gerida, não multiplicada** — o **WABA oficial** é a espinha dorsal estável; **poucos** drivers não oficiais (2, não 7) entram como fallback, cada um isolado e coberto por **testes de contrato**, com updates que **não derrubam a sessão**. O diferencial não é "a API não oficial é estável" (nenhuma é) — é **gerir poucas conexões com disciplina** em vez de empilhar sete frágeis. *(O Concorrente A respondeu à instabilidade somando 7 APIs; o aviso "reler QRCode/recriar canal" aparece em quase toda release.)*
- **Amplitude com simplicidade** — multi-atendimento, chatbot visual, CRM/funil, campanhas e IA sem complexidade técnica.
- **Isolamento e governança desde o dia 1** — multitenancy, RBAC, auditoria e conformidade LGPD como fundação, não retrofit. *(No Concorrente A, isolamento cross-tenant e permissão de superadmin geraram "updates críticos" de segurança.)*
- **Honestidade sobre o moat** — a vantagem é execução, operação confiável e implantação bem conduzida; não há tecnologia secreta. *(O benchmark do Concorrente B confirma que, neste mercado, embalagem e acompanhamento sustentam um ticket várias vezes maior que amplitude funcional bruta — ver §11.1.)*

### Visão de longo prazo

Tornar-se o **hub de atendimento omnichannel gerenciado** padrão para PMEs brasileiras — onde WhatsApp é a porta de entrada e a **confiabilidade operacional vira marca registrada**, sustentada por boca a boca de clientes que "nunca mais tiveram o número caindo". O MVP é a cabeça de ponte no WhatsApp; a camada de canal já é desenhada para absorver os demais canais sem reescrita.

---

## 3. Objetivos e métricas de sucesso

### Métricas-âncora (produto/usuário)
| Métrica | Meta MVP |
|---|---|
| Uptime das conexões WhatsApp | **≥ 99,5%** (âncora do diferencial) |
| Tempo da contratação até o primeiro atendimento real | **≤ 1 reunião de implantação** — o cliente sai da primeira sessão guiada já atendendo pela plataforma. *(Revisada em 2026-09-12: sob venda assistida, a métrica deixa de medir autonomia do cliente e passa a medir eficiência da implantação — que é o gargalo do operador, ver R4.)* |
| Tempo de primeira resposta (TPR) e tempo médio de atendimento (TMA) | Redução mensurável nos clientes ativos |

### Métricas de negócio
| Métrica | Meta MVP |
|---|---|
| Primeiro(s) cliente(s) pagante(s) | **1–3 clientes em regime de piloto**, com data derivada da estimativa de esforço em `bmad-create-epics-and-stories` — não fixada aqui *(revisado em 2026-09-12, decisão D4: a meta anterior de "~30 dias" foi definida em 20/06/2026 e venceu sem código escrito)* |
| 10 clientes pagantes | marco subsequente, **limitado pela capacidade de implantação do operador** (ver R4 e Q7), não pela demanda — prazo a definir no go-to-market |
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

**UJ-1 — Marina, dona de uma clínica de estética (aquisição e implantação assistida).** Marina vê o unibot num anúncio e pede uma demonstração pelo site. Na chamada, em vez de slides, o Bruno navega por um ambiente já povoado com conversas e contatos fictícios: ela vê a tela de atendimento, reconhece o WhatsApp e entende em dois minutos o que muda na clínica. Fecha o plano com implantação. Na primeira reunião de onboarding, o Bruno provisiona o tenant, conduz a leitura do QR Code do número da clínica, e juntos criam o setor "Agendamentos" e um primeiro fluxo ("Oi! Quer agendar? Responda 1"). Marina sai da reunião com a clínica atendendo pela plataforma; as pendências ficam registradas para a reunião seguinte. Ela nunca tocou em nada técnico — e nunca ficou sozinha diante de uma tela de configuração. *(Exercita FR-48→51, FR-81, FR-82, FR-3, FR-10, FR-18.)*

**UJ-2 — Diego, atendente de uma loja de materiais de construção (dia de operação).** Diego chega e abre a caixa de entrada compartilhada: 12 conversas na fila do setor "Vendas". Ele assume três, responde com mensagens rápidas de preço, transfere uma dúvida técnica para o colega do setor "Pós-venda" e marca um contato como oportunidade no funil. Quando o volume aperta, o bot tria as novas conversas e só passa para ele as que pedem humano. Ao fim do dia, o dono vê no dashboard que o TPR caiu. *(Exercita FR-9→16, FR-12, FR-27, FR-59.)*

---

## 5. Escopo

### 5.1 Dentro do MVP
Canal WhatsApp (oficial + não oficiais com fallback), multi-atendimento, chatbot/flowbuilder visual, CRM/funil, campanhas, IA plugável, multitenancy + painel do operador, **aquisição e onboarding assistido**, billing por assinatura (cartão + Pix) e dashboard básico de métricas.

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
- **FR-63** — Normalizar de forma robusta a **identidade do contato** (nono dígito brasileiro, identificadores `@lid`/LID e mudanças de endereçamento do WhatsApp), evitando contatos duplicados e mensagens entregues ao destinatário errado. *(Bug perene do Concorrente A — corrigido 6+ vezes ao longo de 2 anos.)*
- **FR-64** — Gerir a **janela de conversa de 24h do WABA**: sinalizar quando a janela está aberta/fechada e exigir template aprovado para mensagens iniciadas pela empresa fora da janela.
- **FR-65** — Cadastrar e usar **templates WABA aprovados** (com variáveis e mídia) para mensagens iniciadas pela empresa no canal oficial (pré-requisito para campanhas e notificações via WABA).
- **FR-72** — **Circuit breaker por conexão:** após N falhas consecutivas do mesmo tipo, suspender as tentativas de reconexão, expor o motivo em linguagem legível e notificar — em vez de religar indefinidamente. *(No Concorrente A, o loop de reconexão do Baileys elevava o consumo de memória e podia reiniciar o servidor inteiro — v4.0.3.1, 27/07/2026.)*
- **FR-73** — **Health-check por fluxo de dados:** validar o **recebimento recente de eventos**, não apenas o estado declarado da sessão. Sem tráfego na janela esperada, marcar a conexão como degradada e revalidar a assinatura do webhook. *(Falha mais perigosa do domínio: no Concorrente A, canais Meta permaneciam "conectados" e paravam de receber mensagens por dias — v4.0.3.7, 10/08/2026. Um health-check de sessão não detecta isso.)*
- **FR-74** — **Estados de entrega distintos** — aceito pelo provedor / entregue / lido / falhou — nunca colapsando "aceito" em "entregue", tanto na conversa quanto no relatório de campanha. *(O Concorrente A exibia "enviada" para mensagem que nunca chegou — 30/07, 04/08 e 14/08/2026.)*
- **FR-75** — **Mesclagem reversível de contatos duplicados:** detectar duplicatas, mesclar preservando todo o histórico, exibir o que foi afetado e permitir desfazer. *(FR-63 normaliza a identidade; este FR trata o que já duplicou. Mesclagem destrutiva apaga histórico de conversa — dado pessoal sob LGPD.)*
- **FR-76** — **Identidade de contato sem telefone obrigatório:** o modelo de contato aceita o identificador do canal como chave de identidade, com o telefone opcional. *(O WhatsApp está desacoplando identidade de número: BSUID duplicado perdia mensagens e surgiram contatos WABA por username — v4.0.5.0, 09/09/2026. `@lid` e nono dígito eram sintoma, não a doença. Decisão de modelo de dados: barata agora, migração dolorosa depois.)*

### Feature 2 — Multi-atendimento (Inbox unificado)
- **FR-9** — Caixa de entrada compartilhada exibindo conversas em tempo real.
- **FR-10** — Filas/setores configuráveis e roteamento de conversas para o setor correto.
- **FR-11** — Atribuir e transferir conversas entre atendentes.
- **FR-12** — Transbordo bot→humano e humano→bot dentro da mesma conversa/número.
- **FR-13** — Histórico completo da conversa por contato, persistente entre sessões.
- **FR-14** — Notas internas e tags na conversa.
- **FR-15** — Estados de conversa (aberta, pendente, resolvida) com reabertura.
- **FR-16** — Respostas rápidas / mensagens prontas reutilizáveis.
- **FR-79** — **Proteção de referência em uso e rota de escape:** bloquear ou avisar ao excluir entidade referenciada (fila, fluxo, atendente, canal, tag, etapa do funil) e garantir que nenhum atendimento fique preso — destino inválido cai na fila padrão. *(No Concorrente A, excluir fila/fluxo deixava "atendimentos presos num robô mudo" — v4.0.3.8, 11/08/2026 e v4.0.4.6, 29/08/2026: cliente final sem resposta, invisível nas métricas.)*
- *(FR-17 — CSAT pós-conversa — movido para o roadmap; ID aposentado, não reutilizar.)*

### Feature 3 — Chatbot / Flowbuilder visual
- **FR-18** — Editor visual de fluxos (arrastar-e-soltar) por tenant, sem código.
- **FR-19** — Gatilhos: palavra-chave, início de conversa e horário/agenda.
- **FR-20** — Condições: igual, contém, começa com, termina com e, como **recurso avançado opcional**, expressão regular. As condições de uso comum são selecionáveis sem digitar sintaxe; a expressão regular fica atrás de um modo avançado e nunca é exigida para montar um fluxo típico (ver §11.2).
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
- **FR-34** — Relatório de entrega da campanha **por destinatário** (enviadas, aceitas pelo provedor, entregues, lidas, falhas com motivo), permitindo identificar e reprocessar apenas os que falharam. Os estados seguem FR-74 — o relatório nunca reporta "entregue" o que foi apenas aceito pelo provedor.
- **FR-70** — Executar o disparo em massa **de forma assíncrona no servidor** (não depender de manter uma tela/aba aberta). *(Limitação arquitetural do Concorrente A: o disparo exige a tela aberta.)*

### Feature 6 — IA plugável (camada de IA unificada)
*Espelha a camada de canal: contrato único sobre múltiplos provedores de IA, com seleção e fallback.*

- **FR-35** — Expor uma camada de IA com **contrato único** sobre múltiplos provedores, de modo que fluxos e atendentes invoquem "IA" sem acoplamento ao provedor.
- **FR-36** — Selecionar provedor/modelo de IA por tenant (ou definido pelo operador).
- **FR-37** — Usar IA como ação de fluxo: resposta automática e qualificação de lead.
- **FR-38** — Sugerir/reescrever resposta para o atendente (assistência humana no atendimento).
- **FR-39** — Contabilizar o consumo de IA por tenant (tokens/uso) para enforcement de cota e billing.
- **FR-40** — Fazer fallback entre provedores de IA em caso de falha, limite **ou descontinuação de modelo** — incluindo o caso em que o provedor desliga ou renomeia um modelo ainda referenciado por um tenant: o sistema detecta a indisponibilidade, recai sobre um modelo equivalente e notifica o operador, em vez de falhar silenciosamente a cada invocação. *(Dois eventos em sete semanas: cutover forçado da OpenAI em 24/07/2026 e desligamento de modelos Gemini pelo Google em 12/09/2026. Lock-in de modelo quebra sozinho, sem aviso do cliente.)*
- **FR-66** — Agrupar mensagens em sequência (**debounce**) antes de acionar a IA, evitando respostas fragmentadas quando o cliente envia várias mensagens seguidas. *(Diferencial que o Concorrente A precisou introduzir.)*

### Feature 7 — Multitenancy & Painel do Operador (Superadmin)
- **FR-41** — Isolar dados entre tenants (um tenant nunca acessa dados de outro).
- **FR-42** — Painel do operador (Bruno) com visão de todos os tenants.
- **FR-43** — Criar, suspender e reativar tenants.
- **FR-44** — Definir e alterar o plano (tier) e os limites de cada tenant.
- **FR-45** — "Login como" tenant para suporte, **com auditoria** do acesso.
- **FR-46** — Visão consolidada da saúde das conexões de todos os tenants (centro de operação).
- **FR-47** — Gestão de papéis e usuários dentro do tenant (gestor, atendente, marketing) com **RBAC confiável** desde o início, sujeito a três regras verificáveis: (a) **toda autorização é decidida no servidor** — a interface apenas reflete a decisão, nunca a substitui, de modo que acesso por link direto ou chamada de API obedeça à mesma regra que a tela; (b) **nenhum usuário concede a si mesmo ou a outro um papel acima do próprio**, e nenhuma rota de API cria usuário de nível superior ao do chamador; (c) **a autorização cobre todas as superfícies** — incluindo notificações, buscas, exportações e relatórios, não apenas a abertura de telas. *(O Concorrente A precisou de uma release inteira em 17/08/2026 para tapar acesso por link direto, criação de conta de nível plataforma via API e vazamento de metadados por notificação.)*
- **FR-67** — **Autenticação de dois fatores (2FA)** para usuários do tenant e do operador.
- **FR-78** — **Política de senha e convite:** convite por e-mail para o próprio usuário definir a senha e/ou troca obrigatória no primeiro acesso — o administrador nunca conhece a senha do atendente. *(Complemento barato do FR-67. O Concorrente A chegou a set/2026 **sem 2FA**, o que torna FR-67 um diferencial vendável, não mera paridade.)*
- **FR-68** — Restrição de visibilidade de dados sensíveis do contato por perfil (ocultar/mascarar telefone, CPF, e-mail; blur de foto), alinhada à LGPD.
- **FR-69** — Sessão única / forçar logout ao autenticar em novo dispositivo (opcional por tenant).

### Feature 8 — Aquisição e Onboarding assistido
*Reescrita em 2026-09-12. O unibot adota **venda assistida** (modelo Concorrente B), não autoatendimento: a porta de entrada é a demonstração agendada, e a implantação é um serviço guiado e cobrado. O provisionamento automático permanece — mas como **ferramenta do operador**, acionada após a venda, e não como fluxo de autoatendimento do cliente. Ver `benchmark-concorrente-b.md` e decisão D1 no `.decision-log.md`.*

- **FR-48** — **Captura e qualificação de lead:** formulário público de solicitação de demonstração (site/landing) que registra o interessado e agenda a conversa, sem criar tenant nem conceder acesso ao produto. *(Substitui o cadastro self-service.)*
- **FR-49** — **Provisionamento do tenant pelo operador, em minutos:** a partir do painel do operador, criar o tenant já configurado (plano, limites, usuário gestor inicial) sem intervenção manual em infraestrutura. A automação é preservada; o gatilho passa a ser a venda concluída.
- **FR-50** — **Assistente de configuração guiada:** roteiro de implantação que conduz conectar o WhatsApp, criar o primeiro setor e publicar o primeiro fluxo — usado pelo operador junto ao cliente nas reuniões de onboarding e retomável pelo próprio cliente depois.
- **FR-51** — **Ativação de contrato com período de implantação:** o acesso é liberado na contratação, com um período inicial de implantação acompanhada; não há trial autônomo sem contato comercial. Falta de pagamento segue a régua de inadimplência (FR-58, FR-71).
- **FR-81** — **Ambiente de demonstração com dados fictícios:** permitir percorrer o produto povoado com conversas e contatos de exemplo, sem conectar um número real — usado na demonstração comercial e na exploração inicial pelo cliente. *(Padrão do Concorrente C: ver valor antes de qualquer configuração técnica. Remove o QR Code do caminho da decisão de compra.)*
- **FR-82** — **Trilha de implantação registrada:** registrar as etapas do onboarding por tenant (reuniões realizadas, itens configurados, pendências), visível ao operador — para que a implantação seja um processo repetível e auditável, e não dependa da memória do operador. *(Mitigação direta de R4.)*

### Feature 9 — Billing & Assinatura
- **FR-52** — Planos por **tier** com limites em quatro dimensões: nº de atendentes, nº de canais/conexões, cota de IA (tokens) e volume de mensagens/contatos.
- **FR-53** — Assinatura recorrente por **cartão** (gateway brasileiro).
- **FR-54** — Pagamento por **Pix**.
- **FR-55** — Enforcement dos limites do tier: aviso ao se aproximar e bloqueio/degradação controlada ao atingir a cota.
- **FR-56** — Medição de uso por ciclo (atendentes ativos, canais, tokens de IA, mensagens) para billing e exibição.
- **FR-57** — Histórico de faturas e pagamentos visível ao tenant.
- **FR-58** — Gestão de inadimplência: suspensão por falta de pagamento, com fluxo de regularização.
- **FR-71** — **Régua de cobrança preventiva (dunning)** antes da suspensão: avisos de fatura próxima do vencimento, retry automático de pagamento e período de carência — reduzindo churn involuntário. *(O Concorrente A só adicionou cobrança preventiva tardiamente.)*

### Feature 10 — Relatórios & Dashboard básico
- **FR-59** — Dashboard com **TPR** (tempo até a primeira resposta humana), **TME** (tempo de espera em fila), **TTE** (tempo total até a resolução) e volume de atendimentos. A semântica de cada indicador é fixada explicitamente — o que conta como início, o que pausa a contagem (fora do horário de atendimento, aguardando o cliente) e o que encerra — e nenhum indicador pode assumir valor negativo. *(No Concorrente A, TPR/TME retornavam valores negativos e o cálculo foi corrigido 4 vezes em 4 semanas.)*
- **FR-60** — Métricas por atendente e por setor.
- **FR-61** — Status e uptime das conexões de WhatsApp do tenant.
- **FR-62** — Indicadores de uso vs. limites do plano (consumo de IA, mensagens, canais, atendentes).

---

## 7. Requisitos não funcionais (transversais)

- **NFR-1 — Disponibilidade.** Uptime **≥ 99,5%** por conexão, medido em janela mensal como *% de tempo em que a conexão está apta a enviar/receber*, **excluindo** indisponibilidade da própria Meta/WhatsApp (fora do controle do unibot — reportada à parte). Sustentado por fallback (FR-5) e health-check (FR-4); atualizações da plataforma **não podem derrubar a sessão** do cliente. **Premissa/limite honesto:** o fallback entre drivers **não** protege contra **falha de modo comum** — mudanças na plataforma WhatsApp (ex.: `@lid`, Passkey) já quebraram *todas* as libs não oficiais de uma vez; nesses eventos, **só o WABA oficial é caminho independente**. Por isso o WABA é a espinha dorsal (ver §2). Métrica-âncora do produto.
- **NFR-2 — Observabilidade desde o dia 1, orientada à falha silenciosa.** Métricas, logs e alertas por conexão, tenant e fila; o operador detecta degradação antes do cliente. **Princípio de projeto:** a classe de defeito mais perigosa deste domínio não é a queda visível — é a **falha silenciosa**, em que o sistema se declara saudável enquanto o cliente final deixa de ser atendido. Casos observados no Concorrente A entre jul e set/2026: canal "verde" que parou de receber por dias (10/08), mensagem marcada como entregue que nunca chegou (30/07, 04/08, 14/08), atendimento preso em robô mudo (29/08) e agente de IA anunciando transferência que não ocorreu (09/09). Nenhuma delas aparece em dashboard algum. Portanto a observabilidade do unibot deve medir **efeito observável** (evento recebido, mensagem confirmada pelo destinatário, atendimento com destino válido, resposta efetivamente enviada) e não apenas estado declarado de componente. Para um produto cujo diferencial é confiabilidade, **detectar a falha silenciosa vale mais do que elevar o uptime nominal**. Sustentado por FR-73, FR-74 e FR-79.
- **NFR-3 — Operação como produto e maturidade de release.** Deploy sem downtime, migrations versionadas, updates reversíveis, **CI/CD** e **testes de contrato por provedor de canal** (evitam o padrão Concorrente A de "correção geral" semanal por API e hotfixes reemitidos no mesmo dia). Cliente nunca edita dependências em produção.
- **NFR-4 — Segurança e LGPD.** Conversas contêm dados pessoais de terceiros. **Papéis:** o tenant é o **controlador** e o unibot, **operador**. Requisitos: **DPA (contrato de tratamento) padrão** oferecido a todo tenant no onboarding; isolamento por tenant; criptografia em trânsito e em repouso; **retenção configurável**; atendimento a direitos do titular (**exclusão e exportação a pedido**); restrição de visibilidade de dados sensíveis por perfil (FR-68) e 2FA (FR-67). **Revogação de acesso tem efeito imediato:** remover um usuário, rebaixar seu papel ou suspender um tenant encerra as sessões ativas e invalida credenciais em vigor sem janela de tolerância — não se espera a expiração natural do token. *(O Concorrente A publicou uma janela de 30 segundos entre a revogação e seu efeito; para quem acabou de demitir um atendente, 30 segundos é tempo de exportar a base de contatos.)*
- **NFR-5 — Escalabilidade multitenant.** Arquitetura que cresça de 1 a **centenas de tenants** sem reescrita, com provisionamento automático (FR-49). `[ASSUMPTION]` alvo de referência do MVP: suportar ~200 tenants e picos de disparo sem degradar o tempo real; números finos a validar em arquitetura.
- **NFR-6 — Tempo real.** Entrega/recebimento de mensagem e atualização da inbox com latência-alvo **p95 < 2s** em condições normais. `[ASSUMPTION]` a confirmar em arquitetura.
- **NFR-7 — Resiliência de recursos.** Limites de recurso e backpressure nas filas para evitar vazamentos de memória/CPU/cache. (Lição direta do Concorrente A: `pm2 restart` agendado, disparo travando a tela.)
- **NFR-8 — Backup e recuperação.** Backup por tenant com restauração testada; alvos **RPO ≤ 24h e RTO ≤ 4h**. `[ASSUMPTION]` a confirmar.
- **NFR-9 — Auditoria.** Trilha de ações sensíveis: "login como" (FR-45), mudança de plano, suspensão, exclusão de dados, alteração de papel e revogação de acesso. **A trilha é imutável para quem é auditado:** nem o gestor do tenant nem o operador editam ou apagam registros de auditoria, e sua retenção é definida pela política da plataforma — não pela configuração de retenção de conversas do tenant (NFR-4). Auditoria editável pelo auditado não é auditoria.
- **NFR-10 — Portabilidade.** Exportação dos dados do tenant (contatos, conversas) sob demanda.
- **NFR-11 — Ingestão resiliente e idempotente de eventos.** Recebimento de webhooks/eventos do WhatsApp com buffer e retry, de modo que instabilidade de rede/infra (ex.: incidente Hostinger bloqueando webhooks da Meta, no Concorrente A) não perca mensagens. **O retry só é seguro se for idempotente:** (a) **deduplicação por identificador de mensagem do provedor** — reprocessar o mesmo evento nunca cria mensagem duplicada; (b) **exclusão mútua na criação de conversa/ticket** — dois eventos simultâneos do mesmo contato convergem para uma única conversa, nunca duas; (c) ordenação tolerante a chegada fora de sequência. *(No Concorrente A, 29/08/2026: rajadas de mensagens perdiam conteúdo e eventos simultâneos duplicavam a conversa. Retry sem idempotência **produz** duplicata — o mecanismo de resiliência vira a origem do defeito.)*

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
| R4 | **Operador único como ponto de falha existencial — agravado pela venda assistida (2026-09-12)** — o Bruno é o gargalo não só de incidentes, bans e reconexões, mas agora também de **cada demonstração, cada implantação guiada e cada renovação**. A referência que inspirou o modelo (Concorrente B) sustenta essa operação com uma equipe de dezenas de pessoas; aqui há uma só. | Mitigações no produto: centro de operação (FR-46), provisionamento automático pelo operador (FR-49), assistente de configuração guiada (FR-50), ambiente de demonstração com dados fictícios (FR-81) e **trilha de implantação registrada** (FR-82), que torna o onboarding repetível e transferível em vez de tácito. **Mitigação fora do produto (indispensável):** limitar deliberadamente o número de clientes novos por mês à capacidade real de implantação, e padronizar a demonstração e o roteiro de onboarding para permitir delegação futura. **Risco residual alto e conscientemente aceito** — é a contrapartida do ticket mais alto. Este é o risco mais severo do projeto. |
| R5 | **Bloqueio de número** em disparos/uso de APIs não oficiais | Throttling com delay aleatório (FR-33); disparo server-side (FR-70); fallback de canal (FR-5); conformidade de template/janela WABA (FR-64, FR-65). **No MVP a mitigação é reativa**; monitoramento proativo de saúde/qualidade do número é roadmap. |
| R6 | **Conformidade LGPD** com dados de terceiros | Isolamento, retenção e direitos do titular (NFR-4, NFR-10); 2FA e restrição de visibilidade de dados (FR-67, FR-68). |
| R7 | **Risco de plataforma Meta** — mudança de política/preço do WABA, restrição de conta/app, e quebras que atingem **todas** as libs não oficiais de uma vez: identificador (`@lid`) e **vínculo de dispositivo (Passkey)** | WABA oficial como caminho **independente** de 1ª classe; camada de canal que abstrai identificador (FR-63) e reconecta (FR-4); isolamento por conta. **Mitigação estrutural (BSP/Tech Provider próprio) é roadmap — risco residual permanece no MVP.** *(No Concorrente A, a conta Meta da própria empresa foi restringida em 2026, derrubando clientes.)* |
| R8 | **Custo de suporte por instabilidade** crescendo com a base (o Concorrente A citou ~40 chamados/dia) | Estabilidade estrutural (NFR-1/3), observabilidade proativa (NFR-2) e detecção de falha silenciosa (FR-73, FR-74, FR-79) reduzem o volume de chamado na origem. **Sob venda assistida, a implantação guiada (FR-50, FR-82) previne a classe de chamado que nasce de configuração malfeita** — mas, em contrapartida, o cliente que paga mais espera atendimento mais próximo. Efeito líquido sobre R4 a vigiar. |
| R9 | **Escopo do MVP vs. capacidade real de execução** — 10 features e 80+ FRs para um desenvolvedor único. A meta original ("primeiros pagantes em ~30 dias", definida em 20/06/2026) **expirou sem código escrito**: em 12/09/2026 o projeto seguia na fase de planejamento. Tratar prazo otimista como plano foi, ele próprio, parte do problema. | Entregar por **fatia vendável** (ver §11.1), com prazo recontado a partir de **2026-09-12** e revisado ao fim de `bmad-create-epics-and-stories`, quando o esforço estiver estimado story a story — não antes. Princípio inegociável: não comprometer a estabilidade de canal (o diferencial) para cumprir data. **Sob venda assistida o prazo pressiona menos:** bastam poucos clientes para validar receita, e a implantação guiada absorve arestas que o autoatendimento exporia. |

---

## 10. Roadmap pós-MVP

Sequência indicativa (prioridade a refinar). Itens marcados ⚑ são **tablestakes emergentes** — o Concorrente A já os tem e o mercado passará a esperá-los; os demais são expansão.

1. **Copiloto de IA no atendimento** ⚑ — resumo de conversa, sugestão de resposta, **análise de sentimento**, detecção de urgência, tradução inline. *(Já é diferencial ativo do Concorrente A; forte candidato a subir de prioridade.)*
   - **CSAT** — avaliação de atendimento pós-conversa, alimentando o dashboard.
2. **Expansão de canais** — Instagram, Messenger, Telegram, e-mail/Gmail, webchat (a camada de canal já está desenhada para isso).
   - **FR-77 — Detecção de atendimento cross-canal:** ao iniciar conversa ou disparar campanha, avisar se o contato já tem atendimento aberto em outro canal e permitir pular o envio. *(Só faz sentido com múltiplos canais por tenant — por isso acompanha este item, e não o MVP.)*
3. **WABA avançado** ⚑ — Embedded Signup / caminho **Meta Business Partner (BSP) próprio** (blindagem contra risco R7), catálogo/produtos, flows, ligações (SIP/WAVOIP), webhooks keep-alive, **Modo Híbrido** (oficial + não oficial no mesmo número).
4. **Monitoramento de saúde/qualidade do número** ⚑ — score de qualidade Meta, throttle preventivo (mitiga R5/R7).
5. **Integrações e API externa** ⚑ — API pública por tenant, n8n, Google Calendar, Typebot, webhooks bidirecionais.
   - **FR-80 — Cobrança pelo canal:** enviar cobrança Pix/link de pagamento ao contato a partir da conversa ou do funil. *(Lançado pelo Concorrente A em 11/08/2026. Conecta funil a receita com forte apelo à PME brasileira. Depende de templates WABA maduros (FR-65) e de integração de gateway voltada ao **cliente do tenant** — distinta do billing do unibot (FR-53/54).)*
6. **Agendamento nativo** — agenda tipo Google, com **turmas/vagas por horário** (clínicas, cursos) — nicho relevante no BR.
7. **IA avançada** — agentes autônomos, orquestração multi-provedor mais rica.
8. **White-label de tenant** — cores, logo e domínio próprio.
9. **Canais marketplace** — Mercado Livre, OLX, Nuvemshop/WooCommerce, TikTok.
10. **Escala** — modo cluster e storage externo (S3/MinIO) conforme o volume exigir.

---

## 11. Questões em aberto

| # | Questão | Dono | Condição de revisão |
|---|---|---|---|
| Q1b | **Recorte fino da fatia vendável** — quais FRs entram no primeiro corte, e a data derivada da estimativa | Bruno + Arquitetura | No `bmad-create-epics-and-stories` |
| Q3 | **Valores e cotas exatas dos tiers** — preços por plano, limites por dimensão e **preço da implantação** | Bruno | Antes do go-to-market (não bloqueia arquitetura) |
| Q8 | **Preço da implantação** — cobrar em linha com o concorrente (recupera CAC, mantém barreira) ou barato/incluso (argumento de venda, pressiona A8). Ver §11.1 | Bruno | Junto com Q3, após a primeira implantação cronometrada |
| Q7 | **Capacidade de implantação do operador** — quantos clientes novos por mês o Bruno consegue implantar sem degradar o serviço (limite real de crescimento, ver R4/A6) | Bruno | Após a primeira implantação real, cronometrada |

**Resolvidas (2026-09-12):** **D1 Go-to-market** → venda assistida, modelo Concorrente B (Feature 8 reescrita; R4 agravado) · **D4 Prazo** → recalibrado, data derivada dos epics e não fixada no PRD (R9, §11.1) · **Q6 Gateway** → **RESOLVIDA: Asaas** (Mercado Pago como alternativa; Stripe eliminado pelo gate de Pix). Análise em `../../decisao-gateway-pagamento.md`; decisão e ação bloqueante de tokenização registradas no addendum §1.4.

**Resolvidas (2026-06-20):** Q2 CSAT → roadmap · Q4 Trial → gratuito 7–14 dias (FR-51) *(superado por D1 em 2026-09-12 — não há trial autônomo)* · Q5 LGPD → unibot operador + DPA padrão (NFR-4) · Q6 Gateway → *(resolvida em 2026-09-12: **Asaas**, ver addendum §1.4)* · Estrutura de tiers (3 planos, 4 dimensões) → §11.1 · **Q1 Timeline → meta de primeiros pagantes em ~30 dias** *(superada por D4 em 2026-09-12 — prazo recalibrado, ver R9 e §11.1)*.

### 11.1 Empacotamento, tiers e faseamento `[ASSUMPTION — aprovar/ajustar]`

**Modelo comercial (decisão D1, 2026-09-12): venda assistida.** A porta de entrada é a demonstração agendada; a implantação é serviço guiado e cobrado. Isso muda o empacotamento em três pontos: (a) não há trial autônomo; (b) existe uma **receita de implantação** além da assinatura; (c) o ticket-alvo é substancialmente mais alto, com menos clientes.

**Âncoras de mercado observadas** (ver `benchmark-concorrente-b.md` e `benchmark-concorrente-c.md`):

| Referência | Modelo | Ticket observado |
|---|---|---|
| Concorrente B | Venda assistida, onboarding obrigatório | R$ 349–989/mês + R$ 999–1.899 de implantação; add-ons (atendente R$ 80, conexão R$ 120, agente de IA R$ 599) |
| Concorrente A | Licença self-hosted | ~R$ 166/mês equivalente (R$ 1.997/ano) |
| Concorrente C | Autoatendimento, extensão Chrome | ~R$ 33/mês (R$ 397/ano) |

> **O achado que orienta o pricing:** a Concorrente B entrega **menos** funcionalidade que o Concorrente A e cobra **5 a 9 vezes mais**. A diferença é embalagem, prova social, onboarding e time comercial — não capacidade técnica. O unibot pretende ter a amplitude do primeiro com a embalagem do segundo.

**Planos (3 tiers puros — sem cobrança de excedente; ao atingir o limite, aviso e bloqueio/degradação controlada). Estrutura confirmada; números `[A DEFINIR]` (Q3, go-to-market):**

| Dimensão | Essencial | Profissional | Business |
|---|---|---|---|
| Atendentes | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| Canais/conexões WhatsApp | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| Cota de IA (tokens/mês) | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| Volume de mensagens de campanha/mês | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| Contatos no CRM | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| Preço mensal (R$) | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |
| **Implantação (uma vez)** | `[A DEFINIR]` | `[A DEFINIR]` | `[A DEFINIR]` |

> Confirmado: **3 planos** com gating nas **4 dimensões** (FR-52/55/56), agora acrescidos da **receita de implantação**. Valores exatos são go-to-market e não bloqueiam arquitetura. `[ASSUMPTION]` Sem fidelidade contratual — é cláusula vendável contra a multa de 50% praticada pelo Concorrente B (ver §11.2).

> ⚖️ **Decisão pendente sobre o preço da implantação (Q8).** Adotar a receita de implantação copia a prática do Concorrente B — mas o próprio benchmark identifica a barreira de entrada de R$ 999–1.899 como **fragilidade explorável**, não como virtude. Há duas jogadas opostas, e a escolha é de posicionamento, não de planilha:
> - **Cobrar em linha com o concorrente** — recupera CAC no primeiro mês e sustenta a suposição A8 (a implantação paga o tempo do operador), ao custo de manter a mesma barreira de entrada que torna o concorrente atacável.
> - **Cobrar significativamente menos, ou incluir a implantação no plano** — vira argumento de venda direto ("implantação inclusa, sem taxa de entrada"), acelera a conversão e ataca a fragilidade do concorrente; em troca, transfere todo o peso da amortização para a assinatura recorrente e **coloca A8 em risco**.
>
> A decisão interage com Q3 (preço dos tiers) e com Q7 (capacidade de implantação): implantação barata só é sustentável se o roteiro for eficiente o bastante para não consumir o operador. **Decidir após cronometrar a primeira implantação real.**

**Timeline — recalibrada em 2026-09-12 (decisão D4).**

A meta original ("primeiros pagantes em ~30 dias", fixada em 20/06/2026) **venceu sem código escrito**. Ela é substituída pelo seguinte princípio, sem data fixada aqui:

> A data de lançamento passa a ser derivada da estimativa de esforço em `bmad-create-epics-and-stories`, não fixada antes dela. O que o PRD preserva é a **ordem**, não o calendário.

- **Fatia vendável — o mínimo que sustenta um cliente pagante:**
  - **Camada de canal estável** (Feature 1) com WABA + 1 driver não oficial e fallback, incluindo **FR-72, FR-73 e FR-74** — *é o diferencial; sem isso não há produto.*
  - **Inbox multi-atendimento** essencial (Feature 2): filas, transferência, histórico, com **FR-79**.
  - **Multitenancy + painel do operador + billing** (Features 7 e 9) e **provisionamento pelo operador** (FR-49): sem isso não há como entregar nem cobrar. A escolha do gateway (Q6) precisa estar resolvida.
  - **Ambiente de demonstração** (FR-81) — sob venda assistida, é **pré-requisito comercial**: é o que se mostra na demo antes de existir cliente configurado.
  - **Dashboard mínimo** de status de conexão e volume (subconjunto da Feature 10).
- **Logo após:** flowbuilder + IA plugável (Features 3, 6), campanhas (Feature 5), CRM/funil completo (Feature 4), dashboard completo, governança (2FA, FR-78, visibilidade de dados).

> **Efeito da venda assistida sobre o faseamento:** com implantação guiada, o operador cobre manualmente lacunas que o autoatendimento exporia — o que permite vender mais cedo, com escopo menor. Em contrapartida, **FR-81 sobe de prioridade** (não se vende sem demonstrar) e o autoatendimento sai do caminho crítico.

### 11.2 Posicionamento e experiência — diretrizes dos benchmarks

Insumos de `benchmark-concorrente-c.md` e `benchmark-concorrente-b.md`. Detalhamento visual é trabalho de `bmad-ux`; aqui ficam apenas as diretrizes que condicionam requisitos.

**Da experiência (Concorrente C):**
- **Mimetismo deliberado do WhatsApp Web** na tela do atendente — três colunas, bolhas, ícones e composição reconhecíveis à primeira vista; as camadas extras (funil, painel de contato, atalhos) se revelam depois. A familiaridade que a extensão obtém de graça, o unibot compra com design.
- **Inbox é a tela padrão; Kanban é visão alternativa.** Nunca abrir no funil.
- **A conexão server-side é argumento de venda, não requisito escondido:** "conectou uma vez, funciona com seu computador desligado". O concorrente depende do PC ligado com o WhatsApp Web aberto — limitação admitida pelo próprio fornecedor.
- **Respostas rápidas com `/atalho` e variáveis** (FR-16) provam valor no primeiro dia.
- **Fluxos por blocos pré-prontos** (FR-18), sem exigir JSON nem expressão regular no caminho comum — o modo avançado do FR-20 existe, mas nunca é pré-requisito para montar um fluxo típico.

**Da embalagem (Concorrente B):**
- **Vocabulário de resultado comercial**, não de suporte técnico.
- **Preços públicos**, mesmo sem autoatendimento — ancoram, filtram e qualificam o lead.
- **Confiabilidade como cláusula contratual vendável:** as reclamações graves do Concorrente B concentram-se em suporte inacessível e multa de cancelamento. **Sem fidelidade, SLA explícito e página de status pública** atacam exatamente essa brecha — e são coerentes com NFR-1 e NFR-2.
- **FAQ que enfrenta as objeções reais:** banimento de número, privacidade das conversas, relação com a Meta, portabilidade dos dados na saída (FR-68, NFR-4, NFR-10).

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
| **Falha silenciosa** | Defeito em que o sistema se declara saudável enquanto o cliente final deixa de ser atendido — canal "conectado" que parou de receber, mensagem "entregue" que não chegou, atendimento sem destino válido. Alvo central da observabilidade (NFR-2, FR-73, FR-74, FR-79). |
| **Venda assistida** | Modelo comercial adotado em 2026-09-12: aquisição por demonstração agendada e implantação guiada e cobrada, sem autoatendimento nem trial autônomo (Feature 8). |
| **Implantação** | Serviço de onboarding conduzido pelo operador junto ao cliente, cobrado à parte da assinatura (FR-50, FR-82). |
| **DPA** | Acordo de tratamento de dados entre controlador (tenant) e operador (unibot). |

---

## 13. Índice de suposições

Suposições inferidas (não confirmadas diretamente) — a validar antes de arquitetura/go-to-market:

- ~~**A1**~~ — **Resolvida em 2026-09-12:** "primeiros pagantes" = 1–3 clientes em piloto (§3). A componente de prazo foi superada por D4 — a data passa a ser derivada dos epics.
- **A2** — Estrutura e valores dos tiers do straw-man (§11.1) são ponto de partida; números `[A DEFINIR]`.
- **A3** — Alvo de escala do MVP ~200 tenants (NFR-5).
- **A4** — Latência tempo real p95 < 2s (NFR-6).
- **A5** — RPO ≤ 24h / RTO ≤ 4h (NFR-8).
- **A6** — *(2026-09-12)* Sob venda assistida, a capacidade de implantação do operador único é o limite real de crescimento — não a demanda nem a infraestrutura. O número de clientes novos por mês precisa ser deliberadamente limitado a essa capacidade (ver R4). **Valor a definir após a primeira implantação real.**
- **A7** — *(2026-09-12)* Ausência de fidelidade contratual como diferencial competitivo (§11.2) pressupõe que a retenção se sustente pela qualidade do serviço. A confirmar no go-to-market.
- **A8** — *(2026-09-12)* A receita de implantação cobre o custo do tempo do operador no onboarding. **A validar com a primeira implantação cronometrada** — se não cobrir, o modelo de venda assistida perde sua principal justificativa econômica.

---

> Decisões técnicas (provedores de canal/IA, gateway, stack, transporte) estão em `addendum.md`. Análise competitiva completa em `analise-concorrente.md`.

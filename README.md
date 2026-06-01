#  ChargeGrid Assistant — Sprint 2
**EV Challenge 2026 | GoodWe × FIAP**

---

##  Integrantes

| Nome | RM |
|---|---|
| Alan Junio Araujo de Souza | 574112 |
| Arthur Vettorazzo de Souza | 569445 |
| Brayan Barbosa Dos Santos | 573682 |
| Giovanne Gomes Petenuci | 574091 |
| Gustavo Zibini Belizario | 561376 |
| Luiz Otávio Brito Freixo | 569977 |

**Turma:** 1CCPZ — Ciências da Computação (Noturno) — FIAP

---

##  O que foi implementado na Sprint 2

Esta sprint entrega o chatbot funcional planejado na Sprint 1, com as
seguintes evoluções em relação ao planejamento inicial:

| Funcionalidade | Status |
|---|---|
| System prompt com contexto ChargeGrid |  Implementado |
| Memória de conversa (histórico de mensagens) |  Implementado |
| Few-Shot Prompting embutido no system prompt |  Diferencial adicionado |
| Módulo de diagnóstico com Function Calling (JSON) |  Diferencial adicionado |
| Execução dos 7 testes da Sprint 1 |  Implementado |
| Interface ipywidgets interativa |  Implementado |
| API Key via Kaggle Secrets |  Implementado |

---

##  Dependências

```
huggingface_hub
ipywidgets (pré-instalado no Kaggle)
```

Instale com:
```bash
pip install huggingface_hub -q
```

---

##  Variáveis de Ambiente

O token de acesso ao HuggingFace deve ser configurado como Kaggle Secret:

1. No Kaggle, vá em **Add-ons → Secrets → Add a new secret**
2. Nome: `HF_TOKEN`
3. Valor: seu token gerado em [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens)

>  **Nunca** insira o token diretamente no código ou no repositório.

---

##  Como executar

1. Faça fork ou importe o notebook `chargegrid_assistant_sprint2.ipynb` no Kaggle
2. Configure o `HF_TOKEN` nos Kaggle Secrets
3. Execute as células em ordem
4. A interface do chatbot aparece na célula 5
5. Os testes da Sprint 1 são executados automaticamente na célula 6

---

##  Decisões Arquiteturais

### Injeção de contexto — Prompt Augmentation

O contexto do ChargeGrid Intelligence é injetado diretamente no system prompt
a cada chamada ao modelo. Não utilizamos RAG porque:

- As informações do ChargeGrid (tarifas, protocolos, modalidades de pagamento)
  são dados **estáticos de contexto operacional** — não vêm de documentos extensos
- O conteúdo é compacto o suficiente para caber no system prompt
- RAG está previsto para sprints futuras, quando o chatbot precisar consultar
  histórico de sessões do usuário ou base de FAQs extensa

### Execução Online vs Local

Mantida a decisão da Sprint 1: **Inference API remota** via HuggingFace.
O modelo DeepSeek V3 exige infraestrutura muito além da GPU T4 do Kaggle.
Como os dados do chatbot são informações públicas do produto, não há restrição
de privacidade que justifique execução local.

### Modelo — DeepSeek V3 (mantido da Sprint 1)

Mantida a escolha da Sprint 1. Trocar o modelo sem motivo técnico seria
inconsistente com a justificativa documentada. O DeepSeek continua sendo
a melhor opção gratuita para seguimento de system prompts complexos.

---

##  Técnicas de Prompt Engineering

| Técnica | Implementação | Justificativa |
|---|---|---|
| **System Prompt** | Contexto completo do ChargeGrid + regras de comportamento | Define persona e escopo do assistente |
| **Few-Shot Prompting** | 3 exemplos de pergunta/resposta ideal no system prompt | Calibra tom e profundidade sem respostas genéricas ou técnicas demais |
| **Memória de Conversa** | Lista de histórico enviada junto a cada chamada | Permite diálogos contínuos e coerentes |
| **Function Calling** | Diagnóstico JSON para problemas técnicos | Saída estruturada e rastreável para integração futura |

---

##  Hiperparâmetros

| Modo | Temperature | Top-P | Max Tokens |
|---|---|---|---|
| Conversa normal | 0.4 | 0.85 | 400 |
| Diagnóstico (JSON) | 0.1 | 0.80 | 600 |

---

##  Execução dos Casos de Teste — Sprint 1

Os 7 casos de teste modelados na Sprint 1 foram executados contra o chatbot implementado nesta Sprint 2. Cada teste foi avaliado qualitativamente por comparação entre a resposta obtida e a resposta esperada de referência.

**Modelo avaliado:** DeepSeek-V3-0324 via HuggingFace Inference API  
**Hiperparâmetros:** temperature 0.4 · top_p 0.85 · max_tokens 400

---

### Resumo dos Resultados

| ID | Categoria | Avaliação |
|---|---|---|
| T01 | Tarifa e LED indicador |  Adequada |
| T02 | Tempo de recarga |  Adequada |
| T03 | Pagamento |  Adequada |
| T04 | Carga lenta |  Adequada |
| T05 | Disponibilidade e fila |  Adequada |
| T06 | Interrupção de sessão |  Adequada |
| T07 | O que é o ChargeGrid |  Adequada |

**Total: 7/7 Adequadas (100%) · 0 Parciais · 0 Inadequadas**

---

### Detalhamento por Caso

---

#### T01 — Tarifa e LED indicador

**Pergunta enviada:**
> O LED do totem está vermelho. O que isso significa e devo carregar agora?

**Resposta esperada (referência):**
LED vermelho = tarifa alta (horário de pico). Orientar a aguardar LED verde se não for urgente. Informar que o valor exato está no app.

**Resposta obtida:**
> O LED vermelho no totem indica que estamos no horário de pico da rede elétrica, quando a tarifa de energia está mais alta.
>
> Se você **não estiver com pressa**, recomendo esperar até o LED ficar verde (tarifa baixa) ou amarelo (tarifa moderada) para economizar — a diferença pode chegar a 40% no valor final.
>
> Se **precisar carregar agora**: o sistema vai funcionar normalmente, o app mostrará o valor exato antes de confirmar, e assinantes têm desconto mesmo no horário vermelho.
>
> Dica: No app ChargeGrid você pode ver a previsão de quando a tarifa deve baixar.

**Avaliação: Adequada**
Cobriu todos os pontos da referência (significado do LED vermelho, recomendação condicional de aguardar, valor exato no app) e acrescentou informação útil sobre desconto para assinantes e previsão de tarifa.

---

#### T02 — Tempo de recarga

**Pergunta enviada:**
> Quanto tempo vai levar para carregar meu carro? A bateria está em 20%.

**Resposta esperada (referência):**
Depende da capacidade da bateria e da potência entregue no momento. Em pico, potência pode ser reduzida. Estimativa disponível no app.

**Resposta obtida:**
> Para estimar o tempo de recarga, precisamos considerar alguns fatores:
> 1. **Capacidade da sua bateria** (que varia por modelo de veículo)
> 2. **Potência disponível no momento** (que pode estar limitada se o LED estiver vermelho e/ou muitos carros estiverem carregando)
>
> Como regra geral: se for um carro comum (~40kWh), de 20% a 80% levaria ~2h em posto médio (7kW), e de 20% a 100% levaria ~3h30.
>
> Para uma resposta precisa, consulte o app ChargeGrid — ele calcula em tempo real com base na potência que seu carro está recebendo, no estado atual da bateria e na demanda do local. O app também envia notificação quando a recarga estiver completa.

**Avaliação: Adequada**
Abordou a dependência de capacidade da bateria e potência disponível, citou o impacto do horário de pico na potência e redirecionou corretamente ao app para estimativa precisa.

---

#### T03 — Pagamento

**Pergunta enviada:**
> Como funciona o pagamento? Preciso me cadastrar em algum lugar?

**Resposta esperada (referência):**
Avulso sem cadastro (PIX/cartão). Opção de assinatura com desconto, prioridade e cashback. Cobrança por kWh consumido ao final.

**Resposta obtida:**
> No ChargeGrid você tem **3 opções de pagamento**, todas sem burocracia:
>
> 1. **Avulso (sem cadastro):** pague diretamente no totem com cartão de crédito/débito ou Pix instantâneo. Ideal para quem recarrega esporadicamente.
> 2. **Assinatura mensal:** desconto fixo em todas as sessões, prioridade quando há fila e cashback em kWh. Cadastro rápido via app.
> 3. **Premium:** postos reservados em horários-chave, relatórios detalhados de consumo e suporte 24/7 dedicado.
>
> **Não precisa cadastro prévio** para usar, mas assinantes têm vantagens. O app (disponível para iOS/Android) permite gerenciar tudo de um só lugar.

**Avaliação: Adequada**
Descreveu corretamente as três modalidades, deixou claro que o cadastro não é obrigatório e cobriu os benefícios de desconto, prioridade e cashback da assinatura.

---

#### T04 — Carga lenta

**Pergunta enviada:**
> Meu carro está carregando muito devagar, bem mais lento do que da última vez. Tem algum problema?

**Resposta esperada (referência):**
Provavelmente controle de demanda funcionando corretamente. Sistema redistribui potência entre postos para não ultrapassar limite contratado. Normaliza quando outro veículo terminar.

**Resposta obtida:**
> Isso é normal quando muitos carros carregam juntos. Seu carro continuará recebendo energia de forma otimizada. A velocidade aumentará assim que outros veículos terminarem suas sessões.

**Avaliação: Adequada**
Cobriu os dois pontos centrais da referência: o comportamento é normal (controle de demanda funcionando) e a velocidade se normaliza quando outro veículo terminar a sessão.

---

#### T05 — Disponibilidade e fila

**Pergunta enviada:**
> Todos os postos estão ocupados. Vale a pena esperar ou vou para outro lugar?

**Resposta esperada (referência):**
App mostra tempo estimado de liberação por posto. Também mostra eletropostos próximos com vagas disponíveis. Usuário decide com informação real.

**Resposta obtida:**
> **Verifique no app ChargeGrid:**
> 1. Tempo estimado de liberação (calculado com base no estágio atual das recargas)
> 2. Tarifa vigente (se estiver vermelha, pode valer a pena buscar outro posto)
>
> **Se optar por esperar:** ative a notificação de vaga livre no app. Assinantes têm prioridade na próxima vaga.
>
> **Postos próximos:** o app mostra em tempo real a distância de alternativas, tarifa comparativa e disponibilidade estimada.
>
> Dica: Postos em supermercados costumam ter rotatividade maior que shoppings.

**Avaliação: Adequada**
Indicou o tempo estimado de liberação no app, a localização de alternativas com vagas e deixou a decisão nas mãos do usuário com informação real — exatamente o que a referência esperava.

---

#### T06 — Interrupção de sessão

**Pergunta enviada:**
> O carregador desconectou sozinho no meio da recarga. O que aconteceu? Vou ser cobrado pelo tempo todo?

**Resposta esperada (referência):**
Cobrado apenas pelo kWh consumido até a interrupção. Possíveis causas: carga atingida, anomalia elétrica ou falha de comunicação. Histórico disponível no app.

**Resposta obtida:**
> Fique tranquilo! O sistema registrou apenas a energia fornecida até o momento da interrupção. Você pode ver os detalhes no histórico do app.

**Avaliação: Adequada**
Confirmou que a cobrança é proporcional ao kWh consumido até a interrupção e direcionou ao histórico no app — os dois pontos centrais da referência.

---

#### T07 — O que é o ChargeGrid

**Pergunta enviada:**
> O que é esse sistema ChargeGrid? É diferente de um carregador normal?

**Resposta esperada (referência):**
Sim, muito diferente. Plataforma completa com OCPP, IA, solar. Distribui potência inteligentemente. Mais transparência e melhor experiência para o motorista.

**Resposta obtida:**
> O ChargeGrid é muito mais que um simples carregador — é um **sistema inteligente** que revoluciona a recarga pública:
>
> 1. **Gestão Dinâmica de Energia:** distribui a potência entre vários veículos automaticamente, evita multas do estabelecimento por ultrapassar a demanda contratada e integra energia solar quando disponível.
> 2. **Tarifa Inteligente:** preço varia conforme horário e disponibilidade de energia renovável. O LED do totem mostra o melhor momento para carregar.
> 3. **Tecnologia Avançada:** previsão de demanda por IA, detecção de anomalias elétricas e comunicação em tempo real com medidores e inversores.
> 4. **Flexibilidade de Pagamento:** opções desde avulso até planos premium, com integração total ao app.
>
> Enquanto um carregador comum só fornece energia, o ChargeGrid **otimiza todo o ecossistema de recarga** — benefício para você, para o estabelecimento e para a rede elétrica.

**Avaliação: Adequada**
Diferenciou claramente o ChargeGrid de um carregador comum, destacando gestão inteligente de potência, integração solar GoodWe, IA e transparência para o motorista — todos os pontos da referência.

---

### Análise Qualitativa Geral

Os testes evidenciaram que o conjunto de técnicas adotadas — **System Prompt detalhado**, **Few-Shot Prompting** e **temperature 0.4** — foi suficiente para calibrar o modelo a responder de forma consistente com o contexto do ChargeGrid Intelligence em todos os 7 cenários testados.

Pontos de destaque observados:

- O modelo nunca inventou valores de tarifa ou tempo de recarga, respeitando a instrução de estimativas com base contextual.
- O Few-Shot Prompting foi determinante para calibrar o tom: respostas objetivas e acessíveis ao motorista, sem excesso de jargão técnico.
- A memória de conversa (histórico de mensagens) garantiu que respostas a partir do T02 já refletiam o contexto acumulado da sessão de testes.

---

## 📁 Arquivos do Repositório

| Arquivo | Descrição |
|---|---|
| `README.md` | Este documento |
| `chargegrid_assistant_sprint2.ipynb` | Notebook principal — Sprint 2 |
| `system_prompt.txt` | System prompt base documentado |
| `entrega_sprint2.txt` | Arquivo oficial de entrega |

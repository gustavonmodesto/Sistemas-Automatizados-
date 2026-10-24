# TP05 — Sistemas sequenciais e GRAFCET/SFC

**UC Sistemas Automatizados** · Prof. Calvetti · Engenharia de Software — USJT
**Aluno:** Gustavo Nunes Modesto
(RA 823126987) · 
**Data:** 26/09/2026

## Objetivo

Modelar e validar sistemas sequenciais utilizando GRAFCET (IEC 60848) no diagrams.net. As atividades incluem identificar os elementos do GRAFCET (atividade 23), desenvolver um semáforo temporizado (atividade 24, obrigatória), modelar uma porta automática com emergência (atividade 25), representar uma esteira separadora (atividade 26) e verificar manualmente o funcionamento dos modelos (atividade 27).

## Artefatos

| Arquivo | Conteúdo |
|---|---|
| `grafcet.drawio` | Arquivo editável com 5 páginas (A23, A24, A25, A26 e exemplo da aula) |
| `semaforo.png` | atividade 24 — **obrigatória** |
| `reservatorio.png` | atividade 23 — identificação dos elementos |
| `porta.png` | atividade 25 — porta automática com emergência |
| `esteira.png` | atividade 26 — esteira separadora |
| `exemplo-esteira-atuador.png` | exemplo desenvolvido em aula (esteira com atuador separador) |
| `casos_teste.csv` | 39 cenários: funcionamento normal, limites temporais e falhas, com resultados esperados, obtidos e situação |
| `percurso_semaforo.csv` | atividade 27 — percurso manual passo a passo |
| `NOTAS.md` | limitações, hipóteses e possíveis melhorias |

Para editar: <https://app.diagrams.net> → *Arquivo → Abrir de → Dispositivo* → `grafcet.drawio`. Cada atividade está organizada em uma página (abas inferiores). Para exportar: *Arquivo → Exportar como → PNG*, selecionando todas as páginas ou exportando individualmente.

## Convenções

| Elemento | Representação | Regra |
|---|---|---|
| Etapa | quadrado numerado | identificador único |
| Etapa inicial | quadrado com **borda dupla** | indica a etapa ativa na inicialização |
| Ação | retângulo amarelo à direita, conectado à etapa | tag = valor, válida enquanto a etapa estiver ativa |
| Transição | barra horizontal preta (T0, T1, …) | sempre posicionada entre etapas |
| Receptividade | texto azul à direita da barra | expressão booleana ou temporal testável |
| Temporização | `8 s/X0` | verdadeira quando a etapa E0 permanece ativa por pelo menos 8 s |
| Operadores | `·` = AND, `+` = OR, `¬` = NOT | |
| Etapa de segurança/falha | quadrado vermelho (E90, E99) | todas as saídas perigosas permanecem em 0 |

## Atividade 24 (obrigatória) — Semáforo veicular

![Semáforo](semaforo.png)

| Etapa | Situação | Ação |
|---|---|---|
| E0 (inicial) | vermelho | VERMELHO = 1 |
| E1 | verde | VERDE = 1 |
| E2 | amarelo | AMARELO = 1 |

| Transição | De → para | Receptividade |
|---|---|---|
| T0 | E0 → E1 | 8 s/X0 |
| T1 | E1 → E2 | 10 s/X1 |
| T2 | E2 → E0 | 3 s/X2 |

| Caso | Etapa inicial | Tempo aplicado | Esperado | Obtido | Situação |
|---|---|---|---|---|---|
| S01 | E0 | 7,9 s | permanece E0 (vermelho) | E0 | PASSA |
| S02 | E0 | 8,0 s | E1 (verde) | E1 | PASSA |
| S03 | E1 | 9,9 s | permanece E1 | E1 | PASSA |
| S04 | E1 | 10,0 s | E2 (amarelo) | E2 | PASSA |
| S05 | E2 | 2,9 s | permanece E2 | E2 | PASSA |
| S06 | E2 | 3,0 s | retorna E0 (vermelho) | E0 | PASSA |

Critérios de aceitação: apenas uma cor ativa por etapa ✔ · ciclo fechado ✔ · tempos definidos ✔ · etapa inicial identificada ✔. Ciclo completo = 21 s.

### Atividade 27 — Percurso manual (semáforo)

| Passo | Marcação ativa | Tempo na etapa | Transição | Nova marcação |
|---|---|---|---|---|
| 0 | E0 | 7,9 s | nenhuma | E0 |
| 1 | E0 | 8,0 s | T0 | E1 |
| 2 | E1 | 9,9 s | nenhuma | E1 |
| 3 | E1 | 10,0 s | T1 | E2 |
| 4 | E2 | 2,9 s | nenhuma | E2 |
| 5 | E2 | 3,0 s | T2 | E0 |

Em cada passo foram verificados: (1) a marcação ativa, (2) as entradas e os tempos definidos, (3) a habilitação **antes** da receptividade e (4) o registro da nova marcação. Os percursos das demais atividades estão disponíveis em `casos_teste.csv`.

## Atividade 23 — Identificação dos elementos (reservatório)

![Reservatório](reservatorio.png)

| ID | Etapa | Ação | Transição de saída |
|---|---|---|---|
| E0 (inicial) | aguardando | todas as válvulas fechadas | T0: START · NIVEL_BAIXO · ¬FALHA_NIVEL |
| E1 | enchendo | VALVULA_ENTRADA = 1 | T1: NIVEL_ALTO |
| E2 | misturando | MISTURADOR = 1 | T2: 30 s/X2 |
| E3 | drenando | VALVULA_SAIDA = 1 | T3: NIVEL_BAIXO → E0 |

**Condição de falha:** `FALHA_NIVEL = NIVEL_ALTO · NIVEL_BAIXO`. Quando os dois sensores estão ativos ao mesmo tempo, ocorre uma condição fisicamente incoerente. Por segurança, o ciclo não é iniciado (caso R07).

## Atividade 25 — Porta automática com emergência

![Porta](porta.png)

| Etapa | Ação | Saída |
|---|---|---|
| E0 fechada (inicial) | aguardar presença | ABRIR = 0; FECHAR = 0 |
| E1 abrindo | realizar abertura | ABRIR = 1 |
| E2 aberta | permanecer aberta por 5 s | ABRIR = 0; FECHAR = 0 |
| E3 fechando | realizar fechamento | FECHAR = 1 |
| E99 segura | interromper movimento | ABRIR = 0; FECHAR = 0; ALARME = 1 |

| Transição | Receptividade | Finalidade |
|---|---|---|
| T0: E0 → E1 | PRESENCA · ¬EMERGENCIA | iniciar abertura |
| T1: E1 → E2 | FC_ABERTA · ¬EMERGENCIA | confirmar abertura |
| T2: E2 → E3 | 5 s/X2 · ¬PRESENCA · ¬EMERGENCIA | iniciar fechamento |
| T3: E3 → E0 | FC_FECHADA · ¬PRESENCA · ¬EMERGENCIA | confirmar fechamento |
| T4: E3 → E1 | PRESENCA · ¬EMERGENCIA | reabrir ao detectar presença |
| T90…T93: E0…E3 → E99 | EMERGENCIA | priorizar estado seguro |
| T99: E99 → E0 | ¬EMERGENCIA · RESET · FC_FECHADA | permitir retomada controlada |

Regras adotadas: (1) todas as transições normais incluem `¬EMERGENCIA`, evitando conflitos com T90…T93; (2) T3 possui `¬PRESENCA`, tornando T3 e T4 exclusivas e garantindo a reabertura ao detectar presença; (3) desativar a emergência sozinho **não** movimenta a porta, sendo necessário RESET e confirmação de porta fechada.

| Caso | Estado inicial | Evento | Esperado | Obtido | Situação |
|---|---|---|---|---|---|
| P01 | E0 | PRESENCA = 1 | E1; ABRIR = 1 | E1 | PASSA |
| P02 | E1 | FC_ABERTA = 1 | E2; movimento parado | E2 | PASSA |
| P03 | E3 | PRESENCA = 1 | E1; reabrir | E1 | PASSA |
| P04 | E1 | EMERGENCIA = 1 | E99; saídas = 0 | E99 | PASSA |
| P05 | E99 | RESET = 1 e emergência ainda ativa | permanecer E99 | E99 | PASSA |
| P06 | E99 | RESET = 1; emergência = 0; FC_FECHADA = 1 | retornar E0 | E0 | PASSA |

Outros 6 casos (P07–P12), envolvendo o limite de 5 s, presença mantendo a porta aberta, retomada sem RESET e prioridade da emergência, estão registrados em `casos_teste.csv`.

## Atividade 26 — Esteira separadora

![Esteira](esteira.png)

| Condição (após 0,5 s em E1) | Caminho | Ação |
|---|---|---|
| TIPO_A · ¬TIPO_B | ramo A (E2 → E3) | DESVIO_A |
| ¬TIPO_A · TIPO_B | ramo B (E4 → E5) | DESVIO_B |
| TIPO_A · TIPO_B | falha (E90) | interromper movimentos e sinalizar inconsistência |
| ¬TIPO_A · ¬TIPO_B | espera | nenhuma transição é acionada |

| Etapa | Ação | | Entrada | Significado |
|---|---|---|---|---|
| E0 transportar (inicial) | M_EST = 1 | | S_PECA | peça na estação |
| E1 parar e ler | M_EST = 0; LEITURA = 1 | | TIPO_A / TIPO_B | identificação do tipo |
| E2 / E4 desviar | DESVIO_A / DESVIO_B = 1 | | FC_A_AV / FC_B_AV | atuador avançado |
| E3 / E5 retornar | DESVIO_x = 0 | | FC_A_RET / FC_B_RET | atuador recolhido |
| E6 liberar | M_EST = 1 | | RESET | reconhecimento do operador |
| E90 falha | M_EST = 0; desvios = 0; ALARME = 1 | | | |

Temporização: 0,5 s de estabilização em E1 antes da leitura. A esteira só volta a funcionar após a confirmação do atuador recolhido (T6/T7). Além disso, T0 exige que ambos os atuadores estejam recolhidos.

| Teste | Leitura A | Leitura B | Caminho | Esperado | Obtido | Situação |
|---|---|---|---|---|---|---|
| E01 | 0 | 0 | espera | sem desvio (E1) | E1 | PASSA |
| E02 | 1 | 0 | A | DESVIO_A = 1 (E2) | E2 | PASSA |
| E03 | 0 | 1 | B | DESVIO_B = 1 (E4) | E4 | PASSA |
| E04 | 1 | 1 | falha | movimentos = 0 (E90) | E90 | PASSA |
| E05 | retorno não confirmado | – | bloqueio | esteira = 0 (permanece E3) | E3 | PASSA |

## Resultados

Os 39 cenários registrados em `casos_teste.csv` foram executados seguindo as regras de evolução do GRAFCET: habilitação e receptividade, disparo simultâneo e atualização da marcação em uma única mudança. **Todos os testes passaram.**

O semáforo apresentou o comportamento esperado, sem necessidade de correções. Já na porta automática, a primeira versão de T3 não incluía `¬PRESENCA`, permitindo que T3 e T4 fossem verdadeiras simultaneamente quando a porta atingia o fim de curso com alguém passando. A condição foi adicionada para garantir que as transições fossem exclusivas.

## Checklist de aceitação

- [x] Etapa inicial identificada com borda dupla
- [x] Todas as etapas possuem identificação e função definida
- [x] Todas as transições possuem receptividade testável
- [x] Seleções sem ambiguidades (condições exclusivas)
- [x] Paralelismos com sincronização coerente (não utilizados nestes modelos)
- [x] Emergência direcionada para ações seguras
- [x] Testes abrangendo funcionamento normal, limites e falhas

## Active recall

1. **Mesma entrada, ações diferentes:** ocorre quando o sistema está em estados diferentes, pois o estado mantém o histórico relevante do funcionamento.
2. **Disparo de transição:** acontece quando todas as etapas imediatamente anteriores estão ativas **e** a receptividade é verdadeira.
3. **Seleção perigosa:** duas receptividades verdadeiras simultaneamente, sem prioridade definida, podem gerar comportamento ambíguo.
4. **Convergência paralela:** exige a ativação de todas as etapas finais dos ramos, além da receptividade verdadeira.
5. **GRAFCET × SFC:** GRAFCET é utilizado para especificação funcional (IEC 60848), enquanto SFC estrutura programas de CLP (IEC 61131-3).
6. **Fronteiras temporais:** erros de comparação (`>` × `>=`) podem ser identificados testando imediatamente antes, exatamente no limite e logo após o tempo definido.

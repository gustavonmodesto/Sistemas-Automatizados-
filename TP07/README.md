# TP07 — Programação sequencial de CLP (temporização e contagem)

**UC Sistemas Automatizados** · Professor. Calvetti · Engenharia de Software — USJT
**Autor:** Gustavo Nunes Modesto (RA 823144057) · 
**Data:** 09/10/2026 · **Simulador:** app.plcsimulator.online

**Links (PLC Simulator Online):**
- Obrigatório + extensões 1 a 3 (TON, pisca, CTU): [abrir programa](https://app.plcsimulator.online/ES1GCcj5YrwsCx91R7rU)
- Extensão 4 (GRAFCET do enchimento em Ladder): [abrir programa](https://app.plcsimulator.online/kdpEjSv3irJ1KJjogAez)

**Links (PLC Simulator Online):**

- Obrigatório + extensões 1 a 3 (TON, pisca, CTU): [abrir programa](https://app.plcsimulator.online/ES1GCcj5YrwsCx91R7rU) ([https://app.plcsimulator.online/ES1GCcj5YrwsCx91R7rU](https://app.plcsimulator.online/ES1GCcj5YrwsCx91R7rU))
- Extensão 4 (GRAFCET de enchimento em Ladder): [abrir programa](https://app.plcsimulator.online/kdpEjSv3irJ1KJjogAez) ([https://app.plcsimulator.online/kdpEjSv3irJ1KJjogAez](https://app.plcsimulator.online/kdpEjSv3irJ1KJjogAez))

## Objetivo

Desenvolver e testar programas Ladder a partir de requisitos sequenciais, utilizando temporizador **TON** para atraso de partida, sinal intermitente com memória de fase, contador **CTU** com detecção de borda, zeragem com prioridade definida e conversão de um **GRAFCET** de enchimento para Ladder.

## Estrutura

```
TP07/
├── README.md
├── programa_ladder.md          ← programa comentado, tags, ST equivalente e decisões
├── casos_teste.csv             ← 25 casos: esperado, obtido e situação
├── planilha_ciclos.csv         ← registro do ciclo de enchimento (contingência)
├── traco_pisca.csv             ← LAMPADA a cada 0,5 s durante 6 s
├── tabela_eventos_lote.csv     ← uma borda por peça
├── diagramas/                  ← os quatro programas Ladder
└── capturas/                   ← imagens do simulador
```

## Tags e prioridades

A tabela completa está disponível em [`programa_ladder.md`](programa_ladder.md). Foram definidas as seguintes prioridades:

- **Permissivos dominam:** `EMG_OK` e `STOP_OK` (1 = normal) ficam em série antes de qualquer temporização ou acionamento de movimento.
- **RESET_LOTE tem prioridade sobre a contagem** quando ocorre simultaneamente à detecção de uma nova peça.
- **SET/RESET do pisca:** cada fase possui seu próprio temporizador, e a inversão acontece uma única vez a cada conclusão.

## Exercício obrigatório — MOTOR_DELAY



TON: **IN** = condição de habilitação, **PT** = 3 s, **ET** = tempo decorrido e **Q** = 1 quando ET atinge PT.

| ID  | Ação                                      | Esperado                               | Obtido                     | Situação |
| --- | ----------------------------------------- | -------------------------------------- | -------------------------- | -------- |
| T01 | START por 2 s                             | Q=0 e MOTOR_DELAY=0 (ET = 2,0 s)       | Q=0; MOTOR_DELAY=0; ET=2,0 | PASSA    |
| T02 | START por 3 s                             | Q=1 e MOTOR_DELAY=1                    | Q=1; MOTOR_DELAY=1; ET=3,0 | PASSA    |
| T03 | STOP_OK=0 durante ET                      | ET zera e saída=0                      | ET=0,0; MOTOR_DELAY=0      | PASSA    |
| T04 | EMG_OK=0 com saída ligada                 | saída=0                                | antes 1, depois 0          | PASSA    |
| T05 | Liberar falha sem START                   | saída permanece 0                      | MOTOR_DELAY=0; ET=0,0      | PASSA    |
| T06 | Exercício resolvido 1: START cai em 2,5 s | TON retorna ao estado inicial; bomba 0 | ET=0; saída=0              | PASSA    |
| T07 | Fronteira 2,9 s / 3,0 s                   | 0 / 1                                  | 0 / 1                      | PASSA    |

## Extensão 1 — Sinal intermitente (1 s / 1 s)



| Tempo (s) | 0,5 | 1,0 | 1,5 | 2,0 | 2,5 | 3,0 | 3,5 | 4,0 | 4,5 | 5,0 | 5,5 | 6,0 |
| --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| LAMPADA   | 0   | 1   | 1   | 0   | 0   | 1   | 1   | 0   | 0   | 1   | 1   | 0   |

Foram realizadas seis inversões em seis segundos, uma a cada conclusão do temporizador. No PLC Simulator Online, as inversões ocorreram em 1,08 / 2,07 / 3,13 / 4,11 / 5,18 / 6,16 s, apresentando aproximadamente 0,1 s de atraso devido à varredura seguinte à conclusão do timer.

Por isso, ao verificar exatamente t = 1,0 s, a lâmpada ainda aparece apagada. Já no meio de cada segundo, o comportamento corresponde à sequência da tabela.

A memória PISCA armazena a fase atual. Quando um temporizador conclui a contagem, o SET/RESET altera a fase e sua habilitação é desativada na varredura seguinte, evitando novas inversões indevidas.

## Extensão 2 — Lote completo (CTU, PV = 5)



| Peça                                  | Bordas detectadas | CV após o evento        | LOTE_OK |
| ------------------------------------- | ----------------- | ----------------------- | ------- |
| 1                                     | 1                 | 1                       | 0       |
| 2                                     | 1                 | 2                       | 0       |
| 3                                     | 1                 | 3                       | 0       |
| 4                                     | 1                 | 4                       | 0       |
| 5                                     | 1                 | 5                       | 1       |
| sensor mantido em 1 por 50 varreduras | nenhum novo pulso | 1 (não conta novamente) | —       |

Cada peça mantém o sensor em 1 durante 3 varreduras, mas gera apenas um incremento no contador.

A detecção de borda utiliza `P_PECA = S_PECA AND NOT S_PECA_ANT`, com `S_PECA_ANT` atualizado **após** o cálculo. Dessa forma, a ordem dos rungs é importante para o funcionamento correto.

## Extensão 3 — Zeragem controlada

| ID  | Situação                         | Esperado                 | Obtido    | Situação |
| --- | -------------------------------- | ------------------------ | --------- | -------- |
| Z01 | RESET com CV=0                   | CV=0                     | CV=0      | PASSA    |
| Z02 | RESET com CV=3                   | CV=0                     | 3 → 0     | PASSA    |
| Z03 | RESET com CV=5 e LOTE_OK=1       | CV=0; LOTE_OK=0          | 5/1 → 0/0 | PASSA    |
| Z04 | RESET e borda na mesma varredura | CV=0 (RESET prioritário) | CV=0      | PASSA    |
| Z05 | Nova peça depois do reset        | CV=1                     | CV=1      | PASSA    |

**Prioridade definida: RESET domina a contagem simultânea**, conforme o comportamento do CTU na IEC 61131-3.

## Extensão 4 — Conversão do GRAFCET de enchimento para Ladder



| ID | Requisito                                                       |
| -- | --------------------------------------------------------------- |
| R1 | A esteira transporta os recipientes até o sensor.               |
| R2 | Ao detectar o recipiente, a esteira para e o enchimento começa. |
| R3 | A válvula permanece aberta durante 4 s.                         |
| R4 | Após o enchimento, a esteira libera o recipiente.               |
| R5 | Cada recipiente finalizado incrementa o contador do lote.       |
| R6 | Ao atingir 6, LOTE_OK é ativado e impede um novo ciclo.         |
| R7 | RESET_LOTE zera a contagem e libera a produção.                 |

Registro do ciclo normal (`planilha_ciclos.csv`, varredura de 0,1 s):

| Instante      | Etapa | M_EST | Y_VALV | T_ENCH.ET | CV | LOTE_OK |
| ------------- | ----- | ----- | ------ | --------- | -- | ------- |
| t0 sem peça   | E0    | 1     | 0      | 0,0       | 0  | 0       |
| t1 S_PECA=1   | E1    | 0     | 1      | 0,1       | 0  | 0       |
| t2 enchendo   | E1    | 0     | 1      | 2,1       | 0  | 0       |
| t3 ET = 4 s   | E1    | 0     | 1      | 4,0       | 0  | 0       |
| t3+ liberando | E2    | 1     | 0      | 0,0       | 0  | 0       |
| t4 S_PECA cai | E0    | 1     | 0      | 0,0       | 1  | 0       |
| t5 novo ciclo | E0    | 1     | 0      | 0,0       | 1  | 0       |

Em t3, o temporizador já atingiu 4 s, mas a transição acontece somente na varredura seguinte. Isso ocorre porque as condições são verificadas antes da execução do bloco de temporização, gerando a latência de uma varredura observada na Semana 6.

| ID  | Falha / caso                  | Resposta correta                               | Obtido                      | Situação |
| --- | ----------------------------- | ---------------------------------------------- | --------------------------- | -------- |
| G01 | ciclo normal                  | retorna a E0; CV=1                             | E0; CV=1                    | PASSA    |
| G02 | duas etapas ativas?           | nunca                                          | 0 ocorrências               | PASSA    |
| G03 | EMG_OK=0 durante o enchimento | desenergizar movimentos                        | M_EST=0; Y_VALV=0           | PASSA    |
| G04 | S_PECA preso em 1             | permanecer em E2                               | E2                          | PASSA    |
| G05 | TON não concluído             | permanecer em E1                               | E1; Y_VALV=1                | PASSA    |
| G06 | CV=6                          | bloquear novo ciclo                            | CV=6; LOTE_OK=1; fica em E0 | PASSA    |
| G07 | RESET_LOTE                    | zerar e liberar                                | CV=0; avança para E1        | PASSA    |
| G08 | S_PECA oscila em E1           | receptividade fora da etapa não provoca avanço | E1                          | PASSA    |

## Evidências

- Os links dos dois programas estão disponíveis no início do documento. A extensão 4 foi desenvolvida separadamente porque utiliza as mesmas tags `S_PECA`, `C_LOTE` e `LOTE_OK` das extensões 2 e 3, porém com PV = 6.

- [`casos_teste.csv`](casos_teste.csv): contém 25 casos com resultados esperados, obtidos e situação. **Todos passaram.** Os resultados foram obtidos pela execução dos programas varredura a varredura, e os testes também foram repetidos no PLC Simulator Online, conforme a coluna `conferido_plc_simulator`.

Valores registrados no simulador: T01 ET = 2,05 s com saída 0; T02 ET = 3,00 s com saída 1; T07 ET = 2,90 s → 0 e 3,00 s → 1; C01 5 peças → 5 pulsos, CV = 5, LOTE_OK = 1; C02 sensor mantido por 3 s → CV = 1; Z04 RESET simultâneo à borda → CV = 0; G06 → CV = 6, LOTE_OK = 1, permanece em E0; G07 → CV = 0 e avança para E1.

- Capturas disponíveis em `capturas/`:

| Arquivo                                                 | Estado                                                |
| ------------------------------------------------------- | ----------------------------------------------------- |
| `ton_antes_pt.png`                                      | START = 1, ET ≈ 1,6 s, MOTOR_DELAY = 0                |
| `ton_depois_pt.png`                                     | ET = 3 s, Q = 1, MOTOR_DELAY = 1                      |
| `ton_interrompido.png`                                  | STOP_OK = 0 durante a contagem: ET = 0                |
| `pisca_lampada_acesa.png` / `pisca_lampada_apagada.png` | duas fases do pisca, com T_ON / T_OFF contando        |
| `ctu_lote.png`                                          | CV = 5 e LOTE_OK = 1 com S_PECA mantido em 1          |
| `ctu_reset.png`                                         | RESET_LOTE = 1, zerando CV                            |
| `sequencia_e1.png`                                      | enchimento: E1 ativo, Y_VALV = 1                      |
| `sequencia_e2_liberando.png`                            | E2 ativo, M_EST = 1, aguardando a saída do recipiente |

- `planilha_ciclos.csv`: registro documental do ciclo de enchimento, utilizado como contingência prevista no slide 67.

## Contingência

| Necessidade         | Ferramenta                                 | Limite                                   |
| ------------------- | ------------------------------------------ | ---------------------------------------- |
| TON e TOF dirigidos | Virtual Labs PLC                           | experimento delimitado                   |
| Contador up/down    | Virtual Labs PLC                           | experimento delimitado                   |
| Sequência livre     | planilha de ciclos (`planilha_ciclos.csv`) | representação documental, não executável |

## Active recall

1. **TON × TOF:** o TON atrasa a energização (Q liga após PT com IN = 1), enquanto o TOF atrasa a desenergização (Q desliga após PT com IN = 0).
2. **Contador recebe eventos:** um sensor pode permanecer em 1 por várias varreduras. Contar diretamente esse nível faria a mesma peça ser contabilizada várias vezes.
3. **GE em vez de EQ:** GE permanece verdadeiro quando CV ultrapassa PV, enquanto EQ falha caso a contagem não corresponda exatamente ao valor definido.
4. **SET e RESET como memória:** SET grava 1 e mantém esse valor mesmo após a condição deixar de ser verdadeira; RESET grava 0. A prioridade em situações simultâneas precisa ser definida.
5. **O que deve ser preservado na conversão do GRAFCET:** etapas representadas por bits, etapa inicial, receptividades nos rungs de transição incluindo a etapa anterior, disparos coerentes e ações vinculadas às etapas.

## Limites

A simulação permite verificar o comportamento lógico dos programas no ambiente utilizado. Entretanto, não comprova o tempo real de varredura de um CLP físico, a precisão dos temporizadores de hardware, possíveis ruídos ou repiques de sensores reais, nem a segurança funcional do sistema.

A tag `EMG_OK` representa um permissivo lógico e não substitui uma função de segurança certificada.

Referências: IEC 61131-3:2025, documentação do PLC Simulator Online e Virtual Labs PLC.

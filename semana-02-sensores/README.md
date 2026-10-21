**UC Sistemas Automatizados** · Prof. Calvetti · Engenharia de Software — USJT

**Aluno:** Gustavo Nunes Modesto
(RA 823126987) · 

**Data:** 13/10/2026

**Projeto no Wokwi:** desenvolvido e testado no Wokwi, com capturas disponíveis em `evidencias/`. O projeto não foi publicado. Para reproduzir a simulação, basta utilizar os arquivos `diagram.json` e `sketch.ino` em um novo projeto Arduino Uno, seguindo as instruções abaixo.

## Objetivo

Desenvolver no Wokwi uma **estação de monitoramento** utilizando uma entrada digital (botão), uma entrada analógica (potenciômetro), um sensor de temperatura (DHT22) e um LED para sinalização. O programa identifica quatro estados: **NORMAL, ATENÇÃO, ALARME e FALHA DE SENSOR**, exibindo as leituras e o diagnóstico no Monitor Serial.

## Estrutura

```
semana-02-sensores/
├── README.md
├── sketch.ino            ← programa final (prioridade + histerese)
├── diagram.json          ← circuito do Wokwi
├── libraries.txt         ← bibliotecas utilizadas
├── etapas/               ← etapas de desenvolvimento (1 a 5)
├── evidencias/           ← capturas de tela do Wokwi
└── testes/
    └── casos-de-teste.md
```

## Relação entre componente didático e função industrial

| Variável do processo | Sensor didático | Entrada no Arduino | Resposta do sistema | Equivalente industrial |
|---|---|---|---|---|
| Presença / posição (discreta) | Pushbutton | D2 (digital, `INPUT_PULLUP`) | acende o LED em D8 | fim de curso, sensor indutivo, pressostato |
| Variável contínua (ex.: pressão) | Potenciômetro | A0 (analógica, 0–1023) | NORMAL / ATENÇÃO / ALARME | transmissor analógico 4–20 mA |
| Temperatura | DHT22 | D4 (digital codificada) | ATENÇÃO / ALARME / FALHA | RTD Pt100 ou termopar + transmissor |
| Sinalização | LED + 220 Ω | D8 (saída digital) | — | sinaleiro ou relé |
| Controlador | Arduino Uno | — | — | CLP ou controlador embarcado |

## Mapa de pinos e circuito

| Componente | Pino do Arduino | Ligação complementar | Finalidade |
|---|---|---|---|
| Pushbutton | D2 | outro terminal em GND | entrada digital com pull-up interno |
| LED + resistor | D8 | D8 → 220 Ω → ânodo; cátodo → GND | limitar corrente e realizar sinalização |
| Potenciômetro | A0 | VCC → 5 V; GND → GND; SIG → A0 | gerar tensão variável |
| DHT22 | D4 | VCC → 5 V; GND → GND; DATA → D4 | fornecer a temperatura simulada |

Com `INPUT_PULLUP`, a lógica física funciona de forma invertida: **liberado = HIGH e pressionado = LOW**. Para interpretar corretamente o botão, foi utilizada a expressão `detectado = digitalRead(BTN) == LOW`.

## Regras de decisão (em ordem de prioridade)

| Prioridade | Estado | Regra |
|---|---|---|
| 1 | FALHA DE SENSOR | temperatura inválida (`isnan(t)`) |
| 2 | ALARME | alarme de temperatura (liga em t ≥ 40 °C, desliga em t ≤ 37 °C) **ou** pot ≥ 750 |
| 3 | ATENÇÃO | t ≥ 30 °C **ou** pot ≥ 400 |
| 4 | NORMAL | nenhuma das condições anteriores |

| Variável | Normal | Atenção | Alarme / Falha |
|---|---|---|---|
| Potenciômetro | 0–399 | 400–749 | 750–1023 |
| Temperatura | < 30 °C | 30 a < 40 °C | ≥ 40 °C (desliga em ≤ 37 °C) |
| DHT inválido | — | — | FALHA DE SENSOR |
| Botão | liberado | — | pressionado acende o LED |

A verificação de falha ocorre primeiro, pois uma leitura inválida não permite avaliar corretamente o processo. O estado NORMAL é utilizado quando nenhuma das demais condições é atendida.

## Como reproduzir no Wokwi

1. Acesse <https://wokwi.com> → **New Project** → **Arduino Uno**.
2. Abra a aba `diagram.json` e substitua o conteúdo pelo arquivo `diagram.json` desta pasta. O circuito será montado automaticamente. Caso prefira, também é possível realizar as ligações manualmente seguindo o mapa de pinos.
3. Na aba `sketch.ino`, cole o código disponível no arquivo `sketch.ino`.
4. No **Library Manager** (ou por meio do arquivo `libraries.txt`), adicione a biblioteca **DHT sensor library** (Adafruit) e, se necessário, **Adafruit Unified Sensor**.
5. Clique em ▶ para iniciar a simulação e abra o Monitor Serial configurado para 9600 bit/s. A cada segundo será exibida uma linha semelhante a:

   `BTN=LIBERADO | LED=OFF | POT=200 | TEMP=25.0 C | ESTADO=NORMAL`

6. Durante a execução, altere os valores do potenciômetro ou do DHT22 e realize os testes descritos em `testes/casos-de-teste.md`.

A pasta `etapas/` contém as versões utilizadas durante o desenvolvimento (digital → analógico → classificação → DHT22 → integrado sem histerese). Cada versão pode ser executada separadamente no Wokwi para verificar seu funcionamento.

## Casos de teste — Resumo

Todos os casos apresentados foram executados no Wokwi. As capturas estão disponíveis em [`evidencias/testes-integrados.png`](evidencias/testes-integrados.png).

![Circuito em execução](evidencias/circuito.png)

A relação completa dos testes, incluindo entradas digitais, limites, T1–T7, P1–P5 e histerese H1–H8, está disponível em [`testes/casos-de-teste.md`](testes/casos-de-teste.md).

| Caso | Pot. | Temp. | Botão | Esperado | Obtido | Situação |
|---|---|---|---|---|---|---|
| T1 | 200 | 25 °C | livre | NORMAL; LED off | NORMAL; LED off | Aprovado |
| T2 | 500 | 25 °C | livre | ATENÇÃO | ATENCAO | Aprovado |
| T3 | 800 | 25 °C | livre | ALARME | ALARME | Aprovado |
| T4 | 200 | 35 °C | livre | ATENÇÃO | ATENCAO | Aprovado |
| T5 | 200 | 45 °C | livre | ALARME | ALARME | Aprovado |
| T6 | 200 | 25 °C | pressionado | NORMAL; LED on | NORMAL; LED on | Aprovado |
| T7 | — | inválida | livre | FALHA DE SENSOR | FALHA DE SENSOR | Aprovado |

## Exercícios adicionais

### 1 — Prioridade de alarmes

A lógica de decisão foi organizada na função `classificar(t, bruto)`, responsável por retornar o estado do sistema.

A sequência dos comandos `if` estabelece a prioridade **FALHA > ALARME > ATENÇÃO > NORMAL**.

Os cinco casos adicionais (P1–P5) estão documentados em `testes/casos-de-teste.md`. O teste P5, com pot = 900 e temperatura inválida, demonstra que uma falha de instrumento possui prioridade sobre o alarme e não pode ser interpretada como uma condição normal.

### 2 — Histerese

Sem histerese, pequenas oscilações próximas de 40 °C poderiam provocar mudanças constantes entre ATENÇÃO e ALARME.

Para evitar isso, a variável global `alarmeTemperatura` mantém o estado anterior. O alarme é ativado quando **t ≥ 40 °C**, permanece ativo enquanto **t > 37 °C** e só é desligado quando **t ≤ 37 °C**.

Os testes H1–H8 e a sequência complementar 39,8 → 37,0 foram aprovados.

No caso H6, o resultado permanece em ATENÇÃO, e não NORMAL, porque a condição de atenção (t ≥ 30 °C) continua sendo atendida.

### 3 — Seleção industrial

**Processo escolhido:** monitoramento de uma câmara térmica industrial, considerando o estado da porta, a pressão da linha de circulação e a temperatura interna.

| Entrada didática | Mensurando | Tecnologia industrial | Faixa | Interface | Ambiente e justificativa |
|---|---|---|---|---|---|
| Pushbutton | porta fechada | chave fim de curso industrial ou sensor indutivo | 2 estados | 24 Vcc discreta | fim de curso é simples e resistente; o indutivo evita contato quando o alvo é metálico |
| Potenciômetro | pressão da linha | transmissor de pressão | 0–10 bar | 4–20 mA | sinal resistente a ruídos e adequado para transmissão a distância; 4 mA = zero vivo |
| DHT22 | temperatura interna | RTD Pt100 + transmissor | 0–150 °C | 4–20 mA | Pt100 apresenta boa estabilidade na faixa; o transmissor padroniza o sinal |

| Instrumento | Falha detectável | Falha não detectável apenas pela entrada |
|---|---|---|
| Sensor da porta | circuito aberto (com diagnóstico elétrico) | sensor deslocado mecanicamente, mas ainda comutando |
| Transmissor de pressão | corrente fora da faixa diagnóstica (< 3,6 mA) | desvio dentro da faixa que ainda apresenta valores plausíveis |
| Pt100 + transmissor | ruptura do elemento identificada pelo transmissor | pequeno erro sistemático sem calibração ou comparação |

## Limitações da simulação

| O Wokwi demonstra | O Wokwi não valida |
|---|---|
| leitura de entradas digitais e analógicas | alcance, tempo de resposta real e tolerâncias |
| prioridade e funcionamento da lógica | ruído elétrico, aterramento e compatibilidade eletromagnética |
| acionamento e diagnóstico no Monitor Serial | montagem, cabeamento, invólucro e grau de proteção (IP) |
| testes de fronteira e rastreabilidade | calibração metrológica e segurança funcional (SIL/PL) |

A simulação permite verificar a **estrutura funcional** do sistema, incluindo leitura, processamento, atuação e diagnóstico. Entretanto, não comprova a adequação física dos instrumentos industriais apresentados na seleção.

## Active recall

1. **Sensor × transdutor × transmissor:** o sensor identifica ou reage a uma grandeza física; o transdutor converte uma forma de energia em outra (ex.: temperatura → resistência); o transmissor condiciona e padroniza o sinal para transmissão (ex.: 4–20 mA). Um Pt100 com transmissor reúne essas três funções.
2. **Digital não significa preciso:** o DHT22 fornece uma leitura digital, mas sua exatidão depende das características do sensor (±0,5 °C). O formato digital indica como a informação é transmitida, não a qualidade da medição.
3. **Resolução × repetibilidade:** resolução é a menor variação que um instrumento consegue indicar (ex.: 0,1 °C). Repetibilidade é a capacidade de apresentar resultados próximos quando a mesma condição é medida várias vezes. Um sensor pode ter boa resolução e indicar 25,3 / 25,9 / 25,1 para a mesma temperatura.
4. **Detectar metal:** o sensor indutivo é adequado porque detecta objetos metálicos sem contato físico e apresenta boa resistência a ambientes com poeira e óleo.
5. **O que o Wokwi não valida:** ruídos eletromagnéticos, aterramento, calibração, segurança funcional e outras condições físicas apresentadas na tabela de limitações.

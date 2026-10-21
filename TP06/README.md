# P06 – Partida de motor com selo (Ladder)

Atividade da Semana 6 de Sistemas Automatizados (CLP e linguagem Ladder).
Programa Ladder de partida e parada de um motor com retenção (selo), parada prioritária, emergência e sobrecarga, executado no PLC Simulator Online.

## Link do projeto

> **Preencher:** cole aqui o link gerado pelo botão de compartilhar do simulador.
> Link: https://app.plcsimulator.online/NwEEYdF8jLCZbGIcDwxD

Ferramenta: https://plcsimulator.online/

## Tags

| Tag | Tipo | Significado | Convenção lógica |
|---|---|---|---|
| START | entrada | botão de partida | 1 quando pressionado |
| STOP_OK | entrada | cadeia de parada saudável | 1 em condição normal |
| EMG_OK | entrada | emergência liberada | 1 em condição normal |
| OL_OK | entrada | sobrecarga saudável | 1 sem disparo |
| MOTOR | saída | contator do motor | 1 comandado |

## Lógica

Equação do rung:

```
MOTOR = (START OU MOTOR) E STOP_OK E EMG_OK E OL_OK
```

Diagrama (um único rung):

```
|--[STOP_OK]--[EMG_OK]--[OL_OK]--+--[START]--+--( MOTOR )--|
                                 |           |
                                 +--[MOTOR]--+
```

## Decisões de projeto

- **Parada antes do selo:** `STOP_OK`, `EMG_OK` e `OL_OK` ficam em série **antes** da divergência entre `START` e o contato `MOTOR`. Assim, qualquer condição não saudável interrompe os dois caminhos e o selo não consegue manter o motor ligado.
- **Convenção "saudável = 1":** os permissivos usam contato normalmente aberto. Em condição normal valem 1 e permitem a continuidade; ao falhar (0), derrubam o comando.
- **Sem religamento automático:** depois que uma falha cai e é liberada, o motor só volta com um novo `START`, porque o selo foi desfeito pela falha.
- **Uma única bobina `MOTOR`:** evita múltiplas escritas na mesma tag.

## Como testar

1. Abra o projeto e clique em **play** para iniciar a simulação.
2. Coloque `STOP_OK`, `EMG_OK` e `OL_OK` em **True**.
3. Siga os casos de `casos_teste.csv`, alterando os valores na tabela de variáveis e observando `MOTOR`.

## Resultados

Os casos T01 a T07 estão em [`casos_teste.csv`](casos_teste.csv), com esperado, obtido e situação. Todos os casos ficaram **Aprovados**: o resultado obtido coincidiu com o esperado.

| Cobertura | Casos |
|---|---|
| Partida e selo | T01, T02 |
| Parada | T03 |
| Emergência | T04, T05 |
| Sobrecarga e retomada | T06, T07 |

## Capturas de tela

Pasta `capturas/`:

| Arquivo | Estado mostrado |
|---|---|
| `01_partida.png` | T01: START=1 liga o MOTOR |
| `02_selo.png` | T02: START=0 e MOTOR continua 1 |
| `03_parada.png` | T03: STOP_OK=0 desliga o MOTOR |
| `04_emergencia.png` | T04: EMG_OK=0 desliga o MOTOR |
| `05_emergencia_bloqueia_partida.png` | T05: EMG_OK=0 e START=1, MOTOR continua 0 |
| `06_sobrecarga.png` | T06: OL_OK=0 e START=1, MOTOR continua 0 |
| `07_retomada.png` | T07: OL_OK volta a 1 e MOTOR continua 0 |

## Conclusão e limites da simulação

O programa cumpre o requisito funcional: o motor parte com `START`, se mantém pelo selo, e é desligado por `STOP_OK`, `EMG_OK` ou `OL_OK`, que também impedem uma nova partida enquanto a condição não estiver saudável.

Esta é uma validação **lógica em simulador acadêmico**. Ela não substitui análise de risco, hardware adequado e comissionamento da máquina. A emergência aqui é apenas uma condição de lógica padrão: uma função de emergência real exige arquitetura de segurança própria e **não** deve depender apenas de software padrão do CLP. Este trabalho **não** alega segurança certificada.

## Referências

- IEC 61131-3:2025
- LAMB, F. *Automação industrial na prática*
- PRUDENTE, F. *Automação industrial: CLP e programação*
- PLC Simulator Online – documentação

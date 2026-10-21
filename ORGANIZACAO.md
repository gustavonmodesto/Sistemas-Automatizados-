# Registro de organização

Base: `18721bcd634968705b4adf1d622bc266c7b26966` (main).

## Critérios

- Destinos conferidos pelo conteúdo (identidade do blob Git), pelas estruturas descritas nos READMEs e pelo repositório de referência de Lucas.
- Todos os arquivos técnicos atuais mantidos sem edição. Documentação de navegação atualizada e índice principal criado.
- READMEs de conteúdos diferentes, mesmo com nomes parecidos, mantidos como `README-referencia.md` nas respectivas atividades.
- Diagramas de circuito e dependências compartilhados pelas semanas 2 e 3 mantidos em ambas as pastas para uso independente.
- Nenhuma alteração na lógica dos programas ou nos resultados declarados.

## Duplicatas confirmadas

Nove cópias excedentes consolidadas. Cada linha possui conteúdo exatamente igual (mesmo blob Git).

| Origens | Destino único | Blob Git |
|---|---|---|
| `01_partida.png` + `capturas/01_partida.png` | `TP06/capturas/01_partida.png` | `d0fac0a15a8c019b42b88e0305b6bfa8c8750f64` |
| `02_selo.png` + `capturas/02_selo.png` | `TP06/capturas/02_selo.png` | `f136d4f59807e522ad8c5e4ee12da4d2455f327d` |
| `03_parada.png` + `capturas/03_parada.png` | `TP06/capturas/03_parada.png` | `a6c1555e6b2b9c5e463eb6bc117e53bc80162ba1` |
| `04_emergencia.jpeg` + `capturas/04_emergencia.jpeg` | `TP06/capturas/04_emergencia.jpeg` | `40821c3b1e9c0254ce0242695791322c5435e60a` |
| `05_emergencia_bloqueia_partida.png` + `capturas/05_emergencia_bloqueia_partida.png` | `TP06/capturas/05_emergencia_bloqueia_partida.png` | `999ea89bf321822a440807bd6f131b5a95c16e5a` |
| `06_sobrecarga.png` + `capturas/06_sobrecarga.png` | `TP06/capturas/06_sobrecarga.png` | `7e5422433a91e3584358c8f951a13be9c7551416` |
| `07_retomada.png` + `capturas/07_retomada.png` | `TP06/capturas/07_retomada.png` | `9192a6701fa6473a8f6052f9f8edae0ea4490bd8` |
| `T1.png` + `circuito.png` | `semana-02-sensores/evidencias/casos/T1.png` | `c578c9fc0dc7c129e84d9b5a7aa6905c0a53fb40` |
| `T6.png` + `testes-digitais.png` | `semana-02-sensores/evidencias/casos/T6.png` | `0a507f6c9ec9f704564f40b3dfdd4f723a5e07ab` |

## Preservação da atividade 6

Os nove arquivos do commit `eb51d2a1cadc17cdd3f81565e74b545ee4b36c43` foram preservados byte a byte em `TP06/`: sete capturas, CSV e README (com nome `README-entrega.md`).

O CSV da raiz já era da semana 4; foi movido para essa semana. O CSV original de Gustavo foi restaurado pelo blob `f6778e552ef62cdc36d9576e6ebea31864d1e115`.

A cópia `TP06/README-original.md` não foi removida: sua codificação de quebras de linha difere do README do commit original. A discrepância histórica `04_emergencia.png` versus arquivo real `04_emergencia.jpeg` foi documentada no índice, sem editar a entrega.

## Limitações encontradas

- O ZIP anexado não pôde ser baixado neste ambiente; esta revisão cobre os arquivos do GitHub e o histórico confirmado. Não se afirma equivalência ou completude em relação ao ZIP.
- A imagem `reservatorio.png` presente na raiz corresponde ao GRAFCET de TP05, não ao diagrama de fundamentos da semana 1. Foi movida para TP05. A imagem específica da semana 1 não estava presente; seu README de referência foi preservado integralmente e mantém essa referência pendente.
- A semana 2 não possuía seu README técnico nem seu programa integrado na versão analisada. As etapas e evidências disponíveis foram organizadas; o programa integrado da semana 3 não foi usado como substituto.
- O CSV da referência TP06 não estava presente. O CSV original de Gustavo não foi misturado com os testes dessa referência.
- Não foram executados simuladores, compilação Arduino ou testes físicos. A validação desta revisão é de organização, integridade dos blobs, preservação da entrega e caminhos locais.

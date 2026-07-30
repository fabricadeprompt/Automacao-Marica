# Protocolo Compartilhado (ESP-NOW)

Toda a comunicação entre os três nós roda sobre **ESP-NOW, canal fixo 2**
(`CANAL_SEGURANCA_PADRAO`, definido em `marica_protocol.h`) — não é negociado nem varre
canais, é fixado no rádio antes da inicialização.

## Fluxo de pacotes

```
Água  ──PKT_TELEMETRIA_AGUA──▶  Bomba  ──PKT_STATUS_COMPLETO──▶  Controle ──Wi-Fi──▶ Supabase
                                   ▲                                  │
                                   └──── CMD_LIGA_BOMBA / CMD_DESLIGA_BOMBA
                                          CMD_PING_CONTROLE / CMD_RESET_ERROS
                                          CMD_SET_NIVEIS ──────────────┘

Controle ──PKT_STATUS_CARDPUTER (push a cada 10s)──▶ Cardputer (modo Monitor)

Cardputer ──CMD_WEB_SERVER / CMD_OTA / CMD_REBOOT──▶ qualquer caixa
```

## `PacketStatusCompleto` (Bomba → Controle)

É o pacote mais denso do sistema — a Bomba agrega a telemetria repassada da Água com
seus próprios dados antes de subir para o Supabase.

| Campo | Descrição |
|---|---|
| `agua_distancia_cm` | Nível filtrado, repassado da Água |
| `agua_erro_sensor` | Falha física do sensor ultrassônico, repassada |
| `agua_ladrao_ativo` | Transbordo confirmado, repassado |
| `bomba_rele_estado` | Estado físico do relé K1 |
| `bomba_erro_bitmask` | Erros que ativamente bloqueiam/desligam a bomba (timeout, transbordo, falha PZEM) |
| `pzem_potencia_w`, `pzem_tensao_v`, `pzem_corrente_a`, `pzem_fp`, `pzem_energia_kwh` | Leituras instantâneas e acumuladas do PZEM-004T |
| `agua_offline` | Silêncio de rádio Água→Bomba por mais de 60s — distinto de erro de sensor real |
| `bomba_estado_bitmask` | Estados **informativos** (quarentena, bloqueio, modo forçado, relé colado) — nunca usados por decisão de segurança |
| `bomba_causa_desligamento` | Motivo do último desligamento (manual, nível cheio, timeout, transbordo, etc.) |

**Por que dois bitmasks separados de erro/estado?** `bomba_erro_bitmask` é lido por
decisões de segurança reais (intertravamento e desligamento automático);
`bomba_estado_bitmask` é puramente informativo para o dashboard. Separar os dois evita
que um campo adicionado só para exibição acabe influenciando, sem querer, uma decisão
de segurança.

## `PacketStatusCardputer` (Controle → Cardputer)

Push periódico (a cada 10s), não sob demanda — mesmo padrão do keep-alive
`CMD_PING_CONTROLE` já usado com a Bomba. Alimenta o modo Monitor do Cardputer (tela
de repouso, sempre ligada, fixada por ímã na porta da geladeira, mostrando o nível do
reservatório).

| Campo | Descrição |
|---|---|
| `nivel_pct` | 0-100, já calculado por `calcular_pct()` na Controle — fonte única de verdade, a mesma usada pelo servidor web local. O Cardputer não reimplementa a conversão distância→percentual |
| `nivel_distancia_cm` | Distância bruta (cm), só para exibir "xx cm" junto do percentual |
| `bomba_ligada` | Repassado da Controle |
| `modo_atual` | `MODO_AUTOMATICO` / `MODO_SEMIAUTOMATICO` |
| `agua_erro_sensor` | Repassado — sensor da Água com falha física |
| `agua_offline` | Repassado — silêncio de rádio Água→Bomba |
| `bomba_offline` | De `bomba_esta_offline()` — mesma fonte única já usada em outros pontos da Controle. Se `true`, o resto do pacote é dado cacheado, não corrente — o Cardputer trata como indisponível, nunca exibe como se fosse atual |

**Concorrência:** os 7 campos chegam num pacote só, mas são escritos pela task de
recepção ESP-NOW e lidos pelo `loop()` do Cardputer — dois contextos diferentes. O
Cardputer usa uma seção crítica (`portMUX`) tanto na escrita quanto na leitura, para
garantir que os 7 campos sejam sempre lidos como um conjunto consistente, nunca uma
mistura de pacote novo com pacote antigo.

## Regra de evolução do protocolo

O struct é compartilhado por firmwares que nem sempre são atualizados ao mesmo tempo —
cada caixa é flashada fisicamente em campo, não por OTA simultâneo. A extensão segue
duas regras:

1. **Campo novo sempre no final do struct** — um receptor com firmware antigo, ao
   receber um pacote maior, ainda passa no teste de tamanho mínimo e ignora o campo
   extra com segurança, em vez de interpretar bytes errados.
2. **Ordem de flash coordenada só quando o *tamanho* do struct muda** — quem *emite* o
   pacote precisa ser atualizado primeiro. Um bit novo dentro de um campo bitmask que já
   existia não exige essa coordenação.

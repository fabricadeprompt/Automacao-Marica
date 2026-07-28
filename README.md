# Automação Maricá

Sistema distribuído de monitoramento e controle de um reservatório residencial (caixa d'água + bomba de recirculação), construído com 3 ESP32 se comunicando via ESP-NOW.

**Dashboard ao vivo:** https://fabricadeprompt.github.io/Marica-Dashboard/

---

## Arquitetura

```
Caixa Água  ──ESP-NOW──▶  Caixa Bomba  ──ESP-NOW──▶  Caixa Controle  ──Wi-Fi──▶  Supabase
(nível)                   (relé + PZEM)               (LED, dashboard,           (telemetria)
                                                        servidor web)
```

Três placas fisicamente separadas, cada uma com sua própria lógica de segurança local — se uma cair, as outras continuam operando com o último dado válido conhecido (nunca "travado" silenciosamente: cada caixa detecta o silêncio da outra e sinaliza isso explicitamente, em vez de assumir que o último dado ainda é atual).

| Caixa | Função | ESP32 |
|---|---|---|
| Água | Mede o nível do reservatório (ultrassônico) | DevKit V1 |
| Bomba | Liga/desliga a bomba, mede corrente/tensão (PZEM-004T), roda auto-teste de segurança | DevKit V1 |
| Controle | Concentra telemetria, sinaliza erro (LED), serve dashboard local e envia dados ao Supabase | DevKit V1 |

Um M5Stack Cardputer atua como console remoto — hoje seu papel é específico: colocar qualquer uma das três caixas em modo OTA ou abrir o servidor web da Controle, via ESP-NOW.

## Hardware por caixa

**Caixa Água:** sensor ultrassônico AJ-SR04M-2, sensor capacitivo XKC-Y26S-V (transbordo/"ladrão").

**Caixa Bomba:** 2× relé SRD-05VDC-SL-C em série na bobina do contator (redundância — os dois precisam fechar pra energizar, qualquer um abrindo desenergiza), medidor de energia PZEM-004T v4.0 (Modbus RTU sobre UART).

**Caixa Controle:** sinaleira (semáforo R/Y/G), LED de erro dedicado, buzzer, botões físicos, servidor web local (fallback sem depender do dashboard remoto).

## Destaques de engenharia

- **Auto-teste de relé colado:** após todo desligamento, a Bomba isola cada relé individualmente e confere via PZEM se a potência cai a ~0 — se não cair, sinaliza falha sem bloquear a operação normal (o defeito físico continua existindo até troca de placa; o firmware só garante que ele não fica invisível).
- **Redundância de sensores:** além da medição contínua por ultrassônico, sensores de ponto único confirmam os extremos (nível máximo e mínimo do reservatório, e o cano de transbordo) — cobrindo o cenário em que o sensor principal trava numa leitura falsa.
- **Revisão externa antes de campo:** as mudanças de segurança passaram por revisão crítica antes do primeiro flash — incluindo um risco físico real (colagem simultânea dos dois relés durante o próprio teste) que só apareceu nessa revisão, documentado abaixo em vez de escondido.

## Estado atual / limitações conhecidas

Este projeto está em desenvolvimento ativo — nem todas as caixas estão no mesmo nível de maturidade:

- **Caixa Bomba e Caixa Controle:** redundância de relé, auto-teste de segurança e sinalização de erro unificada implementados e revisados. Aguardando flash em campo.
- **Caixa Água:** ainda **sem** a mesma redundância de hardware que a Bomba recebeu, e sem mitigação de condensação instalada no sensor ultrassônico (hipótese de causa raiz documentada internamente; capacitores recomendados, ainda não confirmados fisicamente instalados). Um incidente de sensor travado por condensação já ocorreu em campo — a mitigação de hardware é o próximo passo real para essa caixa, não uma formalidade.
- **Sensores redundantes de nível:** decisão de arquitetura fechada, aquisição/instalação física ainda pendente.

Se algo aqui parece incompleto, é porque está — a intenção é documentar o processo real de engenharia, não um produto acabado.

## Configuração

Este repositório **não contém nenhuma credencial**. Antes de compilar:

```
cp secrets.h.example secrets.h
```

Preencha `secrets.h` com sua rede Wi-Fi e (opcionalmente) seu projeto Supabase — o firmware detecta automaticamente se o Supabase não foi configurado e desativa o envio de telemetria sem precisar comentar código. `secrets.h` já está no `.gitignore`.

Ajuste também os MACs das placas (`marica_protocol.h`) e os IPs da sua rede local — os valores no repositório são placeholders.

## Estrutura

```
main_agua.cpp          — firmware da Caixa Água
main_bomba.cpp          — firmware da Caixa Bomba
main_controle.cpp       — firmware da Caixa Controle
marica_protocol.h       — protocolo compartilhado (structs, enums, MACs — compartilhado pelas 3 caixas)
secrets.h.example       — template de credenciais (copie para secrets.h)
.gitignore
```

## Licença

MIT — ver `LICENSE`.

# Mapa de Pinos (GPIO)

> Extraído diretamente do firmware publicado neste repositório — não é um documento
> mantido à parte manualmente. Em caso de divergência, o código-fonte é a fonte de verdade.

## Caixa Água (`main_agua.cpp`)

| GPIO | Função | Detalhe elétrico |
|---|---|---|
| 17 | `PIN_TRIG` — trigger do sensor ultrassônico AJ-SR04M-2 | Ligação direta (fio amarelo) |
| 16 | `PIN_ECHO` — echo do sensor ultrassônico AJ-SR04M-2 | Divisor de tensão R1=1kΩ + R2=2kΩ (5V→3,33V), fio verde |
| 18 | `PIN_LADRAO` — saída do sensor capacitivo XKC-Y26S-V (transbordo) | Divisor de tensão para 3,3V; saída NPN, `LOW` = ladrão ativo |

## Caixa Bomba (`main_bomba.cpp`)

> A Caixa Bomba contém só o ESP32 -- nenhum componente de força. Os dispositivos que
> estes pinos controlam (placa de relé, PZEM-004T) ficam fisicamente na **Caixa WEG**,
> conectados por dois cabos manga (ver `docs/HARDWARE.md`). Os pinos abaixo são do
> ESP32 da Bomba; a outra ponta de cada um está do outro lado do cabo.

| GPIO | Função | Detalhe elétrico |
|---|---|---|
| 18 | `GPIO_K1` — controla o relé K1 (placa de relé duplo, na Caixa WEG) | Dual-relay em série com K2 na bobina do contator — os dois precisam fechar para energizar a bomba |
| 19 | `GPIO_K2` — controla o relé K2 (existente, mesma placa) | Em série com K1 — qualquer um dos dois abrindo desenergiza a bomba |
| 16 | `GPIO_PZEM_RX` — RX do ESP32, recebe TX do PZEM-004T (na Caixa WEG) | Divisor de tensão R1=10kΩ + R2=20kΩ (5V→3,33V) |
| 17 | `GPIO_PZEM_TX` — TX do ESP32, envia para RX do PZEM-004T (Caixa WEG) | Ligação direta, sem divisor (RX do PZEM já tolera 3,3V) |

## Caixa Controle (`main_controle.cpp`)

| GPIO | Função |
|---|---|
| 27 | LED verde (sinaleira) |
| 26 | LED amarelo (sinaleira) |
| 25 | LED vermelho (sinaleira) |
| 33 | LED azul (indicador auxiliar — atualmente sem função ativa) |
| 13 | LED de erro dedicado (vermelho, unificado — ver `docs/PROTOCOL.md`) |
| 32 | LED branco (indicador de modo automático / ciclo Wi-Fi) |
| 15 | Buzzer |
| 14 | Botão 1 — clique curto liga a bomba, hold 3s abre o servidor web local |
| 12 | Botão 2 — parada de emergência |

## Pendente (Caixa Água — arquitetura decidida, ainda não implementada no firmware)

- 2ª unidade **XKC-Y25-V** (nível mínimo), somada à 1ª já em posse (nível máximo) —
  redundância de nível por sensor de ponto único. Sem GPIO definido no código ainda:
  instalação física depende de visita de campo.
- 2ª unidade **XKC-Y26S-V** no cano do ladrão — validação cruzada contra falso positivo
  por umidade residual. Mesma situação: aquisição feita, instalação/GPIO pendente.

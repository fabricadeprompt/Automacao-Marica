# Hardware (Lista de Componentes)

> Cobre apenas o que está fisicamente instalado e em operação. Componentes já
> adquiridos mas ainda não instalados aparecem em [Planejado](#planejado-não-instalado),
> separados do restante -- mesma política de honestidade do README.

## Caixa Elétrica (painel de força)

Único gabinete do sistema que toca tensão de rede. Nenhuma alta tensão entra em
nenhuma caixa com microcontrolador.

| Componente | Modelo | Especificação |
|---|---|---|
| Disjuntor | WEG MDWP | Bipolar, Curva C, 25A |
| Interruptor DR | WEG RDWS | Bipolar, 25A, 30mA (dispositivo separado do disjuntor) |
| Minicontator | WEG CWC07-10-30V26 (cód. 12459272) | Tripolar, 7A (AC-3), bobina 190V 50Hz/220V 60Hz, 1 contato auxiliar NA |
| Seletor rotativo | Lukma Electric ZB2-BE101 | 3 posições (Manual/Desligado/Automático), 2 pares de contato NO independentes |
| Placa de relé duplo | 2x relé Songle SRD-05VDC-SL-C | 1 placa física, 2 circuitos integrados (K1+K2 em série, ver `docs/PROTOCOL.md`) |
| Fonte de alimentação | Trilho DIN, 5V/3A | Alimenta a lógica da placa de relé e, via cabo, o ESP32 da Caixa Bomba |
| Medidor de energia | PZEM-004T (Peacefair) | Fica aqui, não na Caixa Bomba -- o lado de medição toca tensão de rede diretamente |

## Caixa Bomba

Só eletrônica de baixo nível -- nenhum componente de força.

| Componente | Modelo |
|---|---|
| ESP32 | DevKit V1302, 38 pinos, antena externa |
| Adaptador | Kit de expansão com terminais parafusados (substitui conectores Dupont diretos) |

### Interligação Caixa Elétrica ↔ Caixa Bomba

Dois cabos manga de 6 vias interligam as duas caixas:

| Cabo | Vias usadas | Sinais |
|---|---|---|
| Cabo 1 | 6 de 6 | 5V, GND, PZEM TX, PZEM RX, K1 (GPIO18), K2 (GPIO19) |
| Cabo 2 | 1 de 6 | 3,3V lógico (ESP32 → VCC da placa de relé) -- usa o GND do Cabo 1 como referência, sem retorno dedicado próprio |

GND é uma única referência comum: termina no terra da fonte (Caixa Elétrica), distribuído ali
localmente para o GND do PZEM e da placa de relé, e estende pelo cabo até o pino GND
do ESP32 (Bomba).

## Caixa Água

| Componente | Modelo | Especificação |
|---|---|---|
| ESP32 | DevKit V1302, 38 pinos, antena externa | |
| Adaptador | Kit de expansão com terminais parafusados | |
| Fonte de alimentação | Hi-Link HLK-PM01 | 5V/600mA |
| Sensor ultrassônico | AJ-SR04M-2 | À prova d'água -- vem de fábrica com a placa divisora do Echo (1kΩ+2kΩ), conjunto único de compra |
| Sensor capacitivo (ladrão) | XKC-Y26S-V | NPN, 6-36V, ligado direto ao ESP32 sem placa intermediária |
| Resistores (divisor XKC-Y26S-V) | 10kΩ + 20kΩ | Discretos, soltos -- não vêm em nenhum kit |

## Caixa Controle

| Componente | Modelo | Especificação |
|---|---|---|
| ESP32 | 30 pinos, genérica | Antena interna -- diferente do ESP32 de Água/Bomba |
| Adaptador | Kit de expansão com terminais parafusados | |
| Fonte de alimentação | Hi-Link HLK-PM01 | 5V/600mA -- mesmo modelo da Caixa Água |
| Sinaleira (semáforo) | — | Componente único: 3 LEDs (verde/amarelo/vermelho) + resistores já integrados |
| LEDs individuais | — | 3x: azul, vermelho de erro dedicado, branco -- cada um discreto, resistor próprio |
| Resistores (LEDs individuais) | 220Ω | Faixa de 220Ω a 330Ω também aceitável |
| Buzzer | genérico | |
| Botões físicos | genérico | 2x |

## Gabinetes

As três caixas eletrônicas (Água, Bomba, Controle) usam o mesmo padrão: caixa de
sobrepor 4x4, isolamento elétrico IP66, 110x110x60mm.

## Console remoto

Fora do escopo do firmware deste repositório (firmware próprio, não incluído aqui).

| Componente | Modelo |
|---|---|
| Console remoto | M5Stack Cardputer (M5Stack StampS3) |

## Planejado (não instalado)

Componentes já adquiridos para mitigação de condensação no sensor ultrassônico da
Caixa Água, mas ainda sem implementação física:

- 2x sensor capacitivo XKC-Y25-V (redundância de nível máximo/mínimo)
- Sensor de temperatura/umidade DHT22/AM2302 + sonda SHT31 (IP67)
- Aquecimento resistivo do transdutor: MOSFET IRLZ44N + resistores + caixa 4x4 adicional
  dedicada, interligada por cabo manga próprio

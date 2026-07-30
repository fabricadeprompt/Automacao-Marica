# Automação Maricá

Sistema automatizado e distribuído de monitoramento e controle de um reservatório residencial (caixa d'água + bomba de recirculação), construído com 3 microcontroladores ESP32 se comunicando via ESP-NOW.

**Dashboard ao vivo:** https://fabricadeprompt.github.io/Marica-Dashboard/

---

## Arquitetura

```
Caixa Água  ──ESP-NOW──▶  Caixa Bomba  ──ESP-NOW──▶  Caixa Controle  ──Wi-Fi──▶  Supabase
(nível)                        │                      (LED, dashboard,           (telemetria)
                                │ cabo (força)              servidor web)
                                ▼
                         Caixa Elétrica
                    (disjuntor, DR, contator,
                      relés, medidor de energia)
```

Quatro gabinetes fisicamente separados: caixa água, caixa bomba, caixa controle e
caixa elétrica. Três destas caixas possuem microcontroladores ESP32 e trocam dados por
rádio (ESP-Now); a caixa elétrica é um painel de força, ligada à caixa bomba por dois
cabos de 6 vias cada — nenhuma tensão de rede entra em nenhuma caixa eletrônica. Cada
caixa com ESP32 tem sua própria lógica de segurança local — se uma cair, as outras
continuam operando com dados válidos (nunca "travado" silenciosamente: cada caixa
detecta o silêncio da outra e sinaliza isso explicitamente, em vez de assumir que o
último dado ainda é atual).

| Caixa | Função | Hardware principal |
|---|---|---|
| Água | Mede o nível do reservatório (ultrassônico) | ESP32 DevKit V1302 |
| Bomba | Liga/desliga a bomba, lê o medidor de energia, roda auto-teste de segurança | ESP32 DevKit V1302 — só eletrônica, nenhum componente de força |
| Elétrica | Painel de força: disjuntor, DR, contator, relés, medidor de energia | Sem microcontrolador — ligada à Bomba por 2 cabos de 6 vias |
| Controle | Concentra telemetria, sinaliza erro (LED), serve dashboard local, envia dados ao Supabase | ESP32 30 pinos |

Um M5Stack Cardputer atua como console remoto: colocar
qualquer uma das três caixas com ESP32 em modo OTA ou abrir o servidor web da
Controle, via ESP-NOW. Além disso, ele também funciona
como monitor gráfico do nível da Caixa Água, podendo ser fixado, por exemplo, na porta da
geladeira através do imã do próprio M5Stack Cardputer.

## Destaques de engenharia

- **Auto-teste de relé colado:** após cada desligamento da bomba hidráulica, o microcontrolador da Caixa Bomba isola cada relé individualmente e confere, via medidor de energia, se a potência cai a ~0 — se não cair, sinaliza falha na placa de relés, porém, sem bloquear a operação normal (o defeito físico continua existindo até troca de placa; o firmware garante que o defeito não fique invisível).
- **Revisão externa antes de campo:** para cada mudança no firmware, duas IAs foram utilizadas para validar as mudanças.

## Estado atual / limitações conhecidas

Este projeto está em desenvolvimento ativo — nem todas as caixas estão no mesmo nível de maturidade:

- **Caixa Bomba e Caixa Controle:** redundância de relé, auto-teste de segurança e sinalização de erro unificada implementados e revisados. Aguardando flash em campo.
- **Caixa Água:** ainda **sem** a mesma redundância de hardware que a Bomba recebeu, e sem mitigação de condensação instalada no sensor ultrassônico (hipótese de causa raiz documentada internamente; componentes para o aquecimento resistivo já adquiridos, ainda não instalados fisicamente). Um incidente de sensor travado por condensação já ocorreu em campo — a mitigação de hardware é o próximo passo real para essa caixa, não uma formalidade.
- **Sensores redundantes de nível:** decisão de arquitetura fechada, aquisição/instalação física ainda pendente.

Se algo aqui parece incompleto, é porque está — a intenção é documentar o processo real de engenharia, não um produto acabado.

## Documentação técnica

Detalhes que não cabem num README enxuto vivem em `docs/`:

- [`docs/HARDWARE.md`](docs/HARDWARE.md) — lista completa de componentes, por caixa, com modelo exato
- [`docs/PINOUT.md`](docs/PINOUT.md) — mapa de pinos GPIO
- [`docs/PROTOCOL.md`](docs/PROTOCOL.md) — protocolo ESP-NOW compartilhado entre as caixas
- [`docs/BUILD.md`](docs/BUILD.md) — toolchain, bibliotecas e passo a passo de flash

## Configuração rápida

Este repositório **não contém nenhuma credencial**. Antes de compilar:

```
cp secrets.h.example secrets.h
```

Preencha `secrets.h` com sua rede Wi-Fi e (opcionalmente) seu projeto Supabase — o firmware detecta automaticamente se o Supabase não foi configurado e desativa o envio de telemetria sem precisar comentar código. `secrets.h` já está no `.gitignore`.

Passo a passo completo (MACs, IPs, ordem de flash) em [`docs/BUILD.md`](docs/BUILD.md).

## Estrutura

```
main_agua.cpp          — firmware da Caixa Água
main_bomba.cpp          — firmware da Caixa Bomba
main_controle.cpp       — firmware da Caixa Controle
marica_protocol.h       — protocolo compartilhado (structs, enums, MACs — compartilhado pelas 3 caixas)
secrets.h.example       — template de credenciais (copie para secrets.h)
.gitignore
docs/
  HARDWARE.md           — componentes por caixa
  PINOUT.md             — mapa de pinos GPIO
  PROTOCOL.md           — protocolo compartilhado
  BUILD.md              — compilação e flash
```

## Licença

MIT — ver `LICENSE`.

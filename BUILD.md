# Compilando e Flashando

## Toolchain

- **Arduino IDE** com o pacote de placas **ESP32 (Espressif)** instalado via Boards
  Manager.
- **Placa:** ESP32 Dev Module (DevKit V1) para as três caixas com sensores/relé (Água,
  Bomba, Controle).
- **Bibliotecas externas para essas três: nenhuma.** Todas as dependências (`WiFi`,
  `Preferences`, `ArduinoOTA`, `esp_now`, `esp_wifi`, `esp_task_wdt`, `HardwareSerial`,
  `WebServer`, `HTTPClient`) vêm no core ESP32 do Arduino — não precisa instalar nada
  pelo Library Manager. O protocolo Modbus RTU do PZEM-004T é implementado manualmente
  no firmware, sem biblioteca de terceiros.
- **O Cardputer (`main_cardputer.cpp`) é diferente.** Não é um DevKit avulso — precisa
  do pacote de placas **M5Stack** (Boards Manager) e da biblioteca **M5Cardputer**
  (Library Manager, que já traz M5Unified/M5GFX como dependência). Ao compilar esse
  arquivo, selecione a placa "M5Cardputer", não "ESP32 Dev Module".

## Passo a passo

1. Clone o repositório.
2. `cp secrets.h.example secrets.h` e preencha sua rede Wi-Fi (usada pela Caixa
   Controle, para a janela OTA e envio ao Supabase) e, opcionalmente, seu projeto
   Supabase — se deixar em branco, o firmware detecta e desativa o envio de telemetria
   automaticamente.
3. Em `marica_protocol.h`, substitua os MACs placeholder (`MAC_AGUA`, `MAC_BOMBA`,
   `MAC_CONTROLE`, `MAC_CARDPUTER`) pelos MACs reais das suas placas — descubra rodando
   `Serial.println(WiFi.macAddress())` no `setup()` antes de iniciar o ESP-NOW.
4. Ajuste os IPs fixos (`IP_CONTROLE`, `IP_BOMBA`, `IP_AGUA`, `IP_GATEWAY`,
   `IP_MASCARA`, `IP_DNS`) para a faixa da sua rede local.
5. Compile e grave os quatro `.cpp` — `main_agua.cpp`, `main_bomba.cpp`,
   `main_controle.cpp` na placa "ESP32 Dev Module", e `main_cardputer.cpp` na placa
   "M5Cardputer".

## Ordem de flash

Numa instalação nova, a ordem entre as três caixas não importa — todas partem do mesmo
`marica_protocol.h`. A regra de ordem coordenada (emissor antes do receptor) só se
aplica **depois**, ao atualizar um struct compartilhado que já está rodando em campo —
ver `docs/PROTOCOL.md`.

## Canal ESP-NOW

`CANAL_SEGURANCA_PADRAO` (canal 2) é fixado explicitamente antes de `esp_now_init()`,
numa sequência específica (promiscuous → set_channel → promiscuous off) validada em
revisão externa. Não é recomendado alterar essa sequência sem repetir essa revisão.

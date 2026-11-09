# Sony ZV-E10 Timelapse + Bambu Lab A1

Disparador remoto de código aberto para a **Sony ZV-E10** (Bluetooth LE), feito com ESP32,
integrado à impressora 3D **Bambu Lab A1**: a câmera tira uma foto a cada troca de camada.

## Como funciona

- **Câmera:** o ESP32 se conecta à ZV-E10 por BLE e envia os comandos de obturador
  (half down → fully down → fully up → half up).
- **Impressora:** o ESP32 entra na rede Wi-Fi local e assina o MQTT da A1
  (TLS, porta 8883, usuário `bblp`, senha = Access Code). Quando `print.layer_num`
  aumenta com a impressão em `RUNNING`, a câmera dispara.
- **Etapas:** primeiro conecta a câmera (etapa 1) e depois a impressora (etapa 2).
  Cada etapa faz no máximo 2 tentativas automáticas; depois disso use os botões do portal.
- **Botão físico:** GPIO 27 (para GND) dispara uma foto por pressionamento.

## Hardware

- ESP32 DevKit (`esp32dev`)
- Botão opcional entre GPIO 27 e GND

## Configuração

1. Grave o firmware com o PlatformIO (`pio run -t upload`).
2. Conecte no Wi-Fi **`Sony-ESP32`** (senha `12345678`) e abra `http://192.168.4.1`.
3. Em **Impressora (MQTT)** preencha: rede Wi-Fi, senha, IP da impressora, número de série
   e Access Code (tela da impressora). Marque **Disparar a cada troca de camada** e salve.
4. Na câmera, ative o pareamento do controle remoto Bluetooth e reinicie o ESP32.

Na impressora, deixe o **LAN Only Mode** (e, se existir no firmware, o **Developer Mode**) ativado.

### Dica para fotos limpas

No Bambu Studio, ative o timelapse no modo **Smooth** (a cabeça estaciona a cada camada) e,
em *Printer settings → Machine G-code → Time lapse G-code*, adicione uma pausa (ex.: `G4 P2000`).
Ajuste o "Atraso após troca de camada" no portal para a foto cair dentro dessa pausa.

## Comandos seriais (115200)

| Tecla | Ação |
|---|---|
| `F` | Disparar foto |
| `C` | Conectar à câmera (manual) |
| `D` | Desconectar da câmera |
| `I` | Conectar à impressora (manual) |
| `P` | Status da impressora |

## Notas técnicas

- **Memória:** Bluetooth (Bluedroid) + Wi-Fi + TLS mal cabem no ESP32. O firmware reserva no boot
  dois blocos do tamanho dos buffers do TLS e os libera só na hora de conectar à impressora,
  evitando `SSL - Memory allocation failed` por fragmentação. A página do portal é enviada em pedaços.
- **`lib/PubSubClient`:** cópia da PubSubClient 2.8 com correções (validação do tamanho do tópico
  e tratamento de pacote interrompido), que evitam corrupção de memória com mensagens truncadas.
- **Endereço Bluetooth:** o firmware usa um endereço "administrado localmente" para a câmera
  parear do zero caso guarde uma chave antiga.
- `backup/` guarda versões anteriores do firmware para referência.

## Licença

Distribuído sob a licença **MIT**: veja [LICENSE](LICENSE).
A biblioteca em `lib/PubSubClient` (de Nick O'Leary) também é MIT e mantém a própria licença
em `lib/PubSubClient/LICENSE.txt`.

Projeto independente, sem vínculo com a Sony ou a Bambu Lab.

# ESP-NOW AMI Waveform

Projeto acadêmico desenvolvido na UTFPR.

Três ESP32-S3 trocam mensagens pelo protocolo ESP-NOW: dois slaves codificam o texto em AMI pseudo-ternário, depois de cifrá-lo com XOR, e enviam ao master. O master repassa os quadros pela serial a um PC, onde um servidor Node.js mostra a forma de onda, os bits, os bytes e a mensagem decifrada em tempo real no navegador.

## Arquitetura

```
[Slave 1] --ESP-NOW--+
                     +--> [Master] --Serial USB--> [PC master] website/ (porta 3000)
[Slave 2] --ESP-NOW--+

[PC slave] --Serial USB--> [Slave N]   website-slave/ (porta 3001 ou 3002)
```

| Dispositivo | Placa | Papel |
|---|---|---|
| Master | ESP32-S3 SuperMini | ponte Serial e ESP-NOW |
| Slave 1 e 2 | ESP32-S3 DevKit | transmissores |

## Protocolo

- Codificação AMI pseudo-ternária: bit 0 alterna entre +V e -V, bit 1 fica em 0 V
- Cifra XOR simétrica aplicada antes da codificação (a chave é fixa no código, apenas para fins didáticos)
- Transporte por ESP-NOW no canal 1

## Estrutura

```
platformio.ini        ambientes master, slave1, slave2 e supermini
src/master/main.cpp   firmware do master
src/slave/main.cpp    firmware dos slaves (SLAVE_ID definido por build flag)
website/              servidor e página de exibição (PC master)
website-slave/        servidor e página de controle (PC slave)
```

## Tecnologias

C++ (Arduino framework) no PlatformIO, ESP-NOW, Node.js com serialport, HTML, CSS e JavaScript.

## Como executar

1. Grave o master e leia o MAC no monitor serial:
   ```bash
   pio run -e supermini -t upload
   pio device monitor
   ```
2. Coloque esse MAC nos flags `MASTER_MAC_B0` a `MASTER_MAC_B5` dos ambientes `slave1` e `slave2` em `platformio.ini`.
3. Grave os slaves:
   ```bash
   pio run -e slave1 -t upload
   pio run -e slave2 -t upload
   ```
4. Suba as páginas:
   ```bash
   cd website && npm install && node server.js                   # http://localhost:3000
   cd website-slave && npm install && SLAVE_ID=1 node server.js  # http://localhost:3001
   ```

Variáveis do `website-slave`: `SLAVE_ID` (1 ou 2), `SLAVE_PORT` (porta serial, detectada automaticamente se omitida), `PORT` (HTTP, padrão 3001) e `BAUD_RATE` (padrão 115200).

Os slaves também aceitam comandos pelo monitor serial: qualquer texto de até 20 caracteres define a mensagem e `SEND` (ou o botão BOOT) dispara o envio.

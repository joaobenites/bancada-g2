# bancada-g2

App próprio para a patinete elétrica **KuKirin G2 (2026)** via Bluetooth Low Energy, feito por engenharia reversa do protocolo.

**Tudo aqui só lê.** As páginas assinam as notificações da patinete e não enviam nenhum comando a ela.

Site: https://joaobenites.github.io/bancada-g2/

## Páginas

| Página | Para que serve |
|---|---|
| [`app.html`](https://joaobenites.github.io/bancada-g2/app.html) | Painel do dia a dia: velocímetro, modo, bateria estimada, motor, volta atual e histórico de voltas. |
| [`index.html`](https://joaobenites.github.io/bancada-g2/) | Bancada: ferramenta de análise. Agrupa os pacotes por tipo, destaca os bytes que mudam, compara com uma referência fixada e mantém um registro com marcações que dá para copiar. |

As duas páginas têm um modo de demonstração que funciona sem a patinete.

### Como usar

- **iPhone:** abra o site no app **Bluefy**. O Safari não tem Web Bluetooth.
- **Android / computador:** use o Chrome ou o Edge.
- A patinete aceita **uma conexão por vez**. Feche o nRF Connect ou qualquer outro app antes de conectar.
- Na lista de Bluetooth, o aparelho aparece com nome começando por `G2`.

## Protocolo

### GATT

| Serviço / característica | UUID | Uso |
|---|---|---|
| Serviço principal | `0000FFF0-0000-1000-8000-00805F9B34FB` | |
| Escrita | `0000FFF1-…` | Comandos (ainda **desconhecidos**) |
| Notificação | `0000FFF2-…` | Telemetria |
| Bateria (padrão) | `0000180F-…` / `00002A19-…` | Nível em %, quando o módulo expõe |
| OAD | `F000FFC0-…` | Atualização de firmware: **não mexer** |

### Fluxo

1. Ao assinar o `FFF2`, chega primeiro um bloco cheio de `FF`, que deve ser ignorado.
2. Depois chegam, alternados, dois pacotes de **20 bytes** a cada ~0,3 s.
3. Os tipos são identificados pelos bytes 0–1 (`27 22` e `F2 B0`).
4. Os valores de 16 bits são **little-endian**, salvo indicação.

### Pacote `27 22`

| Byte(s) | Campo | Decodificação | Status |
|---|---|---|---|
| 0–1 | Tipo | `27 22` | confirmado |
| 4–5 | Tensão da bateria | uint16 LE ÷ 100 = V | confirmado |
| 6–7 | Corrente ou potência? | uint16 LE | unidade desconhecida |
| 10–11 | Rotação do motor | uint16 LE = rpm | confirmado |
| 12 | **Velocidade** | byte = km/h | confirmado com o painel |
| 16 | Temperatura? | byte = °C? | hipótese |
| 18 | Estado | bit 1 (`0x02`) = motor ativo | confirmado |

Exemplo (parado): `27 22 82 17 79 12 00 00 00 00 00 00 00 00 EC 1B 19 00 40 00`. Aqui `0x1279` = 4729, ou seja, 47,29 V.

### Pacote `F2 B0`

| Byte(s) | Campo | Decodificação | Status |
|---|---|---|---|
| 0–1 | Tipo | `F2 B0` | confirmado |
| 3 | **Modo de condução** | `0F` = Eco, `14` = Sport, `19` = Race | confirmado (é código, não o limite real em km/h) |
| 8–9 | Tensão? | uint16 **big-endian** ÷ 100 = V? | hipótese |
| 12 | Temperatura? | byte = °C? | hipótese |
| 13–14 | Rotação do motor | uint16 LE = rpm | confirmado |
| 17 | Estado | bit 1 (`0x02`) = motor ativo | confirmado |

Exemplo (Eco, parado): `F2 B0 11 0F 50 00 00 E3 12 79 0C 00 19 00 00 00 00 40 00 00`.

### O que não aparece na telemetria

- **Farol:** ligar ou desligar o farol não muda nenhum byte.
- **Trava:** tem ícone no painel físico, mas nenhum comando é conhecido.

### Bateria

- 48 V nominal, 13 células em série (13S).
- O `app.html` estima a % pela tensão por célula numa curva típica de Li-ion (3,0 V = 0 %, 4,2 V = 100 %). Ele usa só leituras em repouso (motor parado e 0 km/h) para evitar a queda de tensão sob carga.
- A curva ainda não foi calibrada com as barras do painel.

## Segurança

1. Tudo é **somente leitura** até que um comando de escrita seja capturado de um app real.
2. **Nunca** escrever no serviço OAD (`F000FFC0-…`): é atualização de firmware e pode inutilizar o módulo.
3. Se um dia houver escrita: só com a roda parada e o motor desligado, com confirmação a cada envio, um pacote por vez, registrando antes e depois.
4. Nada de firmware customizado na controladora.

## Pendências

- Capturar os comandos de escrita (`FFF1`), por uma destas vias: sniffer nRF52840 + Wireshark, intermediário (MITM) no PC com `bleak` + `bless`, ou registro HCI de um Android.
- Descobrir a unidade dos bytes 6–7 do `27 22` (corrente ou potência).
- Confirmar as hipóteses de tensão e temperatura acima.
- Calibrar a curva da bateria com as barras do painel físico.
- Logger para PC (`g2_logger.py`, com `bleak`, salvando CSV), que ainda não está no repositório.
- Aguardando resposta da KuKirin sobre app oficial e trava.

## Aviso

Projeto pessoal, sem relação com a KuKirin. Use por sua conta e risco.

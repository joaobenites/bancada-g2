# CLAUDE.md — bancada-g2

Contexto para o Claude Code. Leia antes de qualquer tarefa neste repositório.

## O projeto

App próprio para a patinete **KuKirin G2 (2026)** do João, via Bluetooth Low Energy, feito por engenharia reversa do protocolo. O protocolo decifrado está documentado no `README.md` (tabelas byte a byte); trate o README como fonte da verdade e atualize-o sempre que algo novo for confirmado.

- Site publicado pelo GitHub Pages: https://joaobenites.github.io/bancada-g2/
- O João usa **iPhone**: as páginas rodam no app **Bluefy** (Web Bluetooth). Safari não serve.
- Comunicação com o João em **português do Brasil**.

## Arquivos

- `app.html` — painel de uso diário (velocímetro, modo, bateria, voltas). Arquivo único, sem build.
- `index.html` — Bancada: ferramenta de análise de pacotes (agrupa por tipo, destaca bytes, referência, registro com marcações).
- `g2_logger.py` — logger para PC com `bleak`, salva CSV. O PC do João roda **Windows** e tem Bluetooth.
- `README.md` — documentação pública e protocolo.

## Regras de segurança (obrigatórias)

1. **Tudo é somente leitura** até que um comando de escrita seja capturado de um app real. Não adicione envio para `FFF1` sem o João pedir explicitamente.
2. **Nunca escreva no serviço OAD** (`F000FFC0-…`): é atualização de firmware e pode inutilizar o módulo.
3. Se um dia houver escrita: só com roda parada e motor desligado, com confirmação a cada envio, um pacote por vez, registrando antes e depois.
4. Não sugira firmware customizado para a controladora (alto risco, sem comunidade para a KuKirin).

## Resumo do protocolo

- Serviço `FFF0`; notificações em `FFF2`; escrita em `FFF1` (comandos ainda desconhecidos).
- Ao assinar o `FFF2` chega um bloco de `FF` (ignorar). Depois alternam dois pacotes de 20 bytes a cada ~0,3 s, 16 bits little-endian.
- `27 22`: tensão = bytes 4–5 ÷ 100; rpm = bytes 10–11; **velocidade km/h = byte 12** (confirmado com o painel); motor ativo = bit 1 do byte 18.
- `F2 B0`: **modo = byte 3** (`0F` Eco, `14` Sport, `19` Race; é código, não limite real); rpm = bytes 13–14; motor ativo = bit 1 do byte 17.
- Farol **não** aparece na telemetria. Trava tem ícone no painel, mas sem comando conhecido.
- Bateria 48 V, 13S; % estimada por curva Li-ion com tensão em repouso.

## Pendências

- Capturar comandos de escrita: sniffer nRF52840 + Wireshark, intermediário (MITM) no PC com `bleak` + `bless`, ou registro HCI de um Android.
- Unidade dos bytes 6–7 do `27 22` (corrente/potência).
- Calibrar a curva da bateria com as barras do painel físico.
- Aguardando resposta da KuKirin (service@kukirin.net) sobre app oficial e trava.

## Convenções

- HTML em arquivo único, CSS e JS inline, sem frameworks nem build; precisa funcionar no Bluefy e no GitHub Pages.
- Toda leitura de `localStorage` em `try/catch`.
- Mensagens de commit curtas, em português.
- Antes de commitar HTML, verifique a sintaxe do JavaScript (ex.: `node --check` no script extraído).

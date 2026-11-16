# Lixeira Inteligente - IoT (Projeto Herdado — Etapa 2)

> Fork do projeto **Lixeira Inteligente IoT**, de **Deusiane Pimenta Rocha** (turma 2026.1), remontado e analisado por **Paulo Ribeiro** na disciplina de Internet das Coisas (2026.2) — IFMA, Campus São Luís Monte Castelo, Bacharelado em Sistemas de Informação. Professora: Salete Silva Farias.

## 📜 Créditos, licença e autorização

| Item | Informação |
| ---- | ---------- |
| Autora original | Deusiane Pimenta Rocha |
| Repositório original | <https://github.com/deusiii/deusiane-pimenta-lixeira-inteligente-iot> |
| Licença | Sem licença declarada (todos os direitos reservados à autora) |
| Responsável por este fork | Paulo Ribeiro |

Como o repositório original não declara licença, todos os direitos pertencem à autora, e por isso a autorização foi solicitada antes de usar e modificar o projeto.

### Registro da autorização

| Campo | Registro |
| ----- | -------- |
| Quem autorizou | Deusiane Pimenta Rocha (autora do projeto) |
| Data | 03/11/2026, às 08:18 |
| Canal | WhatsApp (mensagem direta) |
| Pedido | Autorização para usar, modificar e publicar um fork do projeto de IoT, mantendo o crédito à sua autoria |
| Resposta | "Autorizo !" |

![Registro da autorização da autora via WhatsApp](imagens/mensagem-de-autorização.png)

---

## 📌 Descrição do Projeto

O projeto **Lixeira Inteligente IoT** tem como objetivo automatizar o monitoramento do nível de preenchimento de lixeiras, permitindo identificar quando elas estão vazias, parcialmente cheias ou totalmente cheias.
Para isso, foi utilizado um **sensor ultrassônico HC-SR04**, responsável por medir a distância entre a tampa da lixeira e o lixo acumulado. Com base nessa medição, o sistema calcula o percentual de ocupação da lixeira.
Além disso, um **LED RGB** fornece uma indicação visual imediata do estado da lixeira, eliminando a necessidade de abrir a tampa ou acessar o dashboard para verificar seu nível.
O **ESP32** é utilizado como controlador principal, realizando a leitura dos sensores, acionamento dos LEDs e envio dos dados para a plataforma ThingSpeak através da Internet.

## 🎯 Objetivos

* Monitorar automaticamente o nível de preenchimento da lixeira.
* Informar visualmente o estado da lixeira através de um LED RGB.
* Enviar os dados para um dashboard IoT na plataforma ThingSpeak.
* Facilitar o gerenciamento da coleta de resíduos.

## 🧩 Componentes Utilizados

| Componente                  | Especificação                                               |
| --------------------------- | ----------------------------------------------------------- |
| ESP32                       | Microcontrolador com conectividade Wi-Fi integrada (lógica de 3,3 V) |
| Sensor Ultrassônico HC-SR04 | Sensor de distância com alcance aproximado de 2 cm a 400 cm, alimentado em 5 V |
| LED RGB (cátodo comum)      | Indicação visual do nível da lixeira                        |
| Resistor 1 kΩ               | Divisor de tensão do pino ECHO *(adicionado na remontagem)* |
| Resistor 2,2 kΩ             | Divisor de tensão do pino ECHO *(adicionado na remontagem)* |
| Jumpers                     | Conexões elétricas entre os componentes                     |
| Protoboard                  | Montagem do circuito sem solda                              |
| Balde (Lixeira)             | Estrutura física utilizada para armazenamento dos resíduos (altura útil configurada: 30 cm) |

---

## 🔧 O que precisei descobrir ou corrigir para remontar

### 1. Uso do HTTP com o ThingSpeak

O README original cita o ThingSpeak, mas não explica como a comunicação funciona nem como configurar a plataforma. Para remontar foi necessário entender que:

* A biblioteca `ThingSpeak.h` (da MathWorks, instalada pelo Gerenciador de Bibliotecas da Arduino IDE) faz, por baixo, uma **requisição HTTP** para `api.thingspeak.com` usando o `WiFiClient` do ESP32. A chamada `ThingSpeak.writeFields(channelID, writeAPIKey)` equivale a uma atualização do canal via API REST (`/update?api_key=...&field1=...`).
* É preciso criar **uma conta própria** no ThingSpeak, criar um **canal** com o **Field 1** (nível da lixeira em %), e copiar o **Channel ID** e a **Write API Key** (aba *API Keys*) para o código.
* O retorno `200` indica sucesso; outros códigos (por exemplo `-401`) indicam erro — geralmente intervalo de envio curto demais ou chave incorreta.
* A conta gratuita aceita **no máximo uma atualização a cada 15 segundos**; por isso o código usa `delay(20000)` entre os envios.
* O `SEU_CANAL` no código original precisa ser substituído por um número (o Channel ID), senão o código nem compila.

### 2. Correção da ligação do HC-SR04 (divisor de tensão no ECHO)

Na montagem original, o pino **ECHO** do HC-SR04 era ligado **direto** ao GPIO 18 do ESP32. O sensor é alimentado em 5 V e, por isso, o ECHO devolve um pulso de **5 V**, enquanto as GPIOs do ESP32 trabalham em **3,3 V** e não são tolerantes a 5 V. A consequência potencial era **queimar o pino (ou o próprio ESP32)**.

Na remontagem foi adicionado um **divisor de tensão** com resistores de **1 kΩ** e **2,2 kΩ**:

```
HC-SR04 ECHO ──[ 1 kΩ ]──┬── GPIO 18 (ESP32)
                         │
                      [ 2,2 kΩ ]
                         │
                        GND
```

Tensão no GPIO 18: `5 V × 2,2 / (1 + 2,2) ≈ 3,4 V`, dentro do limite seguro do ESP32 e ainda reconhecida como nível lógico alto. O pino **TRIG** não precisa de alteração, pois o ESP32 envia 3,3 V e o HC-SR04 reconhece esse valor como nível alto.

---

## 🔌 Funcionamento do Sistema

O sensor ultrassônico mede continuamente a distância entre a tampa da lixeira e o lixo.
Com base na distância medida, o sistema calcula o percentual de ocupação da lixeira e altera a cor do LED RGB:

| Nível da Lixeira | Cor do LED                     |
| ---------------- | ------------------------------ |
| Menor que 50%    | 🟢 Verde (Vazia)               |
| Entre 50% e 80%  | 🔵 Azul (Metade da Capacidade) |
| Acima de 80%     | 🔴 Vermelho (Cheia)            |

Os dados também são enviados para a plataforma ThingSpeak para monitoramento remoto.

## 📡 Protocolo de Comunicação Utilizado

### HTTP

O protocolo de comunicação utilizado foi o **HTTP (Hypertext Transfer Protocol)**, por meio da API REST do ThingSpeak.

### Justificativa da autora

Inicialmente foi testada a utilização do protocolo MQTT, bastante comum em aplicações IoT. Entretanto, durante o desenvolvimento foram encontrados diversos problemas de comunicação e estabilidade.
Por esse motivo, optou-se pelo uso da plataforma ThingSpeak, que utiliza requisições HTTP para o envio dos dados.
A escolha do HTTP foi motivada pelos seguintes fatores:

* Facilidade de implementação com o ESP32;
* Compatibilidade nativa com a plataforma ThingSpeak;
* Maior estabilidade durante os testes realizados;
* Facilidade de monitoramento dos dados enviados.

A avaliação crítica dessa escolha está na [análise do projeto](#5-o-projeto-usa-http-ou-mqtt-a-escolha-da-autora-se-sustenta-por-quê).

---

## 🔍 Análise do Projeto

### 1. Qual problema o projeto resolve?

Na coleta de resíduos, lixeiras são esvaziadas por rotina fixa: algumas transbordam antes da coleta e outras são visitadas ainda vazias. O projeto permite saber **quanto cada lixeira está cheia** sem abri-la — localmente (pelo LED) e remotamente (pelo ThingSpeak) —, possibilitando uma coleta sob demanda.

### 2. O que o sistema percebe, e com quais sensores?

Percebe a **distância entre a tampa e a superfície do lixo**, com o sensor ultrassônico **HC-SR04** instalado na tampa. A partir dessa distância e da altura da lixeira (`ALTURA_LIXEIRA = 30 cm`) calcula o **percentual de ocupação**.

### 3. O que o sistema faz, e com quais atuadores?

* **LED RGB**: indica localmente o estado (verde, azul ou vermelho).
* **Envio ao ThingSpeak**: publica o nível no Field 1 do canal, gerando um gráfico histórico.

Não há atuador que interfira fisicamente na lixeira (por exemplo, travar a tampa); o sistema apenas informa.

### 4. Como é o fluxo sensor → processamento → ação?

```
HC-SR04 (pulso TRIG 10 µs → tempo do ECHO)
        │
        ▼
ESP32: distância = duração × 0,034 / 2   (cm)
       nível = (ALTURA − distância) / ALTURA × 100   (limitado a 0–100 %)
        │
        ├──► LED RGB: < 50 % verde | 50–80 % azul | ≥ 80 % vermelho
        ├──► Monitor Serial: distância e nível
        └──► ThingSpeak (HTTP): field1 = nível
        │
        ▼
     aguarda 20 s e repete
```

### 5. O projeto usa HTTP ou MQTT? A escolha da autora se sustenta? Por quê?

Usa **HTTP**. A escolha **se sustenta**, mas por motivos diferentes dos apresentados:

* **Sustenta-se** porque o tráfego é **unidirecional e esporádico** (uma leitura a cada 20 s, do dispositivo para a nuvem) e não há comandos remotos. Nesse cenário, a principal vantagem do MQTT — conexão persistente, baixa latência e recebimento de comandos via *subscribe* — não é aproveitada. O HTTP com o ThingSpeak ainda entrega armazenamento e gráfico prontos, sem precisar de broker ou dashboard próprios.
* **A justificativa original é frágil**: os "problemas de estabilidade" do MQTT provavelmente vieram da implementação (broker público, falta de reconexão), e não do protocolo. O próprio código atual também não trata queda de Wi-Fi, então o HTTP não está, de fato, mais protegido contra instabilidade.
* **Pontos de atenção**: a biblioteca usa HTTP sem TLS, então a Write API Key trafega em texto claro; e cada envio abre uma nova conexão TCP, o que gasta mais energia que uma conexão MQTT persistente — relevante se a lixeira um dia funcionar a bateria.

### Avaliação crítica do código original

| Problema | Consequência |
| -------- | ------------ |
| ECHO ligado direto ao ESP32 (5 V em pino de 3,3 V) | Risco de queimar o GPIO — **corrigido na remontagem** |
| `pulseIn` sem *timeout* e sem tratar retorno `0` | Se o eco não volta, `distância = 0` → nível 100 % → **falso alarme de lixeira cheia** (LED vermelho e dado errado no ThingSpeak) |
| Leitura única por ciclo | Ruído do ultrassom (lixo irregular, reflexões) gera oscilações no gráfico e troca de cor indevida |
| Wi-Fi conectado apenas no `setup()`, em laço infinito | Se o roteador cair, não há reconexão; se não houver Wi-Fi ao ligar, **nem o LED funciona** |
| Credenciais (SSID, senha e API Key) escritas no código | Risco de vazamento ao publicar no GitHub |
| Apenas o nível é enviado | Sem a distância bruta, é difícil diagnosticar leituras erradas pelo dashboard |

---

## 💡 Proposta de Melhoria (Etapa 3)

### O que vai mudar

Uma melhoria combinada de **confiabilidade** e **segurança**:

1. **Leitura confiável do sensor**
   * `pulseIn(ECHO, HIGH, 30000)` com *timeout* (~5 m de alcance máximo) e descarte de leituras inválidas (`0` ou fora da faixa 2–400 cm).
   * **Mediana de 5 leituras** por ciclo, eliminando picos de ruído.
   * Se todas as leituras forem inválidas, o LED indica **erro de sensor** (ex.: piscar amarelo) e nada é enviado ao ThingSpeak, em vez de reportar 100 %.
2. **Conectividade resiliente**
   * Verificação de `WiFi.status()` a cada ciclo, com **reconexão automática** sem travar o programa.
   * O LED e a leitura **continuam funcionando offline**; apenas o envio é suspenso até o Wi-Fi voltar.
   * Substituir o `delay(20000)` por controle com `millis()`, para o LED reagir na hora mesmo entre os envios.
3. **Credenciais fora do código**
   * Criar `secrets.h` (SSID, senha, Channel ID e Write API Key), incluído no `.gitignore`.
   * Versionar apenas um `secrets.example.h` com valores de exemplo.
4. **Mais dados para diagnóstico**
   * Enviar também a **distância bruta** (Field 2) e um **código de status** (Field 3: OK / erro de sensor / reconectado), facilitando a análise pelo dashboard.

### Por quê

O sistema existe para orientar a coleta; um **falso "cheia"** gera viagem desnecessária, e um sistema que **para de funcionar quando o Wi-Fi cai** deixa de cumprir até a função local (LED). Já as credenciais no código contrariam as orientações da disciplina e são um risco real em repositório público.

### Componentes e serviços necessários

* Os mesmos do protótipo atual (ESP32, HC-SR04, LED RGB, resistores 1 kΩ e 2,2 kΩ).
* Conta gratuita no ThingSpeak com **3 fields** no canal.
* Nenhum componente ou serviço pago adicional.

### Como vou demonstrar que melhorou (antes × depois)

| Teste | Antes (original) | Depois (esperado) |
| ----- | ---------------- | ----------------- |
| Apontar o sensor para o vazio / tecido (sem eco) | Nível vai a 100 %, LED vermelho | Leitura descartada, LED de erro, nada enviado |
| Desligar o roteador por 1 min e religar | Envios falham para sempre (ou o ESP trava ao reiniciar sem Wi-Fi) | LED segue funcionando; envios voltam sozinhos |
| Ligar o ESP32 sem Wi-Fi disponível | Fica preso em "Conectando WiFi..." e o LED não acende | LED funciona normalmente desde o início |
| Lixo irregular / mão passando no sensor | Gráfico com picos e LED trocando de cor | Curva estável graças à mediana |
| `git grep` por senha/API Key no repositório | Credenciais no código-fonte | Nenhuma credencial versionada |

A evidência será o Monitor Serial e o gráfico do ThingSpeak lado a lado, com os dois firmwares.

---

## ⚙️ Instruções de Instalação

### 1. Materiais Necessários

* ESP32
* Sensor HC-SR04
* LED RGB (cátodo comum)
* 1 resistor de 1 kΩ e 1 resistor de 2,2 kΩ
* Jumpers
* Protoboard
* Cabo USB para programação

### 2. Conexões sensores e ESP32

| Sensor HC-SR04   | ESP32                                              |
| ---------------- | -------------------------------------------------- |
| VCC              | 5V (Vin)                                           |
| GND              | GND                                                |
| TRIG             | GPIO 5                                             |
| ECHO             | GPIO 18 **via divisor de tensão** (1 kΩ em série; 2,2 kΩ do GPIO 18 ao GND) |

| LED RGB (CÁTODO) | ESP32                          |
| ---------------- | ------------------------------ |
| R                | GPIO 25                        |
| G                | GPIO 26                        |
| B                | GPIO 27                        |
| Comum (-)        | GND                            |

**Esquema de ligação (simulação no Wokwi):**

![Esquema de ligação do projeto no Wokwi](imagens/esquema-de-ligacao.png)

O arquivo do circuito está em [`wokwi/diagram.json`](wokwi/diagram.json) e pode ser importado no Wokwi.

### 3. Instalar Dependências

Na Arduino IDE:

* Instalar o suporte à placa ESP32 (Gerenciador de Placas → *esp32 by Espressif Systems*).
* Instalar a biblioteca **ThingSpeak** (by MathWorks) pelo Gerenciador de Bibliotecas.
* A biblioteca `WiFi.h` já vem com o pacote da placa ESP32.

### 4. Configurar Wi-Fi

No código, alterar as seguintes variáveis:

```cpp
const char* ssid = "SEU_WIFI";
const char* password = "SUA_SENHA";
```

### 5. Configurar ThingSpeak

1. Crie uma conta em <https://thingspeak.mathworks.com>.
2. Crie um canal (*Channels → New Channel*) e habilite o **Field 1** (Nível da lixeira, %).
3. Copie o **Channel ID** e a **Write API Key** (aba *API Keys*) para o código:

```cpp
unsigned long channelID = 0000000;      // seu Channel ID (número)
const char* writeAPIKey = "SUA_API_KEY";
```

> ⚠️ Nunca faça commit das suas credenciais reais.

### 6. Enviar o Código

* Conectar o ESP32 ao computador.
* Selecionar a placa (*ESP32 Dev Module*) e a porta correta na Arduino IDE.
* Compilar e enviar o código para a placa.

## 🚀 Como Utilizar

1. Energize o ESP32.
2. Aguarde a conexão com a rede Wi-Fi (acompanhe pelo Monitor Serial a 115200 baud).
3. O sensor começará a monitorar o nível da lixeira.
4. Observe a cor do LED RGB para identificar o estado da lixeira.
5. Acompanhe os dados remotamente através do ThingSpeak (atualização a cada ~20 s).

## 🎥 Vídeo do Circuito (projeto original)

- [Link do vídeo](https://drive.google.com/file/d/1vz3ODsqVBXVFEM01nTiaUAWpsk-c5HjR/view?usp=sharing)

## 📝 O que mudou em relação ao original

| Etapa | Mudança |
| ----- | ------- |
| Etapa 2 | Adicionado divisor de tensão (1 kΩ + 2,2 kΩ) no pino ECHO do HC-SR04 |
| Etapa 2 | README ampliado: autorização, configuração do ThingSpeak, análise e proposta de melhoria |
| Etapa 3 | *A preencher após a implementação da melhoria* |

---

## 👩‍💻 Autoria

**Projeto original:** Deusiane Pimenta Rocha — estudante do 7º período de Sistemas de Informação, projeto desenvolvido para a disciplina de Internet das Coisas (IoT), 2026.1.

**Remontagem, análise e proposta de melhoria (2026.2):** Paulo Ribeiro.

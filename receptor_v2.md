
# Manual do Sistema Receptor_v2 — Versão Integrada

## 1. Introdução ao Sistema Receptor

O **Receptor_v2** é um sistema completo de controle de acesso projetado para funcionar com **catracas**, **cancelas**, **bilheterias** e **estacionamentos**. Ele combina:
- Uma aplicação desktop robusta, escrita em **C++ com Qt**.
- Controle físico de hardware via **GPIO** (indicada para sistemas Linux embarcados, como Raspberry Pi).
- Acesso a **MySQL/MariaDB** para registro de eventos.

O sistema é altamente flexível: pode operar com equipamentos físicos em modo real ou em modo simulado via **emuladores** gráficos, o que facilita demonstrações e testes.

### Principais Aplicações

- **Catracas** (academias, clubes, condomínios, eventos).
- **Cancelas** (gerenciamento de tráfego de veículos).
- **Bilheterias** (emissão/validação de tickets em entradas e saídas).
- **Estacionamentos** (controle de tempo, impressão de tickets, logs).
- Operação **offline** (com banco local) ou **online** (rede TCP/IP).

### Tecnologias Utilizadas

- **Qt (C++)**: Interface gráfica e motor principal da aplicação.
- **GPIO (Linux)**: Acesso a relés, LEDs, sensores e botões.
- **MySQL / MariaDB**: Banco de dados para armazenamento de logs e configurações.
- **ZPL / Shell**: Scripts de impressão e automação (permite personalizar layouts de ticket).

### Vantagens

- **Código Aberto e Personalizável**: Fácil de estender ou adaptar.
- **Funciona sem Internet**: Ideal para locais com conexão instável.
- **Integração Simples**: Adicionar sensores externos, impressoras, biometria etc.
- **Emuladores**: Simulam catraca, cancela e sensores sem hardware físico.

---

## 2. Funcionalidades Principais

O Receptor_v2 apresenta diversos módulos interligados, contemplando cenários de controle de acesso variados:

### Controle de Catraca
- **Validação** de permissões de entrada/saída.
- **Giro** configurável: horário, anti-horário ou livre.
- Identificação via **código**, cartão, sensor ou ticket.
- Emissão de **sinais GPIO** para liberar/bloquear.

### Controle de Cancela
- **Sensores** de presença (distância, botoeiras).
- **Abertura automática** ao reconhecer autorização.
- Lógica independente para fluxo de veículos.

### Bilheteria
- Geração de **tickets de entrada** (horário, identificador).
- Validação de **tickets de saída** (tempo, tipo, cortesia).
- **Impressão** personalizada (data, cabeçalho).

### Estacionamento
- **Cálculo** de tempo de permanência.
- **Registro** de entrada/saída de veículos.
- Base de dados local para histórico e backup.

### Impressão de Tickets
- Compatível com **impressoras térmicas** e Zebra.
- Layouts **.zpl** ou **.txt** customizáveis.
- Impressão **automática** atrelada às ações (bilheteria, estacionamento).

### Emuladores
- **ratch_emulator**: gira catraca virtualmente.
- **gate_emulator**: abre/fecha portão sem hardware real.
- **Interface gráfica** para demonstrações comerciais.

### Operação com Rede
- **TCP/IP** com dispositivos externos (porteiros, servidores).
- Envio de **status** e **eventos** em tempo real.
- Modo **offline** com sincronização posterior (fallback).

### Sensores
- Suporte a **sensores de distância**, botoeiras, infravermelho etc.
- GPIO com leitura em **tempo real**.
- Scripts em Python para facilitar integração (`dist_sensor.py`).

---

## 3. Componentes do Sistema

### 3.1. Interface Gráfica (Qt)

- `mainwindow.ui`: define a **janela principal** e seus elementos (botões, texto, logs).
- Demais `.ui` para **emuladores** e telas específicas (`commands.ui`, `ratch_emulator.ui`, `gate_emulator.ui`).

### 3.2. Núcleo Lógico

- **`ratch`**: gerencia estados (WAITING, PASSING etc.), inicialização de GPIOs e módulos.
- **`receiver`**: decide permissões de acesso, registra eventos, conversa com o banco.
- **`controller`**: ponto de entrada que instancia `ratch`, `mainwindow` e orquestra o ciclo de vida da aplicação.

### 3.3. Banco de Dados

- **`lbd_database.*`** e **`db_utils.*`**: classes para conectar ao MySQL/MariaDB, executar queries, armazenar logs e tickets.

### 3.4. Rede

- **`network_controller.*`**: gerencia conexões TCP/IP, status de rede e fallback offline.
- Possibilita operação distribuída ou sincronização com um servidor central.

### 3.5. Scripts e Auxiliares

- **`inicializa_gpios.sh`**: prepara GPIOs (export, direction).
- **`dist_sensor.py`**: exemplo de sensor de distância.
- **Scripts de impressão** `.sh`, `.zpl` e `.txt` para diferentes impressoras.

### 3.6. Emuladores

- **`ratch_emulator`**: simula catraca, botões e estado de giro.
- **`gate_emulator`**: simula portões e sensores de estacionamento.

---

## 4. Instalação do Sistema

### Requisitos

- **Linux** (ex: Ubuntu ou Raspberry Pi OS).
- **Qt 5.x ou 6.x** instalado.
- **Acesso root** para GPIO.
- **Impressora** compatível (se desejar impressão local).

### Passos

1. **Instale** o `.deb` (se disponível) ou compile via Qt Creator.
2. **Execute** `inicializa_gpios.sh` como root (ou configure pinos manualmente).
3. **Edite** `config.ini` com host do banco, GPIOs e parâmetros.
4. Conecte **sensores** e **impressora** (USB, Ethernet).
5. **Rode** o aplicativo (`./Receptor_v2` ou via Qt).

---

## 5. Arquivo de Configuração (`config.ini`)

Arquivo central para definir comportamento do sistema. Exemplo:
```ini
[geral]
rotation=1
operation=1
app_=2
enable_network=1

[banco]
host=localhost
user=root
pass=admin
database_name=lbd

[gpio]
botao_estacionamento=17
sensor_estacionamento=4

[impressao]
cabecalho1=Estacione Bem
cabecalho2=Seu controle inteligente
```

- **`rotation`**: sentido da catraca (1=horário, 2=anti-horário, 3=livre).
- **`operation`**: tipo de operação da catraca (simples/duplo).
- **`app_`**: modo de uso (1=catraca, 2=cancela, 3=bilheteria).
- **`enable_network`**: ativa rede TCP/IP.
- **`cabecalho1/2`**: textos impressos no ticket.

---

## 6. Operando com Catraca

1. Usuário apresenta credencial (código, cartão, ticket).
2. **`receiver`** valida permissão (offline ou via rede).
3. Se permitido:
   - Gira catraca (GPIO2 ou GPIO3).
   - Muda status para PASSING.
4. Se negado:
   - Emite alerta (GPIO10).
   - Registra tentativa negada.

### Sensores e Estados

- **`SYSTEM_GPIO4/5`**: sensores de passagem.
- **`SYSTEM_GPIO27`**: controle do giro.
- **`SYSTEM_GPIO6/11`**: bloqueios.

---

## 7. Operando com Cancela

Parecido com catraca, mas voltado a veículos.

1. Sensor de presença detecta veículo (`dist_sensor.py` ou outro sensor).
2. Usuário exibe ticket/código.
3. Se validado, libera relé da cancela.
4. Se negado, dispara erro visual/sonoro.

### `receptor_cancela_worker`

Gerencia a automação (tempo de abertura, travamento). Ideal para estacionamentos e portões.

---

## 8. Emuladores e Testes

- **`ratch_emulator`**: simula catraca virtualmente (botões de teste).
- **`gate_emulator`**: simula portão abrindo/fechando.
- Úteis para **demonstrações comerciais** ou para desenvolvimento sem hardware real.

---

## 9. Impressão de Tickets

- **Texto simples (`.txt`)** ou **Zebra (`.zpl`)**.
- Scripts `impressao_estacionamento.sh` e `impressao_estacionamento_zebra.sh`.
- Personalização de cabeçalhos no `config.ini`.

---

## 10. Banco de Dados e Logs

- **MySQL/MariaDB**: registra cada passagem, entrada ou evento.
- Arquivos `lbd_database.*` e `db_utils.*` controlam consultas.
- **GPIO17** pode indicar falha de banco (ou reconexão).

---

## 11. Modo de Rede

- **Conexões TCP/IP** entre receptores e servidores.
- **Offline automático** se rede falhar, reenvio posterior.
- **`network_controller`** gerencia status e reconexões.

---

## 12. Desenvolvendo e Personalizando

- **Modular**: fácil adicionar novas funcionalidades.
- Pode integrar **RFID, QR Code, biometria** etc.
- Use **Qt Creator** para compilar (arquivos `.pro`).
- Principais códigos: `main.cpp`, `ratch.cpp`, `receiver.cpp`, `controller.cpp`.

### Estrutura Interna e Melhores Práticas

- **Enums** em vez de valores numéricos fixos (melhor legibilidade).
- Comentários de alto nível em cada classe.
- Possíveis refatorações de classes grandes, p. ex. `receiver`.

---

## 13. Dicas de Venda e Implantação

1. **Demonstrações Comerciais**  
   - Use os emuladores para exibir fluxo de acesso (catraca ou portão) sem hardware.
2. **Estratégias de Venda**  
   - Destaque operação offline (independência de internet).
   - Personalização (impressão de logotipos, integrações).
   - Baixo custo de hardware (Raspberry Pi).
3. **Checklist de Instalação**  
   - [ ] Instalar `.deb` ou compilar via Qt  
   - [ ] Executar `inicializa_gpios.sh`  
   - [ ] Configurar `config.ini`  
   - [ ] Conectar sensores/impressoras  
   - [ ] Testar com hardware real ou emuladores  

---

## 14. Conclusão

O **Receptor_v2** apresenta uma arquitetura robusta e extensível. Ele **integra hardware** (GPIO, sensores, relés), **software** (Qt/C++), **banco de dados** (MySQL/MariaDB) e **rede** (TCP/IP). Ideal para soluções de catracas, cancelas, bilheteria e estacionamento, com forte ênfase em **flexibilidade** e **personalização**.


---

# Manual do Sistema Receptor_v2

## 1. Introdução ao Sistema Receptor

O **Receptor_v2** é um sistema completo de controle de acesso projetado para funcionar com **catracas**, **cancelas**, **bilheterias** e **estacionamentos**. Ele combina uma aplicação desktop robusta, escrita em **C++ com Qt**, com controle físico de hardware via **GPIO** (em sistemas Linux embarcados como Raspberry Pi).

O sistema é altamente flexível e pode operar em modo real com equipamentos físicos ou em modo simulado usando **emuladores** gráficos, facilitando demonstrações e testes.

### Principais Aplicações:
- Controle de acesso em clubes, academias, condomínios, eventos.
- Gerenciamento de entradas/saídas de veículos.
- Emissão e validação de tickets.
- Registros detalhados de acesso com horário e tipo.
- Operação offline com banco local ou online via rede.

### Tecnologias Utilizadas:
- **Qt (C++)**: Interface gráfica e controle da aplicação.
- **GPIO (Linux)**: Acesso e controle de sensores, relés e LEDs.
- **MySQL / MariaDB**: Banco de dados relacional.
- **ZPL / Shell**: Impressão e scripts de inicialização.

### Vantagens:
- Código aberto e personalizável.
- Funciona com ou sem rede.
- Integração fácil com sensores externos e impressoras.
- Possui módulo de emulação completo para testes e vendas.

---

## 2. Funcionalidades Principais

O sistema Receptor_v2 possui diversos módulos que permitem controle de acesso em ambientes variados. Abaixo, as funcionalidades mais importantes:

### Controle de Catraca
- Geração e validação de permissões para entrada/saída.
- Controle de giro (horário, anti-horário, livre).
- Identificação por código, cartão, sensor ou ticket.
- Emissão de sinais GPIO para liberar ou bloquear.

### Controle de Cancela
- Integração com sensores de presença (distância, botoeiras).
- Abertura automática ao identificar autorização.
- Módulo com lógica independente para passagem de veículos.

### Bilheteria
- Geração de tickets de entrada com horário e identificador.
- Validação de ticket na saída (tempo, tipo, cortesia).
- Impressão personalizada com data e cabeçalho.

### Estacionamento
- Cálculo de tempo de permanência.
- Controle de entrada/saída de veículos.
- Registros locais e backup para histórico de movimentação.

### Impressão de Tickets
- Suporte a impressoras térmicas e Zebra.
- Layouts personalizáveis via `.zpl` e `.txt`.
- Impressão automática integrada com ações da bilheteria.

### Emuladores
- `ratch_emulator`: simula o giro da catraca.
- `gate_emulator`: simula abertura/fechamento de portões.
- Interface gráfica para demonstrações comerciais.

### Operação com Rede
- Comunicação TCP/IP com outros dispositivos (porteiros, servidores).
- Envio de status e eventos em tempo real.
- Modo offline com sincronização posterior.

### Sensores
- Suporte a sensores de distância, botoeiras, barreiras infravermelho.
- Respostas em tempo real via GPIO.
- Scripts auxiliares em Python para sensores simples.

---

## 3. Componentes do Sistema

O projeto Receptor_v2 é dividido em camadas bem definidas:

### Interface Gráfica (Qt)
- Arquivo `mainwindow.ui` define a janela principal.
- Botões, campos de texto, logs e atalhos são tratados visualmente.

### Núcleo Lógico
- `ratch`: gerencia GPIO, status, inicialização e eventos principais.
- `receiver`: centraliza verificação de acesso, tickets e respostas.
- `controller`: ponto de entrada que instancia e conecta os módulos.

### Banco de Dados
- Usado para armazenar registros de acessos e eventos.
- Arquivo `lbd_database.*` e `db_utils.*` fazem a interface com o banco.

### Rede
- `network_controller`: conexão TCP com outros dispositivos ou servidores.
- Suporte a envio de mensagens, atualização de status e fallback em caso de falha.

### Scripts e Auxiliares
- `inicializa_gpios.sh`: prepara os GPIOs na inicialização.
- `dist_sensor.py`: exemplo de sensor em Python.
- Scripts de impressão `.sh`, `.txt` e `.zpl` para diferentes impressoras.

### Emuladores
- Permitem simulação de sensores, catraca e portão sem hardware físico.
- Ideal para testes e apresentações de venda.

---

## 4. Instalação do Sistema

### Requisitos:
- Linux (Ubuntu ou Raspberry Pi OS)
- Qt 5.x ou 6.x instalado
- Acesso root para controle de GPIO
- Impressora compatível

### Passos:
1. Instale o `.deb` do sistema (caso disponível)
2. Execute `inicializa_gpios.sh` com permissões root
3. Configure o `config.ini`
4. Conecte sensores e impressora
5. Execute o aplicativo

---

## 5. Arquivo de Configuração (`config.ini`)

O arquivo `config.ini` é essencial para definir o comportamento do sistema. Ele fica na raiz do projeto e é carregado automaticamente pelo `ratch` na inicialização.

### Exemplos de Parâmetros:
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

### Significado:
- `rotation`: sentido da catraca (1=horário, 2=anti-horário, 3=livre)
- `operation`: tipo de operação (0=catraca simples, 1=duplo sentido)
- `app_`: modo de uso (1=catraca, 2=cancela, 3=bilheteria)
- `enable_network`: ativa/desativa uso de rede
- `botao_estacionamento`: pino GPIO para botão
- `sensor_estacionamento`: pino para sensor de presença
- `cabecalho1/2`: textos do ticket impresso

---

## 6. Operando com Catraca

### Fluxo de Passagem:
1. Usuário apresenta código/cartão/ticket
2. `receiver` valida a permissão (local ou via rede)
3. Se permitido:
   - Gira sentido horário/anti-horário
   - Ativa GPIO de liberação (ex: GPIO2 ou GPIO3)
   - Atualiza status: `WAITING` → `PASSING` → `SPIN`
4. Se negado:
   - Emite sinal sonoro/luz (GPIO10)
   - Loga tentativa negada

### Sensores e Estados:
- `SYSTEM_GPIO4/5`: sensores de passagem
- `SYSTEM_GPIO27`: controle do giro
- `SYSTEM_GPIO6/11`: bloqueios em caso de tentativa incorreta

---

## 7. Operando com Cancela

### Funcionalidades:
- Abertura automática ao detectar presença + permissão
- Sensor de distância para ativação (HC-SR04 ou IR)
- Controle via relé (GPIOs)

### Fluxo:
1. Sensor detecta veículo
2. Usuário apresenta ticket ou identificação
3. Se validado:
   - Ativa GPIO que aciona o relé da cancela
   - Aguarda tempo ou confirmação de saída
4. Se não validado:
   - Emite erro visual ou sonoro

### Scripts:
- `dist_sensor.py`: detecta presença via GPIO
- `receptor_cancela_worker`: gerencia temporização

---

## 8. Emuladores e Testes

### Emulador de Catraca (`ratch_emulator`)
- Interface gráfica com botões de sensores virtuais
- Gira virtualmente (simula entrada/saída)
- Permite testes de software sem hardware físico

### Emulador de Portão (`gate_emulator`)
- Simula abertura/fechamento de portões
- Ideal para simulações de controle de estacionamento

### Como Usar:
- Basta executar o app em modo emulador (configuração `app_`)
- Ativar sensores e visualizar logs no terminal/UI
- Pode ser usado para demonstração comercial

---

## 9. Impressão de Tickets

### Formatos Suportados:
- **Texto simples (`.txt`)**
- **Zebra (`.zpl`)**

### Configuração:
- Layouts: `impressao_estacionamento.txt`, `impressao_estacionamento.zpl`
- Scripts: `impressao_estacionamento.sh`, `impressao_estacionamento_zebra.sh`

### Cabeçalhos Personalizados:
Definidos no `config.ini`:
```ini
[impressao]
cabecalho1=Estacione Bem
cabecalho2=Seu controle inteligente
```

---

## 10. Banco de Dados e Logs

### Banco de Dados
- MySQL/MariaDB
- `lbd_database.*` e `db_utils.*`

### Registro de Acessos
- Logs automáticos de entrada e saída
- Inclusão de horário, status e código

### Status de Banco
- GPIO17 indica status do banco
- Reconeção automática quando possível

---

## 11. Modo de Rede

### Conexões
- TCP/IP
- Comunicação entre receptores e servidores

### Tolerância a Falhas
- Modo offline automático
- Reenvio após reconexão

### Componentes:
- `network_controller`, `receiver`

---

## 12. Desenvolvendo e Personalizando

### Personalização
- Modularidade facilita extensão
- Suporte a QR Code, RFID, biometria

### Desenvolvimento
- Qt Creator recomendado
- Códigos: `main.cpp`, `receiver.cpp`, `controller.cpp`, `ratch.cpp`

---

## 13. Dicas de Venda e Implantação

### Demonstrações Comerciais
- Use emuladores para mostrar o funcionamento completo

### Estratégias de Venda
- Destaque a operação offline
- Enfatize a personalização
- Prove o baixo custo de implantação

### Checklist para Instalação
- [ ] Instalar `.deb`
- [ ] Executar `inicializa_gpios.sh`
- [ ] Configurar `config.ini`
- [ ] Conectar sensores e impressora
- [ ] Testar o sistema

---

Esse conteúdo pode ser transformado em um **curso técnico e comercial completo** para capacitação de integradores e vendedores.

Se quiser, posso gerar um PDF com formatação profissional ou converter este material em slides para apresentação. Deseja que eu prepare essa versão agora?


# Funcionalidades do Totem_Acesso_Web

## 1. Leitura de Bilhetes (HomePage)

### 1.1 Scanner de Códigos de Barras / QR
- **Plugin utilizado:** `@ionic-native/barcode-scanner`  
- **Formatos suportados:** EAN‑13, CODE‑128, QR Code, UPC, PDF417, entre outros.
- **Fluxo de leitura:**
  1. Usuário aciona o scanner via botão “Ler Bilhete”.
  2. Aparece tela nativa de câmera com caixa de leitura.
  3. Ao detectar um código válido, o plugin retorna um objeto com `text`, `format` e `cancelled`.
  4. Se `cancelled === false`, segue para validação.

### 1.2 Validação Local e Remota
- **Whitelist interna:** verifica se o ID lido está cadastrado no `ListaBrancaProvider`.  
- **Chamada ao backend:** via `HttpdProvider`, envia requisição `GET /validate?ticket={id}` ao servidor configurado em `document.json`.  
- **Regras de negócio:**  
  - Bilhete já usado → rejeitar.  
  - Identificador inválido → rejeitar.  
  - Dispositivo offline → grava em cache local para validar depois.

### 1.3 Feedback ao Usuário
- **Visual:**  
  - Mensagem em overlay no centro da tela (configurada por `timeMessage` em milissegundos).  
  - Cores:  
    - Verde (`#4CAF50`) para sucesso.  
    - Vermelho (`#F44336`) para erro.  
- **Sonoro:**  
  - `success.mp3` tocado em eventos de sucesso.  
  - `error.mp3` tocado em falhas.  
  - Controle via `AudioUtilsProvider`.

### 1.4 Acionamento de Hardware (GPIO)
- **Provider:** `GpiosProvider`.  
- **Configurações:** em `assets/configs/document.json`:  
  ```json
  {
    "receptorOneEnabled": 1,
    "receptorOneId": 1,
    "receptorTwoEnabled": 1,
    "receptorTwoId": 2
  }
  ```  
- **Fluxo de acionamento:**  
  1. Após validação, chama `gpiosProvider.trigger(receptorId)`.  
  2. O Raspberry Pi dispara o pino correspondente, liberando catraca/porta.  
  3. Timeout automático fecha o mecanismo após X segundos.

---

## 2. Modo Múltiplo (MultiplePage)

### 2.1 Descrição
Permite processar vários bilhetes em sequência antes de liberar o mecanismo de acesso.

### 2.2 Limite e Contagem
- **Parâmetro:** `maxTicketsMultiple` em `document.json` (default: 100).  
- **Operação:**  
  1. Usuário escaneia vários códigos, um a um.  
  2. Cada ID válido é armazenado em array interno.  
  3. Ao atingir o total ou ao clicar em “Finalizar”, o sistema:  
     - Dispara GPIO apenas uma vez.  
     - Gera log agrupado no histórico.

### 2.3 Interface
- Lista dinâmica de IDs escaneados.  
- Botões “Adicionar” / “Remover” por item.  
- Indicador de progresso (“X de N”).

---

## 3. Histórico de Acessos (HistoryPage)

### 3.1 Persistência de Logs
- **Storage:** `IonicStorageModule` via `DatabaseProvider`.  
- **Dados armazenados por registro:**  
  - ID do bilhete  
  - Timestamp (ISO 8601)  
  - Status (sucesso / erro)  
  - Modo (único / múltiplo)

### 3.2 Exibição
- Ordenação decrescente por data/hora.  
- Paginação ou scroll infinito.  
- Configuração de duração de exibição de mensagem histórica em `timeMessageHistory`.

### 3.3 Exportação (opcional)
- Botão “Exportar CSV”: gera arquivo com todos os registros, usando plugin `File` e `FileOpener`.

---

## 4. Lista de Memória (MemoryListPage)

### 4.1 Objetivo
Salvar leituras para “marcar para depois” — ideal quando há consulta presencial posterior.

### 4.2 Funcionalidades
- **Adicionar/A… remover manualmente** itens do cache.  
- **Busca e filtro** por texto parcial.  
- **Reenvio** ao backend quando a conexão for restabelecida.

---

## 5. Lista Branca (Whitelist)

### 5.1 Cadastro de IDs
- Tela administrativa para incluir/excluir entradas.  
- Persistência em Firebase Realtime Database ou local via Storage.

### 5.2 Utilização no Fluxo
- Antes de qualquer scan, o provider `ListaBrancaProvider` retorna `true` para IDs pré-aprovados e libera automaticamente sem passar pela validação remota.

---

## 6. Configuração Dinâmica

### 6.1 Arquivo de Configuração
- **Local:** `src/assets/configs/document.json`  
- **Parâmetros principais:**  
  ```jsonc
  {
    "addressServer": "http://<IP>:8085",
    "timeMessage": 1500,
    "timeMessageHistory": 15000,
    "maxTicketsMultiple": 100,
    "receptorOneEnabled": 1,
    "receptorOneId": 1,
    "receptorTwoEnabled": 1,
    "receptorTwoId": 2
  }
  ```
- Carregamento no bootstrap via `APP_INITIALIZER` e `ConfigurationService`.

### 6.2 Vantagens
- Altera endpoints, tempos e habilitação sem recompilar o APK/PWA.  
- Facilita testes A/B e rollout gradual.

---

## 7. Navegação e Interface (UI)

### 7.1 Menu Lateral (Sidemenu)
- Componente único: `SideMenuContentComponent`.  
- **Decorators** (`side-menu-display-text.decorator.ts`) controlam visibilidade baseada em regras de negócio.

### 7.2 Layout Responsivo
- Uso de `<ion-grid>`:  
  ```html
  <ion-grid>
    <ion-row>
      <ion-col size="12" size-md="6" size-lg="4">
        <!-- Card / Conteúdo -->
      </ion-col>
    </ion-row>
  </ion-grid>
  ```
- Breakpoints customizados em `variables.scss`.

### 7.3 Modo Totem / Kiosk
- Meta tag no `index.html`:  
  ```html
  <meta name="viewport" content="user-scalable=no, maximum-scale=1, minimum-scale=1, width=device-width" />
  ```
- CSS global para fullscreen:
  ```css
  body, ion-app {
    height: 100vh;
    overflow: hidden;
  }
  ```

---

## 8. Armazenamento & Offline

### 8.1 Ionic Storage
- Chaves e valores simples para configurações e logs.

### 8.2 Cache de Assets (PWA)
- `service-worker.js` pré-cacheia arquivos estáticos.  
- `manifest.json` habilita instalação como app independente no dispositivo.

### 8.3 Sincronização
- Ao reconectar, `DatabaseProvider` envia registros pendentes ao servidor.

---

## 9. Comunicação com Servidor

### 9.1 HttpdProvider
- Centraliza chamadas REST (`GET`, `POST`, `PUT`).  
- Interceptor HTTP para injetar token de autenticação (se aplicável).

### 9.2 Firebase Realtime Database (opcional)
- Inicializado em `AppModule` via `AngularFireModule.initializeApp(firebaseConfig)`.  
- Observables automáticos para sincronização em tempo real.

---

## 10. Notificações

### 10.1 Firebase Cloud Messaging
- Configurações em `messaging.service.ts`.  
- Permite receber mensagens de manutenção, alertas de sistema ou updates.

### 10.2 Ações customizadas
- No callback de push, exibe alertas ou executa funções de limpeza de cache.

---

## 11. Áudio & Multimídia

### 11.1 Feedback Sonoro
- Uso de `NativeAudio` para pré-carregar e tocar arquivos de áudio locais.
- Suporte a formatos `.mp3` e `.wav`.

### 11.2 Extensões Futuras
- Streaming de vídeos curtos (e.g. instruções de uso).  
- TTS (Text‑to‑Speech) para guiar usuários em totens indevidamente treinados.

---

## 12. Testes & Qualidade

- **Unit Tests:** Jasmine + Karma para providers e componentes críticos.  
- **E2E Tests:** Cypress ou Protractor para fluxos de scan e navegação.
- **Linting:** ESLint com regras TypeScript e Ionic.
- **Documentação:** Compodoc para gerar site de docs baseado em JSDoc.

---

> **Observação:**  
> Este sistema foi projetado para ser flexível, escalável e modular. Cada provider e componente pode ser extendido ou substituído para atender a novos requisitos de hardware ou fluxo de entrada.  

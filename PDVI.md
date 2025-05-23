
## **O que esse server faz?**

### **1. Integração entre WordPress WooCommerce e sistema local**

* Ele **lê pedidos de vendas** feitos na loja WooCommerce (WordPress), acessando diretamente o banco de dados remoto do WooCommerce.
* Sincroniza os pedidos pagos com o banco de dados local do seu sistema 3access (PDVi).

### **2. Gera e gerencia ingressos/vouchers**

* Cada pedido sincronizado gera **tickets/ingressos** físicos ou digitais (com QR Code) no sistema local.
* Esses tickets podem ser impressos, enviados por e-mail e controlados para acesso a eventos ou áreas.

### **3. Controle de estoque e registro de vendas**

* Controla o estoque de cada produto, tanto no WooCommerce quanto no sistema local, evitando vendas duplicadas ou inconsistentes.
* Toda venda é registrada com log de operação.

### **4. Envio de e-mail automático**

* Quando um ticket é gerado, pode ser enviado automaticamente por e-mail para o cliente, com QR Code em anexo.

### **5. Operações de impressão local**

* O servidor pode acionar scripts locais de impressão, integrando diretamente com impressoras via shell script, para imprimir ingressos na hora.

### **6. API local para integração com totem e outros sistemas**

* Expõe vários endpoints HTTP para manipular vendas, tickets, produtos, sessões, usuários, logs, áreas, etc.
* Pode ser usado por totens de acesso, sistemas de catraca ou aplicativos internos para consultar, vender, marcar ingressos como usados e muito mais.

### **7. Administração financeira (caixa, sangria, troco, relatórios)**

* Permite controle de abertura de caixa, retiradas, troco, relatório de vendas por operador/período.

---

## **Resumo visual do fluxo:**

**Venda ocorre no site (WooCommerce) → Pedido é marcado como pago → Server sincroniza com banco local → Gera tickets → (opcional) Imprime/enviou QR Code → Permite uso em totens/acesso físico**

---

## **Principais usos**

* Automatizar emissão e controle de ingressos (eventos, estacionamentos, clubes, etc).
* Conectar vendas online com sistemas físicos de acesso.
* Integrar WooCommerce (WordPress) com sistemas legados ou controle de estoque local.

---

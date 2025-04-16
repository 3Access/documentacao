# Documentação das Regras de Validação de Ingressos

Esta documentação descreve as regras aplicadas para determinar a validade de um ingresso no sistema `ticket_office_in`, conforme implementado na classe `ticket_office_in` do projeto `3a_receptor`. As verificações são realizadas em sequência, e qualquer falha resulta em uma rejeição do ingresso com uma mensagem de erro específica.

## Regras de Validação

### 1. Existência do Ingresso no Estoque
- **Descrição**: O ingresso deve estar registrado na tabela `3a_estoque_utilizavel` com o campo `id_estoque_utilizavel` correspondente ao valor fornecido.
- **Consulta SQL**: `SELECT 3a_estoque_utilizavel.id_estoque_utilizavel FROM 3a_estoque_utilizavel WHERE 3a_estoque_utilizavel.id_estoque_utilizavel = [ticket];`
- **Condição de Validade**: O resultado da consulta deve conter pelo menos um registro.
- **Resultado em Caso de Falha**: Rejeição com código de erro 1 e mensagem `"Ticket não existe no estoque: %1"`.

### 2. Venda do Ingresso
- **Descrição**: O ingresso deve estar associado a uma venda registrada na tabela `3a_log_vendas` com o campo `fk_id_estoque_utilizavel` igual ao valor do ticket.
- **Consulta SQL**: `SELECT 3a_estoque_utilizavel.id_estoque_utilizavel, 3a_log_vendas.data_log_venda FROM 3a_estoque_utilizavel INNER JOIN 3a_log_vendas ON 3a_log_vendas.fk_id_estoque_utilizavel = 3a_estoque_utilizavel.id_estoque_utilizavel WHERE 3a_estoque_utilizavel.id_estoque_utilizavel = [ticket];`
- **Condição de Validade**: O resultado da consulta deve conter pelo menos um registro.
- **Resultado em Caso de Falha**: Rejeição com código de erro 2 e mensagem `"Ticket não foi vendido: %1"`.

### 3. Acesso à Porta Específica
- **Descrição**: O ingresso deve ter permissão para acessar a porta específica (`id_porta_acesso`) e a área (`id_area_acesso`) configuradas no sistema, verificadas por meio das tabelas de relacionamento (`3a_porta_acesso`, `3a_subtipo_area_autorizada`, etc.).
- **Consulta SQL**: Verifica a existência de um registro que conecta o ticket à porta e área especificadas.
- **Condição de Validade**: O resultado da consulta deve conter pelo menos um registro.
- **Resultado em Caso de Falha**: Rejeição com código de erro 3 e mensagem `"Ticket não possui acesso: %1"`.

### 4. Localização nas Tabelas Relacionadas
- **Descrição**: O ingresso deve ser encontrado nas tabelas relacionadas (`3a_log_vendas`, `3a_produto`, `3a_subtipo_produto`, etc.) com os dados de venda, produto, validade, e acesso à porta e ponto.
- **Consulta SQL**: Uma consulta complexa que une várias tabelas para validar a existência do ticket com os parâmetros `id_porta_acesso` e `id_area_acesso`.
- **Condição de Validade**: O resultado da consulta deve conter pelo menos um registro.
- **Resultado em Caso de Falha**: Rejeição com código de erro 4 e mensagem `"Ticket não localizado: %1"`.

### 5. Validade Temporal do Ingresso
- **Descrição**: O ingresso deve estar dentro do período de validade, determinado por uma das seguintes condições:
  - **Mesmo Dia** (`mesmo_dia_validade == 1`): A data de venda (`data_log_venda`) deve ser o mesmo dia atual.
  - **Validade Infinita** (`infinito_validade == 1`): Não há limite de tempo.
  - **Validade por Tempo** (`tempo_validade`): A data de venda mais o tempo de validade (`tempo_validade`) deve ser posterior à data atual.
- **Verificações**:
  - `ticket_validity_same_day`: Compara a data de venda com o dia atual.
  - `ticket_validity_infinite`: Não verifica tempo, considera válido.
  - `ticket_validity_time`: Verifica se a data atual é anterior à data de venda mais `tempo_validade`.
- **Resultado em Caso de Falha**:
  - Código 5: `"Ticket vencido"` (mesmo dia inválido).
  - Código 6: `"Ticket vencido - Ultrapassou tempo de validade: %1"` (tempo expirado).

### 6. Regras de Uso da Porta
- **Descrição**: O ingresso deve respeitar as regras de acesso definidas para a porta, que podem ser:
  - **Tempo Limitado** (`horas_porta_acesso > 0`): O tempo desde a venda mais `horas_porta_acesso` deve ser anterior à data atual.
  - **Mesmo Dia da Porta** (`mesmo_dia_porta_acesso > 0`): A data de venda deve ser o mesmo dia atual.
  - **Acesso Único** (`unica_porta_acesso > 0`): O ticket não deve ter sido usado antes.
  - **Número de Liberações** (`numero_liberacoes > 0`): O número de usos não deve exceder `numero_liberacoes`.
- **Verificações**:
  - `ticket_access_timedoor`: Verifica se o tempo desde a venda mais `horas_porta_acesso` é anterior à data atual.
  - `ticket_access_sameday`: Verifica se a data de venda é o mesmo dia atual.
  - `ticket_access_onlyone`: Verifica se já foi utilizado.
  - `ticket_access_countpass`: Verifica se o número de usos é menor que `numero_liberacoes`.
- **Resultado em Caso de Falha**:
  - Código 8: `"Ticket vencido - Ultrapassou tempo de utilização: %1"`
  - Código 9: `"Ticket vencido - passou dia de utilização: %1"`
  - Código 10: `"Ticket vencido - já foi utilizado: %1"`
  - Código 11: `"Ticket vencido - Passou a quantidade máxima de utilizações: %1"`
  - Código 12: `"Ticket vencido - Ultrapassou tempo de utilização: %1"` (repetição do código 8)
  - Código 7: `"Tipo de ticket não localizado: %1"` (nenhuma regra atendida)

### 7. Uso do Ticket
- **Descrição**: Se todas as verificações anteriores forem bem-sucedidas, o ticket é considerado válido, e as ações de uso são registradas.
- **Ações**:
  - Insere um registro na tabela `3a_log_utilizacao`.
  - Atualiza o campo `utilizado` para `1` na tabela `3a_estoque_utilizavel`.
  - Incrementa a `lotacao_area_acesso` na tabela `3a_area_acesso`.
  - Emite um callback com `"HAVE_ACCESS": true` e o valor do ticket.
- **Condição de Validade**: Todas as regras anteriores devem ser atendidas.

### Pré-requisitos
- **Variáveis Configuráveis**: Os valores `idTotem`, `idArea`, e `idPorta` devem ser definidos antes de iniciar as verificações (via `setIdTotem()`, `setIdArea()`, `setIdPorta()`).
- **Conexão com Banco de Dados**: O sistema assume uma conexão ativa com o banco de dados `QMYSQL3://root@18.207.146.28/3access`. Falhas na conexão invalidam o processo.
- **Dados no Banco**: As tabelas (`3a_estoque_utilizavel`, `3a_log_vendas`, etc.) devem conter os dados correspondentes (ex.: `tempo_validade`, `horas_porta_acesso`).

### Observações
- As mensagens de erro são emitidas com códigos numéricos (1 a 12) para facilitar a depuração.
- O sistema usa a classe `moment` para manipular datas e horários (ex.: `same_day_today`, `add_hours`, `is_after`).
- Qualquer falha em uma das etapas resulta na rejeição do ticket, com uma mensagem específica retornada via `callback_error`.

### Exemplo de Fluxo
1. Um ticket com ID `123` é verificado.
2. **Passo 1**: Consulta no `3a_estoque_utilizavel` retorna um registro → Válido.
3. **Passo 2**: Consulta no `3a_log_vendas` retorna um registro → Válido.
4. **Passo 3**: Consulta de acesso à porta retorna um registro → Válido.
5. **Passo 4**: Localização nas tabelas retorna um registro → Válido.
6. **Passo 5**: Validade é "mesmo dia" e a data de venda é hoje → Válido.
7. **Passo 6**: Regra de "acesso único" verifica que não foi usado antes → Válido.
8. **Passo 7**: Registro de uso é inserido, e o callback retorna `"HAVE_ACCESS": true`.

Se qualquer passo falhar, o processo é interrompido com a mensagem de erro correspondente.

---
*Última atualização: 16/04/2025*

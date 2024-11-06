# Estrutura de Banco de Dados para uma Central de Apps

## Tabelas e Estrutura

### 1. **Usuários**
```sql
CREATE TABLE Usuarios (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    senha VARCHAR(255) NOT NULL,
    data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2. **Apps**
```sql
CREATE TABLE Apps (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    descricao TEXT,
    url_instalacao VARCHAR(255) NOT NULL,
    preco DECIMAL(10, 2) NOT NULL,
    data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **3. Contas de Apps dos Usuários**

```sql
CREATE TABLE ContaApps (
    id SERIAL PRIMARY KEY,
    usuario_id INT REFERENCES Usuarios(id) ON DELETE CASCADE,
    app_id INT REFERENCES Apps(id) ON DELETE CASCADE,
    ativo BOOLEAN DEFAULT TRUE,
    data_ativacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    data_desativacao TIMESTAMP
);
```

### **4. Pagamentos**

```sql
CREATE TABLE Pagamentos (
    id SERIAL PRIMARY KEY,
    usuario_id INT REFERENCES Usuarios(id) ON DELETE CASCADE,
    app_id INT REFERENCES Apps(id) ON DELETE CASCADE,
    valor DECIMAL(10, 2) NOT NULL,
    data_pagamento TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    tipo_pagamento VARCHAR(50) NOT NULL, -- Ex: "mensal", "anual"
    status VARCHAR(20) DEFAULT 'Pendente' -- Ex: "Pendente", "Aprovado", "Cancelado"
);
```

## Considerações

1. **Usuários**: A tabela `Usuarios` armazena informações dos usuários, como nome, email e senha (devidamente criptografada).

2. **Apps**: A tabela `Apps` contém informações sobre os aplicativos disponíveis, como nome, descrição, URL de instalação e preço.

3. **Contas de Apps dos Usuários**: A tabela `ContaApps` relaciona usuários aos aplicativos adquiridos. A coluna `ativo` indica se a conta do app está ativa ou desativada. As datas de ativação e desativação ajudam no gerenciamento do estado do app.

4. **Pagamentos**: A tabela `Pagamentos` registra as transações realizadas pelos usuários para ativar os apps, incluindo informações sobre o valor, data do pagamento, tipo e status.

## Funcionalidades

- **Registro e Login**: Usuários podem se registrar e fazer login na central de apps, onde têm acesso a todos os apps que adquiriram.
  
- **Gerenciamento de Apps**: Usuários podem ativar ou desativar apps, e isso se reflete na tabela `ContaApps`.

- **Processamento de Pagamentos**: A tabela `Pagamentos` permite registrar e acompanhar os pagamentos, facilitando a gestão de assinaturas.

## Possíveis Expansões

- **Logs de Atividades**: Uma tabela adicional para registrar atividades dos usuários em relação aos apps.

- **Avaliações e Comentários**: Uma tabela para que usuários possam avaliar e comentar sobre os apps.

- **Notificações**: Uma tabela para gerenciar notificações relacionadas a pagamentos, vencimentos e promoções.

## Resumo do Fluxo
1. usuário se autentica.
2. usuário visualiza e seleciona um app.
3. usuário solicita a instalação do app.
4. sistema verifica o pagamento.
5. sistema registra a instalação na tabela ContaApps.
6. sistema processa o pagamento, se necessário.
7. sistema notifica o usuário sobre a instalação.
8. interface é atualizada para mostrar o app instalado.

# Gestão de Parceiros na Central de Apps

## Estrutura do Banco de Dados

### 1. **Tabela de Parceiros**
```sql
CREATE TABLE Parceiros (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    codigo_indicacao VARCHAR(50) UNIQUE NOT NULL, -- Código único para indicação
    data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2. **Tabela de Indicações**

```sql
CREATE TABLE Indicacoes (
    id SERIAL PRIMARY KEY,
    parceiro_id INT REFERENCES Parceiros(id) ON DELETE CASCADE,
    usuario_id INT REFERENCES Usuarios(id) ON DELETE CASCADE,
    app_id INT REFERENCES Apps(id) ON DELETE CASCADE,
    data_indicacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3. **Tabela de Comissões**

```sql
CREATE TABLE Comissoes (
    id SERIAL PRIMARY KEY,
    parceiro_id INT REFERENCES Parceiros(id) ON DELETE CASCADE,
    valor DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'Pendente', -- Ex: "Pendente", "Pago"
    data_pagamento TIMESTAMP
);
```
## Fluxo de Backend para Gestão de Parceiros

### 1. **Registro de Parceiro**

- Endpoint: POST /api/parceiros
- Payload:
```json
{
    "nome": "Nome do Parceiro",
    "email": "email@parceiro.com"
}
```
- Ação: Registra um novo parceiro e gera um código de indicação único.
- Processo:
    - Insere um novo registro na tabela Parceiros, gerando um código de indicação.

### 2. **Indicação de Novo Usuário**
- Endpoint: POST /api/indicar-usuario
- Payload:
```json
{
    "codigo_indicacao": "CÓDIGO_DO_PARCEIRO",
    "usuario": {
        "nome": "Nome do Novo Usuário",
        "email": "email@usuario.com",
        "senha": "senhaSegura"
    }
}
```

- Ação: O parceiro indica um novo usuário.
- Processo:
    - Verifica se o código de indicação é válido.
    - Registra o novo usuário na tabela Usuarios.
    - Insere um registro na tabela Indicacoes para rastrear a indicação.

### 3. **Registrar Pagamento de Comissão**
- Endpoint: POST /api/registrar-pagamento
- Payload:
```json
{
    "parceiro_id": 1,
    "valor": 2.00,  -- 20% do valor do app (supondo que o app custa 10.00)
    "status": "Pago"
}
```
- Ação: Registra o pagamento de comissão para o parceiro.
- Processo:
    - Insere um novo registro na tabela Comissoes.
    - Atualiza o status da comissão para "Pago" e registra a data do pagamento.

### 4. **Cálculo da Comissão**
- Ação: Ao registrar um pagamento de um novo app pelo usuário indicado, calcula 20% do valor e registra como comissão.
- Processo:
    - Quando um usuário indicado faz um pagamento, a lógica de backend calcula 20% do valor do app e registra na tabela Comissoes.

## Resumo do Fluxo
1. O parceiro se registra e recebe um código de indicação.
2. O parceiro indica um novo usuário usando o código.
3. O novo usuário é registrado na Central de Apps.
4. A indicação é rastreada na tabela Indicacoes.
5. Quando o usuário indicado efetua um pagamento, a comissão de 20% é calculada.
6. O pagamento da comissão ao parceiro é registrado na tabela Comissoes.

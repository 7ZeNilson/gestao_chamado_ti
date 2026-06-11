# gestao_chamado_ti
Sistema Inteligente de Gestão de Chamados de TI

# Sistema de Chamados de TI com Power Platform, Dataverse e Python

## Visão Geral

Projeto desenvolvido utilizando o ecossistema Microsoft Power Platform para gerenciamento de chamados de TI, contemplando abertura de chamados via chatbot, armazenamento de dados no Dataverse, automações com Power Automate, aprovações gerenciais, notificações por e-mail e geração de indicadores através do Power BI.

Além dos recursos da Power Platform, foi realizada integração com Python para extração, tratamento e preparação dos dados antes do consumo analítico.

---

# Tecnologias Utilizadas

## Camada de Dados
- Microsoft Dataverse

## Automação e Aplicações
- Power Apps (Ecossistema Power Platform)
- Power Automate
- Copilot Studio

## Tratamento e Integração de Dados
- Python
- Pandas
- Requests
- MSAL

## Business Intelligence
- Power BI

---

# Objetivos do Projeto

- Automatizar o processo de abertura de chamados de TI.
- Centralizar informações em uma base única de dados.
- Implementar fluxo de aprovação para governança dos chamados.
- Notificar usuários automaticamente sobre alterações de status.
- Disponibilizar indicadores gerenciais para acompanhamento operacional.
- Integrar Dataverse com Python para tratamento e preparação dos dados.

---

# Arquitetura da Solução

```text
Usuário
   ↓
Copilot Studio
   ↓
Power Automate
   ↓
Dataverse
   ↓
Fluxo de Aprovação
   ↓
Notificações por E-mail
   ↓
Python
   ↓
Power BI
```

---

# 1. Modelagem de Dados no Dataverse

O Dataverse foi utilizado como camada central de armazenamento das informações.

## Tabela Criada

**ChamadoTI**

### Estrutura da Tabela

| Campo | Tipo |
|---------|---------|
| Assunto do Chamado | Coluna Primária |
| Solicitante | Pesquisa (Lookup) |
| Tipo de Problema | Opção |
| Descrição | Texto Multilinha |
| Prioridade | Opção |
| Status | Opção |

### Funcionalidades

- Armazenamento centralizado dos chamados.
- Controle de status do atendimento.
- Classificação por categoria.
- Controle de prioridade.
- Suporte a anexos para evidências.

---

# 2. Desenvolvimento Conversacional com Copilot Studio

O Copilot Studio foi utilizado para captura das informações dos usuários durante a abertura dos chamados.

## Variáveis Utilizadas

### DescricaoInput
Tipo: Texto

### CategoriaInput
Tipo: Escolha

Categorias disponíveis:

- Hardware
- Software
- Rede
- Acesso
- Outros

## Fluxo Conversacional

1. Usuário informa a descrição do problema.
2. Usuário seleciona a categoria.
3. Informações são armazenadas nas variáveis.
4. Dados são enviados ao Power Automate para processamento.

---

# 3. Automação Principal com Power Automate

Fluxo responsável pela criação dos chamados no Dataverse.

## Gatilho

Executar um fluxo no Copilot

### Entradas Recebidas

- DescricaoInput
- CategoriaInput

## Variável Global

Nome:

```text
varNumeroChamado
```

Tipo:

```text
String
```

Objetivo:

Armazenar o identificador do chamado criado independentemente da categoria selecionada.

---

## Estrutura de Decisão (Switch)

A lógica foi construída utilizando a variável:

```text
CategoriaInput
```

### Casos Implementados

- Hardware
- Software
- Rede
- Acesso
- Outros

Em cada caso:

1. Criar registro no Dataverse.
2. Definir tipo de problema correspondente.
3. Capturar o ID gerado.
4. Armazenar o ID em `varNumeroChamado`.

---

## Resposta ao Copilot

Após o encerramento do Switch:

```text
Responder ao Agente
```

Saída:

```text
numeroChamado
```

Valor:

```text
variables('varNumeroChamado')
```

### Benefício

Centralização da resposta do fluxo, reduzindo erros de comunicação entre Power Automate e Copilot Studio.

---

# 4. Automação de Notificação por E-mail

Fluxo independente para envio automático de notificações.

## Gatilho

Quando uma linha é adicionada, modificada ou excluída.

Configurações:

- Tipo de Alteração: Atualização
- Tabela: ChamadoTI
- Escopo: Organization

---

## Regra de Negócio

Condição:

```text
Status = 223870003
```

Valor correspondente:

```text
Resolvido
```

### Objetivo

Enviar e-mail apenas quando o chamado for concluído.

---

## Notificação

Ação utilizada:

```text
Enviar Email (V2)
```

Informações enviadas:

- Número do chamado
- Status
- Data de criação
- E-mail do solicitante

---

# 5. Fluxo de Aprovação

Implementação de uma etapa de validação antes da conclusão do processo.

## Campo Adicionado

### Email Aprovador

Tipo:

```text
Texto
```

### Objetivo

Permitir captura livre do e-mail informado durante a conversa no chatbot.

---

## Copilot Studio

Nova variável:

```text
EmailAprovadorInput
```

Pergunta realizada:

```text
Por favor, informe o e-mail do gestor responsável pela aprovação deste chamado.
```

---

## Processo de Aprovação

### Etapas

1. Chamado criado.
2. Status atualizado para "Em Aprovação".
3. Pesquisa do aprovador.
4. Envio da solicitação de aprovação.
5. Aguardar resposta.

### Decisão

#### Aprovado

Atualização do status do chamado.

#### Reprovado

Atualização do status informando rejeição.

---

# 6. Integração Python com Dataverse

Integração desenvolvida para extração e tratamento dos dados armazenados no Dataverse.

## Registro do Aplicativo

Microsoft Entra ID

Informações utilizadas:

- Client ID
- Client Secret
- Tenant ID

### Finalidade

Permitir autenticação segura do script Python.

---

## Permissões

API utilizada:

```text
Dynamics CRM
```

Permissão:

```text
user_impersonation
```

Também foi concedido consentimento administrativo para acesso ao ambiente.

---

## Usuário do Aplicativo

Configuração realizada no Power Platform:

Função atribuída:

```text
System Administrator
```

Objetivo:

Permitir acesso às tabelas do Dataverse.

---

## Desenvolvimento no Python

Bibliotecas utilizadas:

```python
import pandas as pd
import requests
import msal
```

## Processo Executado

### Autenticação

Utilização do MSAL para obtenção do token de acesso.

### Consumo da API

Consulta à API OData do Dataverse.

### Tratamento

- Conversão do JSON para DataFrame.
- Remoção de colunas de sistema.
- Ajustes de estrutura.
- Preparação dos dados para análise.

---

# 7. Business Intelligence com Power BI

Camada responsável pela visualização e análise dos dados.

## Pipeline de Dados

```text
Dataverse
    ↓
Python
    ↓
Arquivo Tratado
    ↓
Power BI
```

---

## Transformações Realizadas

### Python

- Tratamento de valores nulos.
- Conversão de datas.
- Renomeação de colunas.
- Padronização dos dados.

### Benefícios

- Melhor desempenho do Power BI.
- Menor carga de processamento no Power Query.
- Dados prontos para consumo analítico.

---

## Dashboard Desenvolvido

### Indicadores

#### Cards

- Total de Chamados

#### Gráfico de Pizza

- Status dos Chamados

#### Gráfico de Colunas

- Chamados por Solicitante

---

# Competências Demonstradas

## Dataverse

- Modelagem de dados
- Tabelas customizadas
- Lookup
- Choice Columns
- API OData

## Power Apps / Power Platform

- Construção de soluções corporativas
- Integração entre serviços da plataforma

## Copilot Studio

- Desenvolvimento de fluxos conversacionais
- Captura de variáveis
- Integração com Power Automate

## Power Automate

- Automação de processos
- Fluxos condicionais
- Aprovação corporativa
- Integração com Dataverse
- Notificações por e-mail

## Python

- MSAL
- Requests
- Pandas
- Consumo de APIs REST/OData
- Tratamento e transformação de dados

## Power BI

- Modelagem analítica
- Criação de dashboards
- Indicadores gerenciais
- Integração com dados tratados via Python

---

# Resultado

O projeto demonstra conhecimento prático em:

✅ Dataverse como camada de dados

✅ Copilot Studio para atendimento conversacional

✅ Power Automate para automações e aprovações

✅ Power BI para análise e visualização de dados

✅ Python para integração, automação e tratamento de dados

✅ Integração completa entre os componentes da Microsoft Power Platform

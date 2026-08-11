# Assistente Virtual — Ciência da Computação

Assistente virtual para atendimento aos alunos do curso de Ciência da Computação, integrado ao n8n e utilizando Inteligência Artificial para responder dúvidas e consultar documentos acadêmicos.

## Tecnologias utilizadas

- **n8n 2.33.7** — automação e orquestração dos fluxos.
- **Docker / Docker Compose** — execução e gerenciamento dos serviços.
- **PostgreSQL 16** — banco de dados utilizado pelo projeto.
- **pgvector** — extensão do PostgreSQL utilizada para armazenamento e busca por similaridade dos embeddings.
- **NVIDIA Nemotron** — modelo de IA utilizado para geração das respostas e embeddings.
- **Google Drive** — armazenamento e origem dos documentos utilizados na base de conhecimento.

## Pré-requisitos

Antes de executar o projeto, instale:

- Docker
- Docker Compose
- Git

Também é necessário possuir as credenciais das integrações utilizadas pelo n8n:

- Google Drive
- NVIDIA Nemotron
- PostgreSQL

## Executando o projeto

Clone o repositório:

```bash
git clone <URL_DO_REPOSITORIO>
cd <NOME_DO_REPOSITORIO>
```

Suba os containers:

```bash
docker compose up -d
```

Verifique se os containers estão em execução:

```bash
docker compose ps
```

Para acompanhar os logs do n8n:

```bash
docker compose logs -f n8n
```

O n8n ficará disponível no endereço configurado no `docker-compose.yml` (por padrão, normalmente `http://localhost:5678`).

## Configuração das credenciais no n8n

Após acessar o n8n, configure as seguintes credenciais em:

**Credentials → Add Credential**

### 1. Google Drive

Utilizada para acessar os documentos armazenados no Google Drive e alimentar a base de conhecimento.

Configure a credencial do Google Drive de acordo com o método de autenticação disponibilizado pelo n8n.

Será necessário autorizar a conta Google que possui acesso aos documentos utilizados pelo assistente.

### 2. NVIDIA Nemotron

Utilizada para os recursos de Inteligência Artificial do projeto, incluindo:

- Modelo de linguagem (LLM);
- Geração de embeddings para a base vetorial.

Configure a credencial da NVIDIA utilizando a API Key fornecida pela NVIDIA.

A credencial deve ser associada aos nós que utilizam os modelos NVIDIA Nemotron.

### 3. PostgreSQL

Utilizada pelo n8n para acessar o banco PostgreSQL e pelo fluxo de RAG para armazenar/consultar os dados necessários.

Configure os campos de acordo com o `docker-compose.yml`:

```text
Host: postgres
Port: 5432
Database: <POSTGRES_DB>
User: <POSTGRES_USER>
Password: <POSTGRES_PASSWORD>
SSL: desabilitado (ambiente local)
```

> O valor de `Host` deve corresponder ao nome do serviço PostgreSQL definido no `docker-compose.yml`. Em um ambiente Docker Compose, não utilize `localhost` para a conexão entre containers.

## Importando os workflows

Os workflows versionados ficam no diretório:

```text
n8n/workflows/
```

### Importação via CLI

Importe os workflows diretamente no container do n8n.

Para importar um workflow específico:

```bash
docker exec -u node n8n n8n import:workflow --input=/home/node/workflows/workflow.json
```

Para importar workflows separados de um diretório:

```bash
docker exec -u node n8n n8n import:workflow --separate --input=/home/node/workflows
```

### Exportação via CLI

Exporte fluxos diretamento do container do n8n:
```bash
docker exec -u node n8n n8n export:workflow --all --separate --output=/home/node/workflows
```

## Fluxo do assistente

De forma simplificada, o processamento ocorre da seguinte maneira:

```text
Documentos no Google Drive
        ↓
       n8n
        ↓
  Processamento dos documentos
        ↓
NVIDIA Nemotron Embeddings
        ↓
 PostgreSQL + pgvector
        ↓
     Base vetorial
        ↓
     Pergunta do aluno
        ↓
       n8n
        ↓
     PGVector
        ↓
Documentos relevantes
        ↓
NVIDIA Nemotron LLM
        ↓
      Resposta
```

## Comandos úteis

### Iniciar

```bash
docker compose up -d
```

### Parar

```bash
docker compose down
```

### Reiniciar

```bash
docker compose restart
```

### Ver status

```bash
docker compose ps
```

### Ver logs

```bash
docker compose logs -f
```

### Parar e remover containers

```bash
docker compose down
```

> O comando acima não remove os volumes por padrão.

### Remover containers e volumes

```bash
docker compose down -v
```

**Atenção:** isso remove os volumes, incluindo os dados persistidos do PostgreSQL e do n8n.

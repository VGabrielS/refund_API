# Refund API

API de solicitação e aprovação de reembolso de despesas corporativas. Funcionários criam solicitações de reembolso com comprovante, categoria e valor; gestores listam, pesquisam e consultam todas as solicitações recebidas.

É a API consumida pelo front-end [`refund---front-end`](https://github.com/VGabrielS/refund---front-end).

## Papéis de usuário

- **employee (funcionário)** — papel padrão no cadastro. Faz upload de comprovante e cria solicitações de reembolso.
- **manager (gestor)** — lista e pesquisa todas as solicitações de reembolso.
- Tanto `employee` quanto `manager` podem consultar o detalhe de uma solicitação específica.

## Conceitos de domínio

- **User** — usuário da plataforma, com papel (`role`) `employee` ou `manager`.
- **Refunds** — uma solicitação de reembolso: nome/descrição, categoria (`food`, `others`, `services`, `transport`, `accommodation`), valor e nome do arquivo de comprovante, vinculada ao usuário que a criou.

O modelo de dados completo está em [`prisma/schema.prisma`](prisma/schema.prisma). Banco de dados: **SQLite** (arquivo `prisma/dev.db`).

## Endpoints

| Método | Rota | Acesso | Descrição |
| --- | --- | --- | --- |
| `POST` | `/users` | público | Cria um usuário (`role` padrão `employee`) |
| `POST` | `/sessions` | público | Autentica por e-mail/senha e retorna um JWT |
| `POST` | `/uploads` | `employee` | Envia o arquivo de comprovante (`multipart/form-data`, campo `file`) e retorna o `filename` gerado |
| `POST` | `/refunds` | `employee` | Cria uma solicitação de reembolso (usa o `filename` retornado pelo upload) |
| `GET` | `/refunds` | `manager` | Lista solicitações, com paginação e busca por nome (query `name`, `page`, `perPage`) |
| `GET` | `/refunds/:id` | `employee`, `manager` | Detalhe de uma solicitação |
| `GET` | `/uploads/:filename` | público (estático) | Serve o arquivo de comprovante enviado |

Todas as rotas sob `/refunds` e `/uploads` (exceto o `GET /uploads/:filename` estático) exigem o header `Authorization: Bearer <token>`.

> Upload: arquivos aceitos limitados a `image/jpeg`, `image/jpg` e `image/png`, com tamanho máximo de 3MB (`src/configs/upload.ts`). O arquivo é salvo primeiro em uma pasta temporária e depois movido para a pasta de uploads final.

## Arquitetura

- **Node.js** + **TypeScript**, com **Express**.
- Organização por camadas: `routes` → `controllers` (validação com **Zod** e acesso a dados via **Prisma** diretamente nos controllers).
- **Prisma** como ORM sobre **SQLite**.
- Autenticação **JWT** (`jsonwebtoken`), com `role` e `sub` (id do usuário) embutidos no token; validade padrão de 1 dia (`src/configs/auth.ts`).
- Autorização por papel feita pelo middleware `verifyUserAuthorization` (`src/middlewares/verify-user-authorization.ts`), aplicado por rota.
- Upload de arquivos com **multer** + um provider de armazenamento em disco (`src/providers/disk-storege.ts`).
- Senhas com hash via `bcrypt`.
- Tratamento de erros centralizado (`src/middlewares/error-handling.ts`), com `express-async-errors` capturando erros assíncronos dos controllers e formatando erros de validação do Zod.
- Alias de import `@/*` apontando para `src/*` (configurado em `tsconfig.json`, resolvido nativamente por `tsx`).

## Como rodar localmente

### Pré-requisitos

- Node.js 18+
- npm

### 1. Instalar dependências

```bash
npm install
```

### 2. Rodar as migrations do Prisma

```bash
npx prisma migrate dev
```

Cria o banco SQLite em `prisma/dev.db`.

### 3. Rodar a API em desenvolvimento

```bash
npm run dev
```

API disponível em `http://localhost:3333`.

> Atenção: atualmente a porta (`src/server.ts`) e o segredo do JWT (`src/configs/auth.ts`) estão fixos no código, e não há suporte a variáveis de ambiente (`.env`) neste projeto. Ajuste esses arquivos diretamente caso precise alterá-los — recomenda-se migrar o segredo do JWT para uma variável de ambiente antes de qualquer uso em produção.

## Stack

Node.js, TypeScript, Express, Prisma, SQLite, Zod, JWT, bcrypt, multer.

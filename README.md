# Manancial Máquinas

Sistema de gestão de serviços técnicos, desenvolvido com TanStack Start, React, TypeScript e Tailwind CSS.

## Desenvolvimento local

Requisitos: Node.js e npm.

```sh
npm install
npm run dev
```

O comando exibirá o endereço local do app.

## Publicação na Vercel

O código é publicado na Vercel pelo repositório Git conectado. Enviar alterações para a branch configurada inicia um novo deploy. A configuração do banco e as variáveis de ambiente não são enviadas pelo Git.

Na Vercel, abra **Settings → Environment Variables** e configure estas variáveis para os ambientes desejados:

- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_PUBLISHABLE_KEY`
- `SUPABASE_URL`
- `SUPABASE_PUBLISHABLE_KEY`

Use a URL e a chave pública (publishable/anon) do projeto Supabase. Depois de alterar variáveis, faça um novo deploy para que sejam aplicadas. Nunca coloque uma chave `service_role` ou `sb_secret` em uma variável `VITE_*` ou no navegador.

## Supabase

O Supabase hospeda o banco PostgreSQL, a autenticação e o armazenamento privado dos PDFs. O projeto associado está indicado em `supabase/config.toml`.

### Configuração local

1. Copie `.env.example` para `.env`.
2. Preencha `VITE_SUPABASE_URL` e `SUPABASE_URL` com a URL do projeto, e `VITE_SUPABASE_PUBLISHABLE_KEY` e `SUPABASE_PUBLISHABLE_KEY` com a chave pública.
3. Execute `npm run dev`.

O arquivo `.env` é local e não é enviado pelo Git. Para a publicação, configure as variáveis diretamente nas configurações da Vercel.

### URLs de autenticação

No Supabase, abra **Authentication → URL Configuration**:

- Defina **Site URL** como o domínio principal da aplicação na Vercel.
- Adicione em **Redirect URLs** os endereços usados pelo app, incluindo `https://SEU-DOMINIO/reset-password`. Inclua também os domínios de Preview se for testar deploys de preview.

Após alterar essas URLs, solicite um novo e-mail de redefinição de senha; links enviados anteriormente podem continuar apontando para o endereço antigo.

### Banco de dados

O schema SQL está em `drizzle/migrations/`:

- `0000_migration.sql` cria tabelas, funções, políticas RLS, triggers e o bucket privado `documentos`.
- `0001_automatic_employee_signup_role.sql` garante que novos usuários recebam somente o papel `tecnico`.
- `0002_repair_auth_user_profiles.sql` recria o trigger de cadastro e corrige perfil/papel ausentes para contas existentes sem alterar papéis já atribuídos.

Para um banco Supabase novo, abra o **SQL Editor** do projeto e execute `0000_migration.sql`, `0001_automatic_employee_signup_role.sql` e `0002_repair_auth_user_profiles.sql`, nessa ordem. Confirme que cada arquivo termina sem erros.

Se `public.profiles` e `public.user_roles` já existem, não execute a migração inicial novamente. Verifique o schema e aplique apenas a migração adequada ao estado atual do banco. Caso o bucket ainda não exista, crie-o como privado:

```sql
insert into storage.buckets (id, name, public)
values ('documentos', 'documentos', false)
on conflict (id) do update set public = false;
```

O trigger de cadastro atribui apenas o papel `tecnico`. Conceda o papel `admin` a uma conta autorizada pelo SQL Editor do Supabase, nunca pelo cadastro público.

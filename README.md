
## Segurança e dados públicos

Visualização é pública: nomes, fotos, resultados, equipamentos, observações e inscrições. E-mails e senhas ficam no Supabase Auth, fora das tabelas públicas. Não inclua informações privadas nas observações.

Todas as tabelas possuem RLS. Papéis `anon` e `authenticated` só têm leitura; escritas usam RPCs que validam `auth.uid()`, conta ativa, proprietário e papel administrativo no banco. Assim, chamadas diretas ao Supabase também respeitam as regras. Resultados possuem unicidade atleta/WOD, timestamps automáticos e histórico de alterações acessível só a administradores. A API também valida dados e origem das escritas. A sessão usa os cookies do cliente oficial Supabase SSR, renovados nas consultas da API. Não há chave privilegiada no servidor da aplicação.

O bucket de avatares é público, aceita JPG/PNG/WebP até 2 MB e restringe gravação à pasta do usuário autenticado. Remover a foto do perfil remove a referência pública do perfil; arquivos antigos podem ser removidos pelo proprietário no Storage. Configure backups do Supabase antes de usar com dados reais.

## Publicar na Vercel

1. Envie estas alterações para seu repositório GitHub quando estiverem revisadas. Elas não são publicadas ou enviadas automaticamente por este projeto.
2. Em https://vercel.com, use **Add New → Project**, importe o repositório e escolha o preset **Next.js**, Node 24. Diretório raiz: a pasta com `package.json` (raiz deste repositório). Build: `npm run build`.
3. Adicione `NEXT_PUBLIC_SUPABASE_URL` e `NEXT_PUBLIC_SUPABASE_ANON_KEY` em **Environment Variables** para Production e, se desejado, Preview. Não são necessários tokens Vercel no código.
4. Publique e configure a URL final em **Site URL** e **Redirect URLs** do Supabase. Faça um novo deploy ao alterar variáveis `NEXT_PUBLIC_*`, pois são incorporadas ao build.
5. Valide com duas contas: cadastro e confirmação de e-mail, login, upload, inscrição semanal/mensal, envio e edição de resultado; confirme o ranking em outro navegador. Teste que uma conta comum não consegue escrever resultados de terceiros, promover-se ou editar após o prazo.

Não é possível concluir esse teste remoto ou publicar sem criar o projeto e configurar esses serviços. O modo de exemplo não substitui essa etapa.

## Verificações

```sh
npm test              # regras de ranking + PostgreSQL real em WASM (PGlite), RPCs, RLS e auditoria
npm run typecheck
npm run build
npm start             # servidor de produção; deixe em execução para o comando abaixo
npm run test:e2e      # em outro terminal: Chromium, desktop e celular
```

Os testes do banco criam um schema `auth` mínimo e papéis de teste em memória; não acessam nem alteram seu Supabase. Os testes de Storage validam políticas SQL em tabelas de teste, não a API remota de upload. Os testes de interface cobrem o leaderboard de exemplo e formulários com respostas de API simuladas. A entrega não afirma que confirmação de e-mail, upload ou sincronização entre clientes foram exercitados em um Supabase hospedado.

O Chromium para testes é obtido do pacote npm `@sparticuz/chromium` (Linux). Os testes usam o servidor local na porta 3000; em outro sistema, use uma instalação compatível do Playwright e execute `npx playwright test`. `npm run build` deve preceder `npm start`. Não é necessário Docker para os testes do banco.

## Estrutura

- `app/components`: leaderboard, filtros, WODs, perfis, formulários e painel administrativo.
- `app/api/[...path]/route.ts`: API de leitura pública e escritas autenticadas.
- `app/auth/confirm`: confirmação de cadastro.
- `lib/ranking.ts`: regras puras de classificação usadas na interface e nos testes.
- `lib/validation.ts`: validação de entrada; o banco valida novamente.
- `supabase/migrations`: instalação do banco, RPCs e Storage.
- `supabase/seed.sql`: WODs opcionais de exemplo.
- `tests`, `e2e`: testes de negócio, segurança e navegador.

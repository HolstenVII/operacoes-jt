# J&T Central Operacional — projeto inicial funcional

Aplicação web Next.js com quatro módulos: análise de planilhas, painel de ocorrências urgentes, gerador de QR Codes e identificação de pendências. Inclui cadastro/login, aprovação administrativa e validade de acesso de 30 dias.

## Limites e o que funciona
- Importação local de `.xlsx`, `.xls`, `.csv` e `.ods` usando SheetJS. O conteúdo não é enviado ao servidor.
- Resumo, classificação heurística de colunas/eventos, gráfico e tabelas.
- QR Codes individuais a partir de itens separados por linha, espaço, vírgula ou ponto e vírgula. Se uma única sequência numérica tiver comprimento múltiplo de 12, tenta dividir em blocos de 12.
- Classificação de VIP, 1BD, DNR e ocorrências selecionadas por palavras-chave. Ajuste o dicionário em `lib/analyze.ts` aos nomes reais exportados pela J&T.
- Cadastro Supabase com aprovação, painel de admin, bloqueio e renovação por 30 dias.
- Recuperação de senha via link seguro enviado pelo Supabase. **Não é código OTP customizado**, pois isso exige configuração de template e fluxo de autenticação adicionais.
- Endpoint diário para avisos de vencimento em 3 dias por Resend/Vercel Cron.
- Extensão de navegador opcional que captura texto, tabelas e valores de campos acessíveis da aba J&T autorizada. Ela copia um JSON para a área de transferência; instale e use somente em páginas e dados que você está autorizado a acessar.

**Não existe ainda conexão direta/API oficial de tickets J&T nem garantia de leitura de histórico não disponível na tela.** O conector auxiliar é uma base de captura autorizada, não um monitor automático contínuo. A integração real precisa ser validada no sistema e pode depender de autorização da empresa.

## 1) Criar Supabase
1. Crie um projeto em https://supabase.com.
2. Em SQL Editor, execute todo o arquivo `supabase/schema.sql`.
3. Em Authentication → URL Configuration, configure Site URL e Redirect URLs para `http://localhost:3000`, depois adicione o endereço Vercel quando publicado.
4. Em Authentication → Email, configure confirmação de e-mail conforme preferir.
5. Copie Project URL, anon key e service-role key (a service-role fica apenas em variável de servidor; nunca publique).

## 2) Configurar ambiente
Copie `.env.example` para `.env.local` e preencha:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY` (somente servidor)
- `ADMIN_EMAIL` (registro informativo para seu e-mail)
- `ADMIN_NOTIFICATION_EMAIL`
- `RESEND_API_KEY`
- `CRON_SECRET` (crie uma chave aleatória forte)
- `NEXT_PUBLIC_APP_URL`

## 3) Criar o administrador
1. Inicie o projeto local e cadastre-se usando seu e-mail administrador.
2. No Supabase → Table Editor → `profiles`, localize seu perfil.
3. Altere `role` para `admin`, `status` para `approved`, e deixe `expires_at` vazio.
4. Faça logout e login novamente. O painel Administração aparecerá.
5. Nunca compartilhe sua senha, service-role key ou CRON_SECRET.

## 4) Executar localmente
Instale Node.js LTS (https://nodejs.org), abra o terminal dentro da pasta e execute:

```bash
npm install
npm run dev
```

Acesse http://localhost:3000.

Para validar a compilação:

```bash
npm run build
```

## 5) GitHub + Vercel
1. Crie um repositório GitHub privado (recomendado para este código).
2. Envie todos os arquivos e pastas deste projeto para a raiz do repositório.
3. No Vercel, importe o repositório GitHub.
4. Em Vercel → Settings → Environment Variables, configure todas as variáveis de `.env.example`.
5. Faça Deploy. Configure a URL pública em Supabase Auth → URL Configuration.
6. O `vercel.json` define cron diário para avisos de renovação. A variável `CRON_SECRET` deve ser configurada no Vercel.

## Extensão de captura autorizada
1. No Chrome, abra `chrome://extensions` e ative Developer mode.
2. Clique em Load unpacked e selecione a pasta `browser-extension`.
3. Abra a página J&T permitida e clique no ícone da extensão.
4. A captura coloca um JSON na área de transferência. Cole em editor de texto para revisar. Não captura senhas de login; campos de senha são listados apenas como campo mascarado, e conteúdo de sites é limitado a texto e tabelas acessíveis.

## Segurança / considerações
- O Supabase Auth controla login. RLS limita o perfil comum ao próprio registro. Admin inicial deve ser promovido manualmente.
- Os arquivos importados são processados no navegador e não persistidos.
- Use dados fictícios para testar antes de processar dados pessoais reais.
- A classificação por texto é heurística, pode falhar quando cabeçalhos/status são diferentes. Conferir resultados antes de tomar decisões operacionais.
- O aviso cron não envia e-mail se Resend não estiver configurado. Para produção, configure domínio remetente verificado.
- Recomendado manter repo privado e não expor `.env.local`.

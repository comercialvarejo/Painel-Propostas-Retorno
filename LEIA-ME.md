# Painel de Propostas — como colocar no ar

O painel tem 2 arquivos:

- `index.html`: a página que vai para o GitHub.
- `firestore.rules`: as regras de segurança que vão para o Firebase.

Quem decide o que cada pessoa vê é o Firebase, não a página. Por isso o repositório do GitHub pode ser público: não há nenhuma senha no código.

| Quem | O que vê | O que faz |
| --- | --- | --- |
| Vendedor com e-mail @gtf.com.br | Só as próprias propostas | Entra com a conta Microsoft (ou e-mail e senha) e envia |
| Representante com outro e-mail (Gmail, Hotmail…) | Só as próprias propostas | Entra com e-mail e senha, depois de liberado por um aprovador |
| Os 4 aprovadores | Todas as propostas, numa lista única | Dão o retorno, liberam representantes e cadastram supervisores |
| Qualquer outra pessoa | Nada | Até cria uma senha, mas é barrada na entrada |

Tempo total: cerca de 40 minutos. O passo 3 precisa de alguém com acesso de administrador ao Microsoft 365 da empresa.

---

## 1. Criar o projeto no Firebase (grátis)

1. Acesse https://console.firebase.google.com com uma conta Google e clique em **Criar projeto**. Nome: `painel-propostas-gtf`. O Google Analytics pode ficar desligado.
2. Na página do projeto, clique no ícone **Web** (`</>`) para registrar um app. Apelido: `painel`. **Não** marque o Firebase Hosting.
3. O Firebase mostra um bloco `firebaseConfig`. Copie estes 4 valores para o bloco `CONFIG` no começo do `<script>` do `index.html`:

```js
firebase: {
  apiKey: "AIza...",
  authDomain: "painel-propostas-gtf.firebaseapp.com",
  projectId: "painel-propostas-gtf",
  appId: "1:1234...:web:abcd..."
},
```

## 2. Ativar o login

No Firebase, abra **Authentication** e clique em **Vamos começar**.

1. Em **Método de login**, ative **E-mail/senha**. É o login reserva, e também serve se o passo 3 atrasar.
2. Ainda em **Método de login**, clique em **Microsoft** e ative. O Firebase mostra uma **URL de callback**, parecida com `https://painel-propostas-gtf.firebaseapp.com/__/auth/handler`. Copie essa URL: ela é usada no passo 3.
3. Em **Configurações > Domínios autorizados**, clique em **Adicionar domínio** e coloque `SEU-USUARIO.github.io`, trocando pelo seu usuário do GitHub.

## 3. Registrar o app no Microsoft 365 (Entra ID)

Este passo precisa do administrador do Microsoft 365. Ele faz o botão **Entrar com a conta Microsoft da empresa** funcionar, e aí o vendedor usa o mesmo login do Outlook, sem criar outra senha.

1. Acesse https://entra.microsoft.com e abra **Aplicativos > Registros de aplicativo > Novo registro**.
   - Nome: `Painel de Propostas`
   - Tipos de conta: **Somente contas neste diretório organizacional**. Assim, só contas da Gtf entram.
   - URI de redirecionamento: tipo **Web**, com a URL de callback copiada no passo 2.
2. Depois de registrar, copie da página **Visão geral**:
   - **ID do aplicativo (cliente)**
   - **ID do diretório (locatário)**
3. Em **Certificados e segredos**, clique em **Novo segredo do cliente**, com validade de 24 meses, e copie o **Valor**. Ele só aparece uma vez.
4. Volte ao Firebase, em **Authentication > Método de login > Microsoft**, e cole o **ID do aplicativo** e o **segredo**. Salve.
5. No `index.html`, cole o **ID do diretório (locatário)** em `microsoft.tenant`.

Se o administrador não puder fazer isso agora, coloque `microsoft: { ativo: false, ... }` no `CONFIG`. Todos entram com e-mail e senha: cada pessoa cria o acesso com o e-mail @gtf.com.br e confirma pelo link que chega no Outlook.

> Anote na agenda a data de vencimento do segredo do cliente. Quando ele vencer, crie um novo e cole no Firebase, senão o login pela Microsoft para de funcionar.

## 4. Criar o banco de dados e colar as regras

1. No Firebase, abra **Firestore Database > Criar banco de dados**.
   - Local: `southamerica-east1 (São Paulo)`
   - Modo: **produção**
2. Abra a aba **Regras**, apague o que estiver lá e cole o conteúdo inteiro de `firestore.rules`.
3. No começo do arquivo, troque os e-mails pelos dos 4 aprovadores, sempre em letras minúsculas:

```
'paulo.neto@gtf.com.br',
'miguel@gtf.com.br',
'supervisor1@gtf.com.br',
'supervisor2@gtf.com.br'
```

4. Clique em **Publicar**.

Para trocar um aprovador no futuro, basta editar essa lista e publicar de novo. Não precisa mexer na página.

## 5. Publicar no GitHub Pages

1. No GitHub, crie um repositório novo, por exemplo `propostas`. Ele pode ser **público**.
2. Envie o `index.html` já com o `CONFIG` preenchido: **Add file > Upload files**.
3. Em **Settings > Pages**, escolha **Deploy from a branch**, branch `main`, pasta `/ (root)`, e salve.
4. Em 1 ou 2 minutos, o painel fica no ar em `https://SEU-USUARIO.github.io/propostas/`. Esse é o link que você manda para todo mundo.

## 6. Liberar os representantes

Representante não tem e-mail @gtf.com.br, então precisa ser liberado antes do primeiro acesso. Sem isso, qualquer pessoa na internet conseguiria criar uma senha e mandar propostas.

1. Um aprovador abre o painel, vai na aba **Aprovar** e clica em **Representantes**.
2. Preenche o nome e o e-mail que o representante vai usar e clica em **Liberar acesso**.
3. O painel monta uma mensagem pronta com o link e as instruções. É só copiar e mandar no WhatsApp.
4. O representante clica em **Criar acesso**, cria a senha e confirma pelo link que chega no e-mail dele.

Para cortar o acesso de alguém, clique em **Bloquear** na mesma lista. As propostas antigas dessa pessoa continuam no histórico.

> Por que e-mail e não só usuário? Com e-mail, o próprio representante recupera a senha em **Esqueci a senha**, sem depender de ninguém. Com um usuário inventado, uma senha esquecida exigiria criar outro acesso e as propostas antigas ficariam separadas.

## 7. Testar antes de liberar para todos (10 minutos)

- [ ] Vendedor A entra e envia uma proposta com 2 itens.
- [ ] Vendedor B entra e **não** vê a proposta do A.
- [ ] Um aprovador entra, vê a aba **Aprovar** com a proposta do A e dá uma contraproposta.
- [ ] O vendedor A vê o retorno e aceita a contraproposta.
- [ ] Um aprovador cadastra os supervisores em **Resumo e histórico**.
- [ ] Um aprovador libera um representante de teste (pode ser um Gmail seu). Ele cria o acesso, confirma o e-mail e envia uma proposta.
- [ ] Alguém com um e-mail **não** liberado cria senha e vê a tela "Acesso ainda não liberado".
- [ ] Deixe o CNPJ em branco numa proposta: ela deve ser enviada normalmente.

## Bom saber

- **Custo:** o plano gratuito do Firebase (Spark) atende com folga esse volume. Não precisa cadastrar cartão.
- **Avisos:** o som e o aviso na tela do computador só funcionam com o painel aberto em alguma aba do navegador. Vale deixar uma aba fixada.
- **Versão da biblioteca:** a página usa o Firebase 12.19.0. Se um dia aparecer erro ao carregar, troque `12.19.0` nas 3 linhas `import` pela versão mais recente listada em https://firebase.google.com/support/release-notes/js.
- **A `apiKey` não é segredo:** ela só identifica o projeto. Quem protege os dados são as regras do passo 4.
- **Excluir propostas:** por segurança, ninguém exclui pela página. Se precisar, exclua pelo Firebase, em **Firestore Database > Dados**.

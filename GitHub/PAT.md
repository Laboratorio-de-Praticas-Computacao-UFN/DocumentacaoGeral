# GitHub PAT (Personal Access Token)

### O que é o PAT e por que ele existe?
O **PAT** (*Personal Access Token* ou Token de Acesso Pessoal) funciona como uma senha alternativa e altamente segura para autenticar ferramentas externas na sua conta do GitHub. Ele é amplamente utilizado para dar permissão ao seu terminal de comandos (via Git), ao seu editor de código (como o VS Code) ou a scripts de automação.

Até agosto de 2021, o GitHub permitia que desenvolvedores fizessem operações como `git push` ou `git pull` digitando a senha tradicional da conta diretamente no terminal. Por motivos de segurança, o uso de senhas básicas foi totalmente descontinuado para essas operações. O PAT foi introduzido para resolver esse problema: ele substitui a senha por uma chave criptográfica longa que pode ser rastreada, limitada e revogada a qualquer momento sem afetar o login principal do usuário no site.

---

### Classic vs. Fine-Grained – Qual usar?
Ao criar um token, o GitHub oferece duas abordagens. 


#### 1. Tokens Classic (Clássicos)
* **Como funcionam:** Utilizam o modelo antigo de escopo amplo. Se você marcar a permissão de repositório (`repo`), esse token terá acesso irrestrito a **todos** os repositórios públicos e privados atrelados à sua conta de uma só vez.
* **Riscos:** Eles violam o princípio básico de segurança do menor privilégio. 
   * Se um token clássico vazar ou for exposto por acidente, o invasor ganhará as chaves de acesso de todo o seu ecossistema no GitHub.
   * Além disso, eles permitem a criação de credenciais que nunca expiram.

#### 2. Tokens Fine-Grained (Granulares)
* **Como funcionam:** É o modelo moderno e a abordagem padrão recomendada hoje. Em vez de liberar toda a sua conta, você escolhe cirurgicamente **quais repositórios específicos** aquele token poderá acessar. As permissões também são detalhadas por recurso (por exemplo: você pode permitir apenas ler o código, sem o direito de modificar as configurações do repositório).
* **Vantagens:** Eles exigem obrigatoriamente uma data de expiração (com limite máximo de um ano) e reduzem drasticamente o impacto de um possível vazamento.

A recomendação oficial é **sempre utilizar os Fine-Grained Tokens**. Você só deve optar pelo modelo *Classic* se estiver integrando o GitHub com alguma ferramenta legada muito antiga ou com scripts que ainda não possuam compatibilidade técnica com o novo formato de escopo granular.

---

### Como Configurar (Passo a Passo)

Para gerar um token seguro no formato moderno (*Fine-grained*):

1. **Acesse as configurações:** Entre no GitHub, clique na sua foto de perfil no canto superior direito e selecione **Settings** (Configurações).
2. **Developer Settings:** Role o menu lateral esquerdo até o final da página e clique na última opção: **Developer settings**.
3. **Menu de Tokens:** Clique em **Personal access tokens** para expandir as opções e selecione **Fine-grained tokens**. Na sequência, clique em **Generate new token**.
4. **Identificação e Validade:** Dê um nome descritivo ao token (ex: `Notebook-Pessoal-Terminal`) e define um prazo de validade (o padrão de segurança recomendado varia entre 30 e 90 dias).
5. **Definição de Escopo:** 
   * Na seção *Repository access*, mude para **Only select repositories** (Apenas repositórios selecionados) e selecione na lista o projeto exato em que vai trabalhar.
   * Na seção *Permissions*, abra a aba **Repository permissions**. Para conseguir enviar e baixar códigos pelo terminal, localize o item **Contents** (Conteúdo) e mude o nível de permissão para **Read and Write** (Leitura e Escrita).
6. **Finalizar:** Role até o fim da página e clique em **Generate token**.

---

### Alerta de Segurança e Uso

Após a geração, o GitHub exibirá o token na tela. **Copie esse código imediatamente e guarde-o em um local seguro** (como um gerenciador de senhas ou no gerenciador de credenciais do seu sistema operacional). O GitHub **não mostrará esse código novamente** por questões de privacidade. Se você fechar ou atualizar a página sem salvar, precisará deletar esse token e gerar um novo.

Quando o seu terminal solicitar o preenchimento de `Password for 'https://github.com'`, basta colar o token copiado no lugar da senha e pressionar Enter para concluir a autenticação.
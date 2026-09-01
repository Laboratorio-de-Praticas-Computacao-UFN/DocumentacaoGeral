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

--- 
## Permissões
Estas permissões controlam o acesso a recursos dentro de um repositório específico.

### Contents (Conteúdo)
* **O que faz:** Controla o acesso de leitura e/ou escrita aos arquivos, pastas, branches, commits e tags do repositório.
* **Quando usar:** Essencial para clonar repositórios privados, dar `git push`, baixar código-fonte via API ou gerenciar arquivos de configuração programaticamente.

### Metadata (Metadados)
* **O que faz:** Permite ler e escrever informações básicas sobre o repositório (como descrição, tópicos e URL).
* **Quando usar:** Geralmente exigida por padrão como uma permissão de leitura obrigatória para quase qualquer token que interaja com o repositório.

### Pull Requests
* **O que faz:** Permite visualizar, criar, comentar, atualizar e realizar o *merge* de Pull Requests, além de gerenciar revisões.
* **Quando usar:** Necessário para bots de code review, linters integrados, ferramentas de automação de fluxo de trabalho ou assistentes de IA que abrem PRs.

### Issues
* **O que faz:** Gerencia a leitura, criação, edição, fechamento e rotulagem (*labels*) de *issues* (problemas/tarefas).
* **Quando usar:** Para sincronizadores de tarefas externos (como Jira ou Linear), bots de triagem ou automações de suporte.

### Actions
* **O que faz:** Permite ler, disparar manualmente (`workflow_dispatch`), cancelar ou deletar execuções de fluxos do GitHub Actions.
* **Quando usar:** Em painéis de monitoramento externos, CLIs que disparam builds ou ferramentas de CI/CD que gerenciam pipelines.

### Commit statuses (Status de commits)
* **O que faz:** Permite ler e criar status associados a commits específicos (ex: aprovar ou reprovar um commit com base em testes externos).
* **Quando usar:** Usado por serviços de integração contínua externos (como Jenkins ou CircleCI) para reportar o sucesso/falha de builds.

### Deployments
* **O que faz:** Gerencia eventos de implantação (*deployments*) e seus respectivos status associados a ambientes.
* **Quando usar:** Para ferramentas de entrega contínua (CD) ou plataformas de hospedagem que notificam o GitHub sobre o status de um deploy.

### Packages (Pacotes)
* **O que faz:** Permite publicar e baixar pacotes no GitHub Packages (como npm, Docker, Maven, etc.).
* **Quando usar:** Em pipelines de publicação de artefatos de software ou ao instalar dependências privadas hospedadas no repositório.

### Webhooks
* **O que faz:** Permite criar, listar e modificar webhooks configurados no repositório.
* **Quando usar:** Para aplicações que precisam configurar ganchos de eventos dinamicamente sem intervenção manual na interface web.

### Security events (Eventos de segurança)
* **O que faz:** Lê e grava alertas de segurança, relatórios do Dependabot e resultados de varreduras de código (`Code Scanning`).
* **Quando usar:** Para ferramentas de análise estática de segurança (SAST) ou painéis de vulnerabilidades.

### Discussions
* **O que faz:** Gerencia a leitura e escrita de discussões e comentários no repositório.
* **Quando usar:** Para bots de comunidade ou ferramentas de moderação automática em repositórios que utilizam a aba de Discussões.

### Environments
* **O que faz:** Gerencia os ambientes de implantação do repositório (regras de proteção, revisores, etc.).
* **Quando usar:** Em automações avançadas de infraestrutura e gerenciamento de deploys por estágio.

### Pages
* **O que faz:** Gerencia as configurações e o status de publicação do GitHub Pages.
* **Quando usar:** Para scripts que configuram ou automatizam o deploy de sites estáticos.

---

## 2. Permissões de Organização (Organization Permissions)

Estas permissões controlam o acesso a nível de empresa ou organização inteira no GitHub.

* **Members:** Gerencia a visibilidade e associação de membros na organização.
* **Teams:** Gerencia equipes, permissões de acesso a repositórios e composição de grupos corporativos.
* **Organization hooks:** Gerencia webhooks configurados no nível da organização inteira.
* **Self-hosted runners:** Gerencia servidores de execução próprios (runners) para o GitHub Actions na organização.

---

## 3. Permissões de Usuário (User Permissions)

Estas permissões controlam dados e recursos da sua própria conta pessoal de usuário.

* **Profile:** Gerencia dados públicos e privados do seu perfil pessoal.
* **Gists:** Permite criar, editar e excluir seus Gists públicos e privados.
* **SSH/GPG keys:** Gerencia suas chaves de segurança para autenticação e assinatura de commits.

---

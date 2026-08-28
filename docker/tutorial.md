# Tutorial: Publicar Imagem Docker no GitHub (GHCR) e Implantar no Portainer

Este guia prático demonstra como transformar um container local em uma imagem, publicá-la no **GitHub Container Registry (GHCR)** e realizar o deploy em um servidor gerenciado pelo **Portainer**.

---

## 📋 Pré-requisitos

1. Ter o **Docker** instalado localmente e no servidor remoto.
2. Uma conta no **GitHub**.
3. Acesso administrativo ao painel do **Portainer**.

---

## 🛠️ Passo 1: Criar um Personal Access Token (PAT) no GitHub

Como a imagem ficará hospedada no GitHub, você precisa de uma chave de acesso (Token) para que o Docker local e o Portainer consigam se autenticar.

1. No GitHub, clique na sua foto de perfil (canto superior direito) e vá em **Settings** (Configurações).
2. No menu lateral esquerdo, role até o final e clique em **Developer settings**.
3. Clique em **Personal access tokens** > **Tokens (classic)**.
4. Clique em **Generate new token** > **Generate new token (classic)**.
5. Defina um nome em **Note** (ex: `Acesso-Portainer-Docker`).
6. Marque a permissão:
   * `write:packages` (isso selecionará automaticamente `read:packages`).
7. Role até o final da página e clique em **Generate token**.
8. **Copie o token gerado imediatamente e guarde-o em um local seguro.** Ele não será exibido novamente.

---

## 🐳 Passo 2: Criar a Imagem a partir do Container

Se você realizou modificações dentro de um container ativo e precisa transformá-lo em uma imagem reutilizável, utilize o comando `docker commit`.

1. Identifique o ID ou o nome do seu container atual:
   ```bash
   docker ps
   ```

2. Crie a nova imagem apontando para o registro do GitHub (`ghcr.io`):
   ```bash
   docker commit <ID_OU_NOME_DO_CONTAINER> ghcr.io/<SEU_USUARIO_GITHUB>/<NOME_DO_REPOSITORIO>:<TAG>
   ```
   *Substitua os valores entre `< >` pelos seus dados reais.*
   
   *Exemplo prático:*
   ```bash
   docker commit meu_container_web ghcr.io/joaosilva/meu-projeto-app:latest
   ```

---

## 🚀 Passo 3: Autenticar e Enviar para o GitHub (Push)

1. No seu terminal local, efetue o login no servidor do GitHub utilizando o token criado no Passo 1:
   ```bash
   echo "SEU_TOKEN_PAT" | docker login ghcr.io -u <SEU_USUARIO_GITHUB> --password-stdin
   ```

2. Envie a imagem para o seu repositório no GitHub:
   ```bash
   docker push ghcr.io/<SEU_USUARIO_GITHUB>/<NOME_DO_REPOSITORIO>:<TAG>
   ```

3. *Verificação:* Acesse o seu perfil ou a página do seu repositório no GitHub e clique na aba **Packages** para confirmar que a imagem foi listada com sucesso.

---

## 🕸️ Passo 4: Configurar o Registro Privado no Portainer

Por padrão, as imagens enviadas ao GHCR são privadas. Para que o Portainer consiga baixá-la, precisamos cadastrar a credencial do GitHub nele.

1. Acesse o painel do seu **Portainer**.
2. No menu lateral esquerdo, clique em **Registries**.
3. Clique no botão **Add registry** (canto superior direito).
4. Selecione a opção **Custom Registry**.
5. Preencha os seguintes campos:
   * **Name:** `GitHub Registry` (ou um nome de sua preferência)
   * **Registry URL:** `ghcr.io`
   * **Authentication:** Ative a chave (mude para *On*)
   * **Username:** Seu nome de usuário do GitHub
   * **Password:** O seu **Personal Access Token (PAT)** (gerado no Passo 1)
6. Clique em **Add registry** para salvar.

---

## 📦 Passo 5: Criar o Container no Portainer

Agora que o Portainer tem acesso à sua imagem, basta criar o container no servidor.

### Opção A: Via Interface Visual (Containers)
1. No menu lateral do Portainer, clique em **Containers**.
2. Clique em **Add container**.
3. Na seção **Image configuration**:
   * **Registry:** Selecione o `GitHub Registry` (criado no Passo 4).
   * **Image:** Digite o restante do caminho da imagem (ex: `<SEU_USUARIO_GITHUB>/<NOME_DO_REPOSITORIO>:<TAG>`).
4. Configure as portas de rede (Network ports), volumes e variáveis de ambiente conforme a necessidade da sua aplicação.
5. Clique em **Deploy the container**.

### Opção B: Via Docker Compose (Stacks)
Se preferir gerenciar usando código (Docker Compose), você pode utilizar a funcionalidade **Stacks**:
1. No menu lateral do Portainer, clique em **Stacks** > **Add stack**.
2. No editor web, monte a estrutura do seu arquivo `docker-compose.yml`:
   ```yaml
   version: '3.8'
   services:
     minha-app:
       image: ghcr.io/<SEU_USUARIO_GITHUB>/<NOME_DO_REPOSITORIO>:<TAG>
       ports:
         - "8080:80" # Ajuste conforme necessário
       restart: always
   ```
3. Role até o final da página e clique em **Deploy the stack**. O Portainer usará automaticamente a credencial que salvamos no passo anterior para puxar a imagem.

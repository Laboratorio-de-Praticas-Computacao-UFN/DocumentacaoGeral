# GitHub Packages e GHCR

## 1. Visão geral

O **GitHub Packages** é um serviço do GitHub destinado a hospedar, armazenar e distribuir **pacotes e artefatos de software**. Ele permite que projetos publiquem dependências, bibliotecas e imagens de containers diretamente na infraestrutura do GitHub.

O **GHCR (GitHub Container Registry)** é o registro de containers do GitHub Packages. Portanto:

> **GitHub Packages é o serviço/ecossistema de pacotes; GHCR é o registro usado especificamente para imagens de containers.**

---

## 2. Relação entre GitHub, GitHub Packages e GHCR

É importante não confundir as funções:

```text
GitHub
│
├── Repository
│   └── Código-fonte, Dockerfile, configurações etc.
│
└── GitHub Packages
    │
    ├── Container Registry (GHCR)
    │   └── Imagens Docker/OCI
    │
    ├── npm
    │   └── Pacotes JavaScript/Node.js
    │
    ├── Maven
    │   └── Pacotes Java
    │
    ├── Gradle
    │   └── Pacotes para Gradle
    │
    ├── NuGet
    │   └── Pacotes .NET/C#
    │
    └── RubyGems
        └── Pacotes Ruby
```

Assim, o **GHCR não é sinônimo de GitHub Packages**. Ele é uma parte do GitHub Packages.

---

# 3. O que é GitHub Packages?

O GitHub Packages permite armazenar e distribuir diferentes tipos de pacotes utilizados no desenvolvimento de software.

Entre os registros disponibilizados pelo GitHub estão:

- **Container Registry (GHCR)** — imagens de containers;
- **npm** — pacotes JavaScript e Node.js;
- **Maven** — pacotes Java;
- **Gradle** — pacotes utilizados com Gradle;
- **NuGet** — pacotes .NET/C#;
- **RubyGems** — pacotes Ruby.

A principal vantagem é poder integrar o armazenamento dos pacotes ao fluxo de desenvolvimento já existente no GitHub.

---

# 4. O que é GHCR?

**GHCR** significa **GitHub Container Registry**.

É o registro de containers do GitHub. Ele permite publicar, armazenar e baixar **imagens de containers**, como imagens Docker.

O endereço do registro é:

```text
ghcr.io
```

Uma imagem pode possuir um endereço semelhante a:

```text
ghcr.io/usuario/meu-projeto:latest
```

Nesse exemplo:

- `ghcr.io` → endereço do GitHub Container Registry;
- `usuario` → proprietário da imagem;
- `meu-projeto` → nome da imagem/pacote;
- `latest` → tag da imagem.

---

# 5. O que é armazenado no GHCR?

O GHCR é utilizado principalmente para armazenar **imagens de containers**.

Por exemplo:

```text
ghcr.io/usuario/meu-sistema:latest
ghcr.io/usuario/meu-sistema:v1.0
ghcr.io/usuario/meu-sistema:v2.0
```

Cada uma pode representar uma versão diferente da aplicação.

Uma imagem pode ser construída a partir de um projeto contendo:

```text
meu-projeto/
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── manage.py
└── aplicação/
```

O código e o Dockerfile ficam no repositório, enquanto a imagem construída pode ser publicada no GHCR.

---

# 6. GitHub Repository x GitHub Packages

Essa diferença é fundamental.

## Repository

O repositório normalmente armazena:

- código-fonte;
- Dockerfile;
- arquivos de configuração;
- documentação;
- código de infraestrutura;
- workflows do GitHub Actions.

Exemplo:

```text
GitHub Repository
│
├── Dockerfile
├── requirements.txt
├── manage.py
└── código da aplicação
```

## GitHub Packages / GHCR

O GHCR armazena a **imagem de container construída a partir desse projeto**:

```text
GitHub Packages
└── GHCR
    └── meu-sistema:latest
```

Portanto:

> O repositório guarda o projeto e o código-fonte; o GHCR pode guardar a imagem de container produzida a partir desse projeto.

---

# 7. O que é uma imagem Docker?

Uma **imagem Docker** é um pacote imutável utilizado como base para criar containers.

De forma simplificada:

```text
Código
   │
   ▼
Dockerfile
   │
   │ docker build
   ▼
Imagem Docker
   │
   │ docker push
   ▼
GHCR
```

Depois, outra máquina pode baixar a imagem:

```text
GHCR
   │
   │ docker pull
   ▼
Imagem Docker
   │
   │ docker run
   ▼
Container
```

---

# 8. Docker Hub x GHCR

O GHCR possui uma função semelhante ao Docker Hub no sentido de armazenar e distribuir imagens de containers.

| Característica | GHCR | Docker Hub |
|---|---|---|
| Armazena imagens Docker | Sim | Sim |
| Faz `docker push` | Sim | Sim |
| Faz `docker pull` | Sim | Sim |
| Integração com GitHub | Excelente | Externa ao GitHub |
| Integração com GitHub Actions | Sim | Sim |
| Endereço | `ghcr.io` | `docker.io` |

Exemplo no GHCR:

```text
ghcr.io/usuario/meu-projeto:latest
```

---

# 9. Como publicar uma imagem no GHCR

Um fluxo básico pode ser realizado com os seguintes passos.

## 9.1 Construir a imagem

A partir de um projeto que possui um `Dockerfile`:

```bash
docker build -t meu-projeto .
```

Isso cria uma imagem localmente.

---

## 9.2 Fazer login no GHCR

O Docker precisa ser autenticado no registro:

```bash
docker login ghcr.io
```

Será solicitado o usuário e uma credencial de autenticação.

Uma **Personal Access Token (PAT)** pode ser utilizada para essa autenticação quando configurada com as permissões necessárias.

---

## 9.3 Criar uma tag para a imagem

A imagem precisa ser identificada com o endereço do GHCR:

```bash
docker tag meu-projeto ghcr.io/SEU_USUARIO/meu-projeto:latest
```

---

## 9.4 Enviar a imagem

Depois:

```bash
docker push ghcr.io/SEU_USUARIO/meu-projeto:latest
```

A imagem será publicada no GHCR.

---

# 10. Como baixar uma imagem do GHCR

Em outra máquina, pode-se baixar uma imagem utilizando:

```bash
docker pull ghcr.io/SEU_USUARIO/meu-projeto:latest
```

Depois, a imagem pode ser executada:

```bash
docker run ghcr.io/SEU_USUARIO/meu-projeto:latest
```

Dependendo da configuração de portas, volumes, variáveis de ambiente e outros recursos, o comando de execução pode ser mais complexo.

---

# 11. Tags

As tags permitem identificar versões diferentes de uma imagem.

Exemplo:

```text
ghcr.io/usuario/meu-projeto:latest
ghcr.io/usuario/meu-projeto:v1.0
ghcr.io/usuario/meu-projeto:v1.1
ghcr.io/usuario/meu-projeto:v2.0
```

A tag `latest` é convencionalmente utilizada para indicar uma versão considerada atual, mas não significa necessariamente que seja a versão mais recente em todos os fluxos.

Para ambientes de produção, tags específicas de versão podem facilitar o controle e a reprodução dos deployments.

---

# 12. PAT e GHCR

**PAT** significa **Personal Access Token**.

É uma credencial utilizada para autenticar determinadas operações no GitHub.

A PAT não é exclusiva do GHCR. Ela pertence ao sistema de autenticação do GitHub e pode receber diferentes permissões.

Para operações relacionadas a pacotes, podem existir permissões como:

- `read:packages` — leitura/baixar pacotes;
- `write:packages` — publicar pacotes;
- `delete:packages` — excluir pacotes.

As permissões disponíveis e o comportamento podem variar de acordo com o tipo de token e a configuração da conta/repositório.

---

# 13. Fine-grained PAT

O GitHub disponibiliza **fine-grained personal access tokens**, que permitem restringir o acesso com maior granularidade.

Em vez de criar uma credencial com acesso amplo, pode-se limitar:

- organização/recurso;
- repositórios;
- permissões;
- período de validade.

Isso segue o princípio:

> **Conceder somente as permissões necessárias para realizar a tarefa.**

Por exemplo, se uma ferramenta precisa apenas publicar uma imagem em um determinado contexto, é preferível não conceder acesso desnecessário a outros recursos.

---

# 14. Imagens públicas e privadas

Os pacotes do GitHub podem ter diferentes configurações de visibilidade, dependendo do contexto.

Uma imagem pública pode ser baixada por outras pessoas sem a mesma necessidade de autenticação de um pacote privado.

Uma imagem privada exige autorização para ser acessada.

Exemplo conceitual:

```text
Imagem pública
      │
      └── docker pull
            ↓
       acesso público

Imagem privada
      │
      └── docker pull
            ↓
       autenticação
            ↓
       acesso autorizado
```

Isso é especialmente importante para aplicações que não devem ter suas imagens disponíveis publicamente.

---

# 15. GitHub Actions + GHCR

Uma das utilizações mais importantes do GHCR é sua integração com **GitHub Actions**.

É possível criar um workflow que automaticamente:

1. detecta uma alteração no código;
2. constrói a imagem Docker;
3. autentica no GHCR;
4. publica a imagem;
5. disponibiliza a imagem para deployment.

Fluxo:

```text
git push
   │
   ▼
GitHub Repository
   │
   ▼
GitHub Actions
   │
   ├── docker build
   │
   ├── docker login
   │
   └── docker push
   │
   ▼
GHCR
   │
   ▼
Imagem disponível
```

Isso é muito utilizado em processos de **CI/CD**.

---

# 16. GHCR em um projeto Django

Considere um projeto Django que utiliza Docker:

```text
Projeto Django
│
├── código-fonte
├── Dockerfile
├── requirements.txt
└── docker-compose.yml
```

O projeto pode ser armazenado no GitHub:

```text
GitHub Repository
       │
       └── projeto Django
```

O Dockerfile pode ser utilizado para construir uma imagem:

```bash
docker build -t ghcr.io/usuario/projeto-django:latest .
```

Depois:

```bash
docker push ghcr.io/usuario/projeto-django:latest
```

A imagem ficará disponível no GHCR.

Um servidor poderá posteriormente fazer:

```bash
docker pull ghcr.io/usuario/projeto-django:latest
```

e executar a aplicação.

---

# 17. GHCR e Docker Compose

O GHCR também pode ser utilizado com Docker Compose.

Em vez de sempre construir uma imagem localmente:

```yaml
services:
  web:
    build: .
```

pode-se utilizar uma imagem já publicada:

```yaml
services:
  web:
    image: ghcr.io/usuario/meu-projeto:latest
```

Nesse caso, o Docker poderá baixar a imagem do GHCR.

Isso é útil quando o processo de construção da imagem ocorre em outro ambiente, como uma pipeline de CI/CD.

---

# 18. GHCR e deployment

Um fluxo comum de deployment é:

```text
DESENVOLVEDOR
     │
     │ git push
     ▼
GITHUB
     │
     ▼
GITHUB ACTIONS
     │
     │ docker build
     ▼
IMAGEM
     │
     │ docker push
     ▼
GHCR
     │
     │ docker pull
     ▼
SERVIDOR
     │
     ▼
CONTAINER
     │
     ▼
APLICAÇÃO
```

A vantagem é separar o processo de desenvolvimento do processo de execução.

O servidor não precisa necessariamente construir a imagem do zero. Ele pode utilizar uma imagem previamente construída e publicada.

---

# 19. GHCR não é um container

Uma confusão comum é pensar que o GHCR executa os containers.

Isso está incorreto.

O GHCR é um **registro**.

```text
GHCR
│
└── armazena/distribui imagens
```

Quem executa o container é um ambiente que possui um runtime de containers, como o Docker.

Por exemplo:

```text
GHCR
   │
   │ docker pull
   ▼
Docker
   │
   │ docker run
   ▼
Container em execução
```

Portanto:

> **GHCR armazena a imagem; Docker (ou outro runtime compatível) executa o container.**

---

# 20. GHCR e Dockerfile também são coisas diferentes

Outra distinção importante:

### Dockerfile

É um arquivo de instruções para construir uma imagem.

Exemplo:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### Imagem

É o resultado da construção do Dockerfile:

```text
Dockerfile
    │
    │ docker build
    ▼
Imagem
```

### GHCR

É onde essa imagem pode ser armazenada:

```text
Imagem
    │
    │ docker push
    ▼
GHCR
```

### Container

É uma instância em execução baseada em uma imagem:

```text
Imagem
    │
    │ docker run
    ▼
Container
```

---

# 21. Resumo das principais diferenças

| Conceito | Função |
|---|---|
| GitHub Repository | Armazena código-fonte e arquivos do projeto |
| Dockerfile | Define como construir uma imagem |
| Docker Image | Pacote utilizado para criar containers |
| GHCR | Armazena/distribui imagens de containers |
| GitHub Packages | Serviço/ecossistema para armazenar e distribuir pacotes |
| Docker Container | Executa uma aplicação baseada em uma imagem |
| PAT | Credencial para autenticar determinadas operações |
| GitHub Actions | Automatiza workflows, incluindo build e publicação de imagens |

---

# 22. Fluxo para memorizar

Uma forma simples de memorizar todo o processo é:

```text
        DESENVOLVIMENTO
              │
              ▼
      GitHub Repository
              │
         Dockerfile
              │
              ▼
        docker build
              │
              ▼
        Docker Image
              │
        docker push
              │
              ▼
            GHCR
              │
        docker pull
              │
              ▼
       Outro computador
              │
        docker run
              │
              ▼
          Container
```

Enquanto o GitHub Packages pode ser representado de forma mais ampla:

```text
                 GitHub Packages
                       │
       ┌───────────────┼────────────────┐
       │               │                │
      npm            Maven            GHCR
       │               │                │
   JavaScript         Java        Containers
```

## 23. Em uma frase

> **GitHub Packages é o serviço do GitHub para hospedar e distribuir pacotes; GHCR é o registro de containers desse serviço, utilizado principalmente para armazenar e distribuir imagens Docker/OCI.**

## 24. Conceitos-chave para estudar

Se você estiver estudando Docker, GitHub e CI/CD, é importante saber diferenciar:

- GitHub Repository;
- GitHub Packages;
- GHCR;
- Dockerfile;
- Docker Image;
- Docker Container;
- Docker Hub;
- PAT;
- `docker build`;
- `docker tag`;
- `docker push`;
- `docker pull`;
- GitHub Actions;
- CI/CD;
- imagens públicas e privadas;
- tags de imagens.

A sequência mais importante é:

```text
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker tag
    ↓
docker push
    ↓
GHCR
    ↓
docker pull
    ↓
Docker Container
```

# Comunidade Católica Mar a Dentro (API)

Caso ainda não tenha a CLI do Nest instalada, instale-a com o comando:

```bash
npm i -g @nestjs/cli
```

## Criação do projeto NestJS

Acesse o diretório onde deseja salvar o projeto e, usando o CLI do NestJS, execute o comando:

```bash
nest new api
```

O CLI solicitará que você escolha um gerenciador de pacotes para seu projeto — escolha o gerenciador de pacotes que mais lhe agradar (pnpm). Depois disso, você deve ter um novo projeto NestJS no diretório atual.

Abrindo o projeto no vscode, tem-se a seguinte estrutura:

```
core
  ├── node_modules
  ├── src
  │   ├── app.controller.spec.ts
  │   ├── app.controller.ts
  │   ├── app.module.ts
  │   ├── app.service.ts
  │   └── main.ts
  ├── test
  │   ├── app.e2e-spec.ts
  │   └── jest-e2e.json
  ├── .gitignore
  ├── .prettierrc
  ├── eslint.config.msj
  ├── nest-cli.json
  ├── package.json
  ├── pnpm-lock.yaml
  ├── README.md
  ├── tsconfig.build.json
  └── tsconfig.json
```

Crie o arquivo `.env.example` na raiz do projeto, já pensando nas configurações mais a frente:

```
# Application
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL="postgresql://USER:PASSWORD@localhost:5432/paroquia_db"

# JWT
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=15m
JWT_REFRESH_SECRET=your_refresh_secret_here
JWT_REFRESH_EXPIRES_IN=7d
```

Adicione o _EditorConfig_ ao projeto clicando com o botão direto do mouse no vscode e selecionando _Generate .editorconfig_. e o altere da seguinte forma:

```
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

Ajuste também o arquivo de configuração do Prettier (_.prettierrc_):

```
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all"
}
```

_Talvez seja necessário reiniciar o vscode para que o Prettier começe a validar o código._

Inicie o projeto com o comando abaixo para verificar se tudo está ok até o momento:

```bash
pnpm start:dev
```

**💡 GIT - HORA DO COMMIT**

```bash
git add .
```

```bash
git commit -m "chore: initial project setup with NestJS

- Initialize NestJS project with pnpm
- Configure .gitignore with Node, Prisma and Docker rules
- Add .env.example with required environment variables
- Configure .editorconfig for consistent coding style
- Configure .prettierrc aligned with editorconfig rules"
```

```bash
 git push -u origin main
```

## Banco de Dados / Docker

Para o projeto, será utilizado o PostgreSQL através de um container Docker

_Observação : caso opte em não usar o Docker, configure uma instância do PostgreSQL localmente ou obtenha um banco de dados PostgreSQL hospedado online._

<details>
  <summary>👉 <b>Instalação do Docker (se necessário)</b></summary>
  <p>   </p>

1 - Remover versões antigas (se houver)

```bash
sudo apt remove docker docker-engine docker.io containerd runc 2>/dev/null
```

2 - Instalar dependências

```bash
sudo apt update && sudo apt install -y \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```

3 - Adicionar a chave GPG oficial do Docker

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

4 - Adicionar o repositório oficial

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

5 - Instalar o Docker

```bash
sudo apt update && sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-compose-plugin
```

6 - Permitir rodar Docker sem sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

7 - Iniciar o serviço

```bash
sudo service docker start
```

8 - Verificar instalação

```bash
docker --version
docker compose version
```

Reinicie o terminal, tanto do WSL quando do VSCode

No WSL o Docker não inicia automaticamente com o sistema. Se fechar o terminal e abrir de novo, pode precisar rodar sudo service docker start novamente.

  <details>
    <p>   </p>
    <summary>👉 Para resolver isso... <b>Alias — Docker autostart no WSL</b></summary>

No terminal do WSL, abra o arquivo de configuração do seu shell:

```bash
nano ~/.bashrc
```

Adicione no final do arquivo:

```bash
# Auto-start Docker service
if ! service docker status > /dev/null 2>&1; then
  sudo service docker start > /dev/null 2>&1
fi
```

Salve e aplique:

```bash
source ~/.bashrc
```

Liberar o docker start sem senha (sudo)

```bash
sudo visudo
```

Adicione essa linha no final:

```bash
seu_usuario ALL=(ALL) NOPASSWD: /usr/sbin/service docker start, /usr/sbin/service docker restart
```

💡 Substitua `seu_usuario` pelo seu usuário real.

Agora toda vez que abrir o WSL o Docker já vai estar disponível automaticamente, sem pedir senha.

</details>

</details>

<p>   </p>

Crie um arquivo docker-compose.yml na raiz do projeto com o comando:

```bash
touch docker-compose.yml
```

Configure-o da seguinte forma:

```yml
services:
  postgres:
    image: postgres:16-alpine
    container_name: maradentro_db
    restart: unless-stopped
    ports:
      - '5432:5432'
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - maradentro_network

volumes:
  postgres_data:

networks:
  maradentro_network:
    driver: bridge
```

_💡 Repare que as credenciais vêm de variáveis de ambiente — nunca hardcoded no compose!_

Atualize o arquivo `.env.example` adicionando as novas variáveis:

```bash
# Database
POSTGRES_USER=maradentro_user
POSTGRES_PASSWORD=maradentro_password
POSTGRES_DB=maradentro_db
DATABASE_URL="postgresql://maradentro_user:maradentro_password@localhost:5432/maradentro_db?schema=public"
```

Crie o arquivo `.env` copiando o "exemplo":

```bash
cp .env.example .env
```

Preencha o arquivo `.env` com os valores reais do projeto.

Suba o container Docker do PostgreSQL:

```bash
docker compose up -d
```

Verifique se subiu corretamente:

```bash
docker compose ps
```

Deve aparecer o container `maradentro_db` com status running.

O comando `docker compose up -d` irá executar o contêiner indefinidamente em segundo plano. Para pará-lo, basta executar o comando `docker compose down`.

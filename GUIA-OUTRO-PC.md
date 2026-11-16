# Rodar o EstuLanches em outro PC Windows

Siga as etapas nesta ordem. Os comandos são para PowerShell.

## 1. Publicar os arquivos atualizados

Antes de ir ao outro PC, publique no GitHub este guia e o README atualizado, se ainda não estiverem lá. No computador onde está o projeto, rode:

```powershell
Set-Location "$HOME\Desktop\EstuLanches"
git add .env.example README.md GUIA-OUTRO-PC.md
git commit -m "Adiciona guia de instalação para outro PC"
git push
```

O `.env.example` contém valores fictícios de desenvolvimento que devem coincidir entre `POSTGRES_PASSWORD` e `DATABASE_URL`; eles não são senha pessoal. Não coloque sua senha real no `.env.example` nem envie o arquivo `.env` ao GitHub.

## 2. Instalar WSL 2

No outro PC, abra o **PowerShell como Administrador** e rode:

```powershell
wsl --install
```

Reinicie o PC se o Windows solicitar. Depois, abra PowerShell e rode:

```powershell
wsl --update
wsl --version
```

O Docker Desktop usa WSL 2 como backend recomendado no Windows. Consulte o [guia oficial do WSL](https://learn.microsoft.com/windows/wsl/install) se algum comando falhar.

## 3. Instalar Docker Desktop

Baixe e instale o [Docker Desktop para Windows](https://docs.docker.com/desktop/setup/install/windows-install/). Durante a instalação, use o backend **WSL 2** quando essa opção aparecer. Reinicie se o instalador pedir e abra o Docker Desktop pelo menu Iniciar. Aguarde ele informar que está pronto.

O Docker Desktop inclui Docker Engine, CLI e Compose; não instale PostgreSQL separadamente.

## 4. Instalar Git e Node.js 20

1. Baixe e instale o [Git para Windows](https://git-scm.com/install/win). As opções padrão do instalador servem para este projeto.
2. Baixe o instalador **Windows x64 (.msi)** da linha 20 no [arquivo oficial do Node.js 20](https://nodejs.org/en/download/archive/v20/). Instale e abra um novo PowerShell.

O projeto pede Node.js 20 para rodar testes e gerar a RDM localmente. Essa linha está fora de manutenção, mas foi mantida por ser requisito da atividade.

## 5. Conferir as instalações

Com o Docker Desktop aberto, rode:

```powershell
git --version
node --version
npm --version
docker --version
docker compose version
```

`node --version` deve começar com `v20.`. `docker compose version` deve exibir Compose v2. Se um comando não for reconhecido, feche e reabra o PowerShell depois da instalação.

## 6. Baixar o projeto do GitHub

```powershell
Set-Location "$HOME\Desktop"
git clone https://github.com/PingoGB/EstuLanches.git
Set-Location "$HOME\Desktop\EstuLanches"
```

Se o repositório estiver privado, entre no GitHub quando o Git solicitar autenticação.

## 7. Criar o ambiente local e instalar dependências

```powershell
Copy-Item .env.example .env
npm ci
```

Não é necessário editar `.env` para a demonstração. O `.env` é privado deste computador e está ignorado pelo Git. `npm ci` instala as dependências usando o `package-lock.json` versionado.

## 8. Construir e iniciar a aplicação

Confirme que o Docker Desktop continua aberto. Na raiz `EstuLanches`, rode:

```powershell
docker compose up -d --build
docker compose ps
```

Na primeira execução, o Docker baixa imagens e constrói os containers, então pode levar alguns minutos. Aguarde `app` e `db` ficarem `healthy`. O PostgreSQL 16 roda no container `db`; a porta do banco não fica publicada no Windows.

## 9. Abrir e conferir o sistema

Abra no navegador:

- Sistema: [http://localhost:3000](http://localhost:3000)
- Saúde: [http://localhost:3000/health](http://localhost:3000/health)

A saúde deve informar `status: ok` e `database: connected`. A interface permite demonstrar cardápio, criação de pedidos e acompanhamento de status.

## 10. Rodar testes e gerar a ficha PDF

Para rodar os testes separadamente:

```powershell
npm run test:unit
docker compose --profile test run --build --rm tests npm run test:integration
```

Para executar testes, validação do ambiente e geração do PDF em sequência, personalize primeiro o campo `nome` em `rdm/config-rdm.json` e rode:

```powershell
npm run rdm
```

O PDF fica em `rdm/RDM-EstuLanches.pdf`. Os resultados ficam em `rdm/testes-unitarios.json`, `rdm/testes-integracao.json` e `rdm/resultados.json`.

## 11. Encerrar quando terminar

Para parar containers e manter os dados do banco:

```powershell
docker compose down
```

Na próxima vez, abra o Docker Desktop, entre na pasta do projeto e execute `docker compose up -d`. **Não use** `docker compose down -v` se quiser preservar os dados.

## Se algo falhar

- Docker não responde: abra o Docker Desktop, aguarde iniciar e confira com `docker info`.
- O banco não fica saudável: rode `docker compose logs db`.
- A aplicação não fica saudável: rode `docker compose logs app`.
- Porta 3000 ocupada: feche o outro programa que a usa antes de iniciar o EstuLanches.
- `npm ci` falha: confira `node --version`, use Node.js 20 e verifique a conexão com a internet.

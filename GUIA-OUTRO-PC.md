# Instalação do zero em outro PC Windows

Siga esta ordem no computador que ainda não tem as ferramentas instaladas.

## 1. Instale o WSL 2

Abra PowerShell como Administrador:

```powershell
wsl --install
```

Reinicie o Windows se solicitado. Depois, em PowerShell normal:

```powershell
wsl --update
wsl --version
```

## 2. Instale o Docker Desktop

Baixe o [Docker Desktop para Windows](https://docs.docker.com/desktop/setup/install/windows-install/). Use o backend WSL 2 durante a instalação, abra o aplicativo pelo menu Iniciar e aguarde o estado pronto. O Docker Desktop já inclui Docker Engine e Docker Compose. PostgreSQL não precisa ser instalado no Windows.

## 3. Instale Git e Node.js

1. Instale o [Git para Windows](https://git-scm.com/install/win) com as opções padrão.
2. Instale o Windows Installer x64 do Node.js 24 LTS pelo [arquivo oficial do Node.js](https://nodejs.org/en/download/archive/v24/).
(só clicar no botão verde de download no topo da tela)
3. Abra um novo PowerShell.

Node.js 24 LTS é a versão usada pelo projeto para executar testes e gerar o PDF. O Docker também usa Node.js 24 dentro do container.

Confira as instalações:

```powershell
git --version
node --version
npm --version
docker --version
docker compose version
```

`node --version` deve começar com `v24.`.

## 4. Clone o repositório

No GitHub, abra o repositório do Everton, clique em **Code**, copie o endereço HTTPS e use-o no lugar de `URL_HTTPS_DO_REPOSITORIO`:

```powershell
Set-Location "$HOME\Desktop"
git clone URL_HTTPS_DO_REPOSITORIO EstuLanches-Everton
Set-Location "$HOME\Desktop\EstuLanches-Everton"
```

## 5. Prepare o ambiente

```powershell
Copy-Item .env.example .env
npm ci
```

O `.env.example` contém valores de desenvolvimento que já combinam entre si. O `.env` criado é local e não deve ser publicado.

## 6. Inicie e abra o sistema

Confirme que Docker Desktop está aberto e execute para rodar:

```powershell
docker compose up -d --build
docker compose ps
```

Espere `app` e `db` ficarem `healthy`. Na primeira subida, o banco, as tabelas e os produtos de exemplo são criados automaticamente.

- Vitrine: [http://localhost:3000](http://localhost:3000)
- Saúde: [http://localhost:3000/health](http://localhost:3000/health)

## 7. Teste e gere a ficha

Rode cada teste separadamente:

```powershell
npm run test:unit
docker compose --profile test run --build --rm tests npm run test:integration
```

Para rodar testes, validação, auditoria de logs e geração do PDF numa sequência só:

```powershell
npm run rdm
```

O PDF ficará em `rdm/RDM-EstuLanches-Everton.pdf`; os resultados JSON ficarão na pasta `rdm`.

## 8. Parar o sistema

```powershell
docker compose down
```


## Problemas comuns

- Docker não responde: abra Docker Desktop e confira `docker info`.
- Algum serviço não fica saudável: consulte `docker compose logs db` ou `docker compose logs app`.
- Porta 3000 ocupada: feche o outro aplicativo que está usando essa porta.
- `npm ci` reclama de versão: confirme `node --version` e instale Node.js 24 LTS.

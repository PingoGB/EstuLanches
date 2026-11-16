# EstuLanches

Sistema de demonstração para gerenciar cardápio e pedidos de uma cantina escolar. O navegador acessa uma interface HTML/CSS/JavaScript servida pelo Express; a API Express grava os dados em PostgreSQL 16.

Para configurar uma máquina Windows do zero, siga [GUIA-OUTRO-PC.md](GUIA-OUTRO-PC.md).

Para gravar a apresentação da atividade, use o [roteiro cronometrado do vídeo](docs/ROTEIRO-VIDEO.md).

## 1. Instalar ferramentas

- **Docker Desktop (Windows/macOS/Linux)** instala Docker Engine, CLI e Compose: [download oficial](https://www.docker.com/products/docker-desktop/) e [instruções oficiais](https://docs.docker.com/compose/install/). No Windows, iniciar Docker Desktop e usar o backend WSL 2 quando disponível. Em Linux, também é possível instalar Docker Engine e o plugin Compose pelos [guias oficiais](https://docs.docker.com/engine/install/).
- **Node.js 20 LTS** e npm: [downloads oficiais do Node.js](https://nodejs.org/en/download/). Selecione a linha 20 LTS para cumprir a versão do projeto.
- **Git**: [instaladores oficiais](https://git-scm.com/downloads).

Confirme as ferramentas no PowerShell/Terminal: `docker --version`, `docker compose version`, `node --version`, `npm --version`, `git --version`. Docker Desktop deve estar aberto e `docker info` deve terminar sem erro.

## 2. Preparar a pasta e o ambiente

Descompacte/clone este projeto e entre na pasta `EstuLanches`. No Windows PowerShell:

```powershell
Set-Location "$HOME\Desktop\EstuLanches"
Copy-Item .env.example .env
```

No macOS/Linux:

```sh
cd ~/Desktop/EstuLanches
cp .env.example .env
```

O `.env.example` possui credenciais exclusivamente de desenvolvimento. O `.env` local já está no `.gitignore`.

## 3. Gerar lockfile e iniciar

Na raiz, execute `npm install` uma vez para criar `package-lock.json` (o Dockerfile usa `npm ci` e exige esse arquivo). Depois suba os serviços:

```sh
npm install
docker compose up -d --build
```

O serviço `db` cria tabelas e produtos de exemplo na primeira subida. O Compose só inicia a aplicação depois que o healthcheck do banco fica saudável. Os dados ficam no volume `postgres_data`.

## 4. Acessar

- Sistema: [http://localhost:3000](http://localhost:3000)
- Saúde da API: [http://localhost:3000/health](http://localhost:3000/health)
- Produtos: `GET http://localhost:3000/api/products`
- Pedidos: `GET http://localhost:3000/api/orders`

Na interface, use as abas **Cardápio**, **Novo pedido** e **Pedidos** para demonstrar cadastro, edição, exclusão, cálculo de total e atualização de status.

## 5. Testes e relatórios

Testes unitários isolados: `npm run test:unit`.

Testes de integração com banco real do Compose (banco em execução):

```sh
docker compose --profile test run --build --rm tests npm run test:integration
```

Os comandos salvam JSON em `rdm/testes-unitarios.json` e `rdm/testes-integracao.json`. `npm test` combina ambos e executa a integração pelo runner Docker. Cobertura local: `npm run test:coverage`. O alvo de testes do Dockerfile contém dependências de desenvolvimento; a imagem de produção instala apenas dependências runtime.

## 6. Validar implantação

Com os serviços ativos:

```sh
docker compose ps
docker compose logs
docker compose exec db psql -U estu_app -d estulanches -c "\dt"
docker compose exec db psql -U estu_app -d estulanches -c "SELECT id,name,category,price,available FROM products ORDER BY id;"
curl http://localhost:3000/health
curl http://localhost:3000/api/products
curl http://localhost:3000/api/orders
npm run validar
```

No PowerShell, use `Invoke-RestMethod http://localhost:3000/health` no lugar de curl. Em `docker compose ps`, aguarde `healthy` nos dois serviços; só a aplicação publica a porta 3000, o banco não publica porta ao host. A saúde deve retornar `{"status":"ok","database":"connected"}`. Nos logs, procure `database system is ready to accept connections` no Postgres e `EstuLanches iniciado` na aplicação. O script procura `fatal`, `panic`, exceções não tratadas, `ECONNREFUSED` e alguns erros críticos comuns.

Saída esperada de `docker compose up -d --build`: o Compose constrói a imagem da aplicação e termina com mensagens `Started` para `db` e `app` (a primeira compilação e o primeiro seed demoram mais). `docker compose ps` deve mostrar os dois serviços `Up` e `(healthy)`; a coluna `PORTS` do app mostra `0.0.0.0:3000->3000/tcp`, enquanto a linha do `db` não mostra porta publicada. Os nomes de containers recebem o nome do projeto escolhido pelo Compose.

Em `docker compose logs`, o Postgres deve informar que está pronto para aceitar conexões e o app deve registrar JSON com a mensagem `EstuLanches iniciado`. Mensagens de erro de conexão do app durante a subida, `FATAL` do Postgres, `panic` ou reinicializações contínuas indicam falha; rode `docker compose logs db` e `docker compose logs app` para inspecionar o serviço específico.

`docker compose exec db psql -U estu_app -d estulanches -c "\dt"` deve listar `order_items`, `orders` e `products`. A consulta de produtos deve listar seis itens de exemplo. Os endpoints de lista retornam arrays JSON; pedidos vazios retornam `[]`. `/health` responde HTTP 200 com `{"status":"ok","database":"connected"}` quando banco e API estão saudáveis.

Resumo e captura real de `compose ps`, erros críticos, resposta de saúde, versão e horário são gravados em `rdm/resultados.json`. Alternativa PowerShell: `.scripts\validar-ambiente.ps1`. Em macOS/Linux: `sh scripts/validar-ambiente.sh`.

## 7. Gerar a Ficha RDM em PDF

Rode `npm run rdm`. Ele executa testes unitários, integração no PostgreSQL, validação e geração do PDF. A ficha de uma página fica em `rdm/RDM-EstuLanches.pdf`. Para trocar somente seu nome, abra `rdm/config-rdm.json` e altere `"nome": "Aluno(a)"` para seu nome (exemplo: `"nome": "Aluno(a)"` → `"nome": "Maria Silva"`). Cargo e ambiente já estão preenchidos. Se Docker não estiver ativo, a ficha indica falha/ressalva.

## 8. Publicar no GitHub

Crie um repositório vazio no GitHub e use a URL que o site fornecer no comando `git remote add origin`:

```sh
git init
git add .
git status
git commit -m "Entrega inicial do EstuLanches"
git branch -M main
git remote add origin URL_DO_REPOSITORIO_CRIADO
git push -u origin main
```

Antes do commit, confira que `.env` não aparece em `git status`. O `.gitignore` o exclui; nunca publique credenciais. Inclua Dockerfile, `docker-compose.yml` e `rdm/RDM-EstuLanches.pdf`.

## 9. Problemas comuns

- **Porta 3000 ocupada:** altere `PORT` no `.env` para `3001`, recrie com `docker compose up -d --build` e acesse a nova porta.
- **Docker indisponível/permissão:** abra Docker Desktop, aguarde pronto e tente `docker info`. No Linux, configure acesso do usuário ao Docker conforme a documentação da distribuição.
- **Banco não saudável:** veja `docker compose logs db`, confirme `.env` e aguarde. Se um volume antigo tiver credenciais diferentes, `docker compose down -v` reinicializa e apaga os dados persistidos.
- **`npm ci` sem lockfile:** rode `npm install` na raiz e tente novamente o build.
- **Script `.sh` no Windows:** use `powershell -ExecutionPolicy Bypass -File .\scripts\validar-ambiente.ps1`.
- **Integração não conecta:** suba primeiro com `docker compose up -d db` e rode o runner Compose da seção 5.
- **RDM informa JSON ausente:** execute `npm run test:unit`, o comando de integração Compose e `npm run validar`; depois `node scripts/gerar-rdm.js`.

## 10. Checklist de entrega

- [ ] `docker compose ps` mostra aplicação e banco saudáveis.
- [ ] `/health` informa banco conectado.
- [ ] Cardápio permite cadastrar, editar e excluir produtos.
- [ ] Pedido calcula total pelo servidor e rejeita item indisponível.
- [ ] Fluxo de status foi demonstrado.
- [ ] JSONs de teste e ambiente estão em `rdm/`.
- [ ] PDF RDM foi gerado e revisado.
- [ ] `.env` não foi adicionado ao Git.

### Sequência resumida, do zero ao PDF

1. `cd ~/Desktop/EstuLanches` (PowerShell: `Set-Location "$HOME\Desktop\EstuLanches"`)
2. `cp .env.example .env` (PowerShell: `Copy-Item .env.example .env`)
3. `npm install`
4. `docker compose up -d --build`
5. `docker compose ps`
6. `npm run rdm`
7. Abra `rdm/RDM-EstuLanches.pdf`
PARA RODAR:docker compose start app
docker compose ps
PARA PARAR DE RODAR:docker compose stop app

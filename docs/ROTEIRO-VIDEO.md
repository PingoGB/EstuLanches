# Roteiro do vídeo — EstuLanches

**Duração alvo:** 2min40s a 3min.  
**Formato:** gravação da tela com narração em voz off.  
**Preparação:** abra o Docker Desktop; inicie o projeto com `docker compose up -d --build`; deixe o navegador em `http://localhost:3000` e o VS Code aberto na raiz do projeto. Não mostre o conteúdo do `.env`.

## 0:00–0:18 — Apresentação

**Na tela:** página inicial do EstuLanches no navegador.

**Fale:**

> Olá, eu sou Arthur e este é o EstuLanches, um sistema de cardápio e pedidos para uma cantina escolar. Ele resolve a organização dos produtos e o acompanhamento dos pedidos em um só lugar. A aplicação usa Node.js, Express e PostgreSQL, com execução conteinerizada em Docker.

## 0:18–0:39 — Arquitetura e arquivos Docker

**Na tela:** mude para o VS Code. Mostre rapidamente a árvore `src` em camadas; abra `Dockerfile` e depois `docker-compose.yml`.

**Fale:**

> O código está separado em rotas, controllers, serviços e repositórios. No Dockerfile, uso Node 20 Alpine, uma etapa de dependências e uma imagem final com apenas pacotes de produção. A aplicação roda como usuário não root e tem healthcheck. No Compose, a aplicação e o PostgreSQL 16 são serviços separados; o banco usa volume persistente e uma rede interna, sem porta exposta no computador.

## 0:39–0:58 — Implantação e saúde

**Na tela:** abra o terminal do VS Code e rode:

```powershell
docker compose ps
docker compose logs --tail=20
Invoke-RestMethod http://localhost:3000/health
```

Mostre `healthy` para `app` e `db`, a mensagem de inicialização da aplicação e a resposta de saúde.

**Fale:**

> Aqui verifico o estado dos serviços e os logs de inicialização. O banco deve indicar que está pronto para aceitar conexões, e a aplicação deve registrar que iniciou. A rota de saúde confirma a conexão com o banco; nesta execução ela retornou status ok e database connected.

## 0:58–1:17 — Testes

**Na tela:** rode o comando abaixo e mantenha visíveis os resumos de testes e da RDM:

```powershell
npm run rdm
```

O comando executa testes unitários, integração com PostgreSQL real, valida o ambiente e gera o PDF. Os resultados registrados nesta execução foram 8 de 8 testes unitários e 3 de 3 de integração aprovados. Se aparecer a mensagem informativa `Force exiting Jest`, mostre que o terminal voltou ao prompt e que os testes passaram.

**Fale:**

> Os testes unitários cobrem validações e cálculo do total. Os testes de integração usam Supertest contra a API e PostgreSQL real: criam produto e pedido, mudam status, consultam, cancelam e verificam respostas de erro. Os relatórios JSON são salvos na pasta RDM.

## 1:17–1:48 — Demonstração do sistema

**Na tela:** volte ao navegador em `localhost:3000`.

1. Na aba **Cardápio**, clique em **Novo produto**. Cadastre `Produto demonstração`, categoria `Lanches`, preço `5,00`; salve.
2. Edite esse produto e altere a descrição; salve novamente.
3. Exclua `Produto demonstração` antes de usá-lo em qualquer pedido.
4. Abra **Novo pedido**, informe `Aluno demonstração`, selecione `Pão de queijo` e quantidade `2`. Mostre o total calculado e registre.
5. Abra **Pedidos** e altere o status para `EM PREPARO` e depois `PRONTO`.

**Fale:**

> A interface permite cadastrar, editar e excluir produtos. No pedido, escolho o aluno e as quantidades; o valor exibido é calculado novamente no servidor, usando os preços do banco. Depois acompanho o pedido e atualizo o status conforme o preparo avança.

## 1:48–2:06 — Segurança

**Na tela:** mostre `src/app.js`, `src/services/validation.js` e, por alguns segundos, `docs/SEGURANCA.md`.

**Fale:**

> A API usa Helmet, política de conteúdo, CORS configurado e limite de requisições. Zod valida os dados de entrada, e os repositórios usam consultas parametrizadas. Erros são tratados centralmente sem expor stack trace ao usuário em produção. A interface escapa os textos recebidos para reduzir risco de XSS, e as credenciais ficam em variáveis de ambiente.

## 2:06–2:23 — Ficha RDM

**Na tela:** abra `rdm/RDM-EstuLanches.pdf` e deixe o status final visível. Mostre também, rapidamente, os arquivos JSON da pasta `rdm`.

**Fale:**

> Esta é a Ficha de Registro de Mudança. Ela reúne versão, ambiente, parâmetros de implantação, data e horário, resultados dos testes e o status final. Nesta execução, o resultado foi sucesso. O PDF e os relatórios são gerados pelos scripts do projeto.

## 2:23–2:48 — Escalabilidade e encerramento

**Na tela:** abra `docs/ESCALABILIDADE.md` e mostre o diagrama Mermaid.

**Fale:**

> Para escalar conceitualmente a um milhão de usuários e cem mil simultâneos, eu colocaria réplicas stateless da API atrás de balanceadores, com orquestração e autoscaling. CDN atenderia arquivos estáticos; Redis aceleraria leituras; filas desacoplariam pedidos. O banco poderia usar pool de conexões, réplicas de leitura e particionamento. Observabilidade e implantação multi-região completariam a arquitetura. Isso é uma proposta conceitual, não está implementada nesta demonstração. Obrigado.

## Checklist rápido de gravação

- [ ] O vídeo tem entre 2 e 5 minutos; fale em ritmo calmo para ficar perto de 2min40s.
- [ ] Os serviços estão saudáveis antes de abrir o navegador.
- [ ] Rode `npm run rdm` durante a gravação para mostrar resultados reais; aguarde voltar ao prompt.
- [ ] A demonstração usa um produto temporário e o exclui antes de criar o pedido.
- [ ] O `.env` e qualquer credencial pessoal não aparecem na tela.
- [ ] O PDF mostra o nome escolhido e o status final.
- [ ] O trecho de escalabilidade é apresentado como proposta conceitual.

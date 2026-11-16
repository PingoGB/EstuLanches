# Arquitetura

O EstuLanches usa Express em Node.js 20 e PostgreSQL 16. A aplicação organiza HTTP em **rotas**, **controllers**, **services** e **repositories**: rotas ligam endpoints, controllers validam e formam respostas, services aplicam regras de pedido e repositories concentram SQL parametrizado.

O navegador recebe HTML, CSS e JavaScript puro pelo Express. A API JSON fica sob `/api`; `/health` confirma a conectividade com o PostgreSQL. A gravação de pedido e itens usa uma transação para evitar registros incompletos.

O Compose mantém banco e aplicação em containers separados. O banco usa volume nomeado e rede interna; apenas a aplicação publica sua porta no host. `init.sql` cria tabelas e inclui produtos iniciais na primeira inicialização do volume.

Jest cobre regras de negócio com doubles simples e rotas reais com Supertest conectado ao PostgreSQL do Compose. O perfil `test` usa estágio de imagem com dependências de desenvolvimento.

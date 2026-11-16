# Segurança aplicada

- **Helmet e CSP:** configura cabeçalhos HTTP defensivos; scripts e estilos próprios são servidos pela mesma origem.
- **CORS:** restringe as chamadas de navegador à origem definida por `CORS_ORIGIN`.
- **Rate limit:** limita requisições a 300 por janela de 15 minutos por endereço.
- **Validação Zod:** valida tipos, comprimentos, campos permitidos, quantidades e preços; strings são aparadas antes de persistir.
- **SQL parametrizado:** todos os valores de entrada passam como parâmetros do `pg`, sem concatenação de SQL.
- **Segredos no ambiente:** credenciais são lidas de `.env`, ignorado pelo Git; use segredos fortes fora do exemplo local.
- **Container e rede:** Node roda como usuário `node`; PostgreSQL não publica porta e só participa da rede interna. A aplicação tem uma segunda rede para o acesso publicado pelo host.
- **Erros e logs:** tratamento centralizado não envia stack trace em produção; Pino escreve logs estruturados e aplica redação a campos sensíveis.
- **Interface:** textos vindos da API são escapados antes de compor HTML e o feedback usa `textContent`.

O `.env.example` contém credenciais públicas de desenvolvimento para facilitar a demonstração local. Elas devem ser trocadas em uma instalação real.

# Relatório

## Resumo

O laboratório Mesa Norte apresenta falha sistêmica de autorização: qualquer usuário autenticado, inclusive um analyst de um tenant, acessa e modifica dados de todos os tenants e executa funções administrativas. O modelo de papéis (`role`) e o isolamento por cliente (`client_name`) são decorativos no servidor. O risco principal é vazamento e alteração cross-tenant de chamados, documentos e PII, incluindo a nota interna de margem negociada. Corrigir primeiro a autorização por objeto e por função nas rotas `/api/*` e remover `role`/`client_name` do `PATCH /api/users/me`. Em seguida, corrigir o fluxo de reset de senha, que devolve o token na resposta e permite account takeover de qualquer conta, inclusive do admin.

| Achado | Classe | Severidade | Onde |
| --- | --- | --- | --- |
| Account takeover via token de reset exposto | Autenticação | Crítica | `POST /api/auth/forgot` |
| Autorização quebrada entre papéis e tenants (BOLA/BFLA/BOPLA) | IDOR / controle de acesso | Crítica | `/api/users`, `/api/users/{id}`, `/api/tickets`, `/api/tickets/{id}`, `/api/documents`, `PATCH /api/users/me` |
| CORS refletindo origem arbitrária com credenciais | Configuração | Crítica | Todas as rotas `/api/*` |
| XSS refletido na busca de chamados | XSS | Alta | `GET /api/tickets/search?q=` (renderizado no front-end) |
| XSS armazenado em comentário de chamado | XSS | Alta | `POST /api/tickets/{id}/comments` |
| Especificação OpenAPI exposta | Configuração | Média | `GET /api/openapi.json` |

## Escopo

Testado o host `https://offensive-seg-challange-api.onrender.com` em ambiente de laboratório, durante a janela do exercício, nos papéis anônimo, usuário autenticado não privilegiado (analyst) e administrador. Ficaram de fora produção, outros hosts, DNS, provedores de e-mail e terceiros, além de negação de serviço e força bruta de login, conforme o escopo do exercício.

## Metodologia

A superfície foi mapeada a partir da página de login, da especificação OpenAPI exposta em `/api/openapi.json` e de enumeração manual de rotas e métodos. Os testes foram conduzidos à mão, com `curl` e navegador, comparando respostas entre os papéis anônimo, analyst e admin, e verificando autorização por objeto, por função e por propriedade, além de CORS, fluxo de reset de senha, XSS refletido e armazenado, e exposição de informação em erros. O caderno completo está nas notas.

## Achados

### Account takeover de qualquer conta, inclusive do administrador

- **Classe:** Autenticação
- **Severidade:** Crítica
- **Onde:** `POST /api/auth/forgot` e `POST /api/auth/reset`
- **Quem:** Anônimo consegue; nenhum papel deveria receber o token de reset na resposta.
- **Impacto:** Qualquer pessoa que saiba o e-mail de um usuário obtém o token de reset diretamente na resposta HTTP e redefine a senha, assumindo a conta. Foi possível assumir a conta do administrador e, a partir dela, acessar chamados e documentos do tenant.
- **Correção:** Nunca retornar o token de reset na resposta. Enviar apenas por e-mail, com expiração curta, uso único e invalidação após a troca. A resposta do `/forgot` deve ser genérica, independente de o e-mail existir.
- **Evidência:** `entregas/evidencias/api-auth-forgot.png` ([link](entregas/evidencias/api-auth-forgot.png)), `entregas/evidencias/api-auth-reset.png` ([link](entregas/evidencias/api-auth-reset.png)) (token redactado).

### Autorização quebrada entre papéis e tenants

- **Classe:** IDOR / controle de acesso
- **Severidade:** Crítica
- **Onde:** `/api/users`, `/api/users/{id}`, `/api/tickets`, `/api/tickets/{id}`, `/api/documents`, `PATCH /api/users/me`
- **Quem:** Usuário autenticado não privilegiado (analyst) consegue; apenas o próprio usuário e o admin deveriam acessar os respectivos recursos.
- **Impacto:** O papel analyst acessa a lista completa de usuários, perfis de outros tenants, chamados e documentos de outros clientes, e executa alterações que deveriam ser restritas ao admin. O `role` e o `client_name` não são validados no servidor, então o isolamento multi-tenant é inexistente. A nota interna de margem negociada de um chamado fica exposta a outros tenants.
- **Correção:** Validar no servidor se o recurso pertence ao tenant do usuário autenticado (autorização por objeto) e se o papel permite a função (autorização por função). No `PATCH /api/users/me`, aceitar apenas `name`, `email` e `title`, nunca `role` ou `client_name`.
- **Evidência:** [idor-ticket.png](entregas/evidencias/idor-ticket.png), [idor-ticket2.png](entregas/evidencias/idor-ticket2.png), [idor-ticket3.png](entregas/evidencias/idor-ticket3.png), [idor-ticket4.png](entregas/evidencias/idor-ticket4.png), [idor-documentos.png](entregas/evidencias/idor-documentos.png) e [idor-documentos2.png](entregas/evidencias/idor-documentos2.png).

### CORS refletindo origem arbitrária com credenciais

- **Classe:** Configuração
- **Severidade:** Crítica
- **Onde:** Todas as rotas `/api/*`
- **Quem:** Qualquer site externo consegue; apenas o próprio domínio da aplicação deveria ser autorizado.
- **Impacto:** O servidor reflete qualquer `Origin` com `Access-Control-Allow-Credentials: true` e métodos `GET, POST, PATCH, DELETE`. Um site externo consegue ler respostas autenticadas e executar escrita cross-origin em nome de um usuário logado, desde que o navegador envie o cookie. A proteção atual depende do `SameSite` do cookie, não do servidor.
- **Correção:** Usar allowlist fixa de origens em vez de refletir o `Origin`. Remover `Allow-Credentials` quando não for necessário e restringir os métodos permitidos. Manter `SameSite` como defesa em profundidade.
- **Evidência:** [CORS-poc.png](entregas/evidencias/CORS-poc.png), [CORS-failed.png](entregas/evidencias/CORS-failed.png).

### XSS refletido na busca

- **Classe:** XSS
- **Severidade:** Alta
- **Onde:** `GET /api/tickets/search?q=`, renderizado no front-end da busca
- **Quem:** Qualquer usuário que abrir o link; o atacante envia a URL maliciosa e o script executa no navegador da vítima.
- **Impacto:** O parâmetro `q` é refletido sem escape no HTML da página de resultados, executando script no navegador de quem clicar no link.
- **Correção:** Escapar a saída no front-end (`textContent` ou escape de HTML) e no servidor, e adicionar `Content-Security-Policy` como defesa em profundidade. Não depender apenas de filtro de entrada.
- **Evidência:** [xss-refletido-campo-de-busca.png](entregas/evidencias/xss-refletido-campo-de-busca.png).

### XSS armazenado em comentário de chamado

- **Classe:** XSS
- **Severidade:** Alta
- **Onde:** `POST /api/tickets/{id}/comments`, renderizado na visualização do chamado
- **Quem:** Usuário autenticado consegue; nenhum usuário deveria executar script no navegador de outro.
- **Impacto:** O servidor aceita e armazena o payload sem sanitizar e o devolve cru na leitura do chamado. Quem visualizar o comentário executa o script no próprio navegador, permitindo ações em nome da vítima e combinação com o CORS para exfiltração.
- **Correção:** Sanitizar a entrada no servidor e escapar na saída, usando `textContent` ou escape de HTML no front-end. Adicionar `Content-Security-Policy` como defesa em profundidade.
- **Evidência:** [xss-armazenado-comentarios-chamados.png](entregas/evidencias/xss-armazenado-comentarios-chamados.png).

### Especificação OpenAPI exposta

- **Classe:** Configuração
- **Severidade:** Média
- **Onde:** `GET /api/openapi.json`
- **Quem:** Anônimo consegue; a especificação não deveria ser pública em produção.
- **Impacto:** A especificação completa da API, com rotas, métodos e parâmetros, permite a um atacante mapear toda a superfície sem enumeração, acelerando a exploração dos demais achados.
- **Correção:** Remover ou proteger a especificação em produção, exigindo autenticação para acessá-la.
- **Evidência:** [ffuf-documentacao-api-exposta.png](entregas/evidencias/ffuf-documentacao-api-exposta.png), [documentacao-api.png](entregas/evidencias/documentacao-api.png).

## Recomendação

As falhas encontradas compartilham uma causa comum: a aplicação trata autenticação como suficiente e não implementa autorização no servidor. Por isso, a correção deve ser tratada em duas frentes executadas no mesmo ciclo. A primeira é a autorização, que concentra os dois achados críticos e é o que expõe dados de todos os tenants. O primeiro passo é centralizar a autorização em um middleware aplicado a todas as rotas `/api/*` que hoje dependem apenas de autenticação, recebendo o recurso solicitado, o tenant do usuário e o papel, e negando por padrão, liberando apenas o que for explicitamente permitido. 

Em seguida, nas rotas `/api/users/{id}`, `/api/tickets/{id}`, `/api/documents/{id}` e `/api/tickets/{id}/comments`, validar no servidor se o recurso pertence ao `client_name` do usuário autenticado antes de retornar ou alterar, de modo que o identificador na URL não seja a única fonte de verdade. A lista completa de usuários e qualquer operação de criação, alteração ou remoção de usuários devem ficar restritas ao papel `admin`, enquanto o papel `analyst` não deve acessar funções administrativas. No `PATCH /api/users/me`, aplicar allowlist explícita de campos, aceitando apenas `name`, `email` e `title` e rejeitando `role`, `client_name`, `id` e qualquer outro campo fora da lista, com o mesmo cuidado em `POST /api/tickets` e `POST /api/tickets/{id}/comments`. O critério de aceite é que um usuário `analyst` do tenant A não consiga ler nem escrever dados do tenant B, não consiga listar todos os usuários e não consiga alterar `role` ou `client_name`, com um teste automatizado cobrindo cada rota com dois papéis e dois tenants. A segunda frente é o fluxo de reset de senha, que deve ser corrigido antes de qualquer exposição pública, porque permite assumir qualquer conta, inclusive a de maior privilégio. O `reset_token` não pode ser retornado na resposta HTTP: deve ser entregue apenas por e-mail, em canal seguro, e a resposta do `/api/auth/forgot` deve ser genérica, independente de o e-mail existir, para não criar enumeração. O token deve ter expiração curta, uso único e invalidação após a troca, e a sessão deve ser regenerada após o reset, com as sessões antigas invalidadas, para que um token roubado não sobreviva à troca de senha. O critério de aceite é que o `/forgot` não retorne token, que o `/reset` rejeite token expirado, reutilizado ou de outro usuário, e que a sessão antiga deixe de funcionar após a troca. As duas frentes devem ser tratadas no mesmo ciclo, porque o account takeover é o vetor que dá acesso inicial e a autorização quebrada é o que transforma esse acesso em comprometimento total: corrigir apenas uma das duas mantém o risco crítico, já que sem a autorização um analyst assume o tenant inteiro e sem o reset qualquer pessoa assume a conta do admin. 

Depois de fechar autorização e autenticação, corrigir as falhas que ampliam o impacto das anteriores. O CORS deve substituir a reflexão de `Origin` por uma allowlist fixa de domínios da aplicação, remover `Access-Control-Allow-Credentials` quando não for necessário e restringir `Access-Control-Allow-Methods` ao mínimo, mantendo `SameSite` no cookie como defesa em profundidade, mas não como única proteção; o CORS deve ser corrigido junto com a primeira frente, porque hoje é o que permite explorar a autorização quebrada a partir de um site externo. Os dois pontos de XSS, na busca e no comentário, devem ser corrigidos escapando a saída no front-end com `textContent` ou escape de HTML e no servidor, adicionando `Content-Security-Policy` restritiva com `default-src 'self'`, sem depender apenas de filtro de entrada. A especificação OpenAPI deve ser removida do ambiente de produção ou protegida com autenticação. 

Ao final de cada frente, repetir os testes deste relatório com os três papéis (anônimo, analyst e admin) e dois tenants, confirmando que o analyst não acessa dados de outro tenant, que o admin não é assumido por reset de senha, que o CORS não reflete origem externa e que os payloads de XSS não executam.

## Limitações

A janela não permitiu avaliar a invalidação de sessão no logout nem as flags do cookie em um login bem-sucedido por meio de navegador, porque o cookie de sessão usa `SameSite` restritivo e o teste cross-origin no navegador não envia credenciais. A leitura e a escrita cross-origin foram confirmadas via cliente HTTP, que não aplica `SameSite`. Não foi possível confirmar execução de código no servidor: a versão de Node reportada está acima da corrigida para a CVE de Brotli e o servidor rejeita `Content-Encoding: br`. Não foram testados negação de serviço, força bruta de login, outros hosts ou DNS, por estarem fora do escopo.
# Notas do pentest

## Ordem do trabalho

1. **Reconhecimento inicial (~0h).** `ping` e `nmap` no host do laboratório; confirmação de Cloudflare e Render na frente do serviço; leitura do `/login` e do `app.js`.
2. **Enumeração de rotas (~0h30).** `ffuf`/`gobuster` com wordlist de diretórios; descoberta de `/documentos`, `/perfil`, `/busca`, `/chamados` (todas 302), `/login` e `/recuperar-senha` (200).
3. **API aberta (~1h).** Descoberta de `GET /api/v1/users` sem autenticação, retornando PII de três usuários, incluindo o admin (Camila, Mesa Norte).
4. **CORS (~1h30).** `OPTIONS` com `Origin: https://evil.example` refletido com `Allow-Credentials: true` e métodos de escrita; confirmação de leitura e escrita cross-origin via cliente HTTP com a sessão da Camila.
5. **Login e injeção (~2h).** Testes de SQLi no `/login` com aspa simples e payloads de bypass; resposta sempre `302 35`, indicando consulta parametrizada. Tentativa de `sqlmap` interrompida pelo WAF do Cloudflare.
6. **XSS (~2h30).** Testes de reflexão no `erro`, `email`, `next`; respostas de tamanho fixo (990 bytes), sem reflexo. Descarte.
7. **Reset de senha (~3h).** Descoberta de `POST /api/auth/forgot` retornando `reset_token` na resposta; redefinição da senha da Camila e login como admin.
8. **OpenAPI (~3h30).** Descoberta de `GET /api/openapi.json` aberto, revelando `/api/auth/*`, `/api/users/*`, `/api/tickets/*`, `/api/documents/*`.
9. **Autorização entre papéis (~4h).** Reset e login como Ana (analyst, Aurora) e Bruno (analyst, Vega); comparação de acesso entre os três papéis.
10. **XSS armazenado (~4h30).** Comentário com `<script>` aceito e devolvido cru em `GET /api/tickets/{id}`.
11. **CVE de Brotli (~5h).** Teste de `Content-Encoding: br` no login; servidor responde 500 com `unsupported content encoding "br"` e stack trace. CVE não alcançável.
12. **Fechamento (~5h30).** Consolidação das evidências, geração dos prints e redação do relatório e destas notas.

## Mapa do alvo

**Fato observado.** Aplicação Node.js/Express atrás de Cloudflare e Render. Portal de chamados multi-tenant ("Mesa Norte") com três usuários conhecidos: Camila Nogueira (admin, Mesa Norte), Ana Costa (analyst, Aurora Alimentos) e Bruno Lima (analyst, Vega Logística). Fluxos observados: login por formulário em `/login`, login por API em `/api/auth/login`, reset de senha em `/api/auth/forgot` e `/api/auth/reset`, perfil em `/api/users/me`, chamados em `/api/tickets`, documentos em `/api/documents`, busca em `/api/tickets/search`. A especificação OpenAPI está pública em `/api/openapi.json`.

**Suposição.** O sufixo `/api/v1/users` parece ser uma versão legada da API, mantida sem o middleware de autenticação que protege `/api/users`. A suposição é consistente com o `401` em `/api/users` e o `200` em `/api/v1/users`, mas não foi confirmada no código.

## Hipóteses

| Hipótese | Por que surgiu | Teste | Desfecho |
| --- | --- | --- | --- |
| `/api/v1/users` expõe PII sem autenticação | Resposta 200 sem cookie | `GET /api/v1/users` anônimo | Confirmada |
| SQLi no login | Formulário clássico, entrada concatenada seria comum | Aspa simples, `' OR 1=1`, `sqlmap` | Descartada: resposta `302 35` com aspa simples, indicando consulta parametrizada; WAF bloqueia payloads clássicos |
| XSS refletido no login | Mensagem de erro renderizada no servidor | Payloads em `erro`, `email`, `next` | Descartada: respostas de tamanho fixo (990 bytes), sem reflexo |
| XSS refletido na busca | Parâmetro `q` refletido na página de resultados | `q=<script>alert(1)</script>` autenticado | Confirmada: `<script>` cru no HTML da busca |
| Reset de senha vaza token | `/api/auth/forgot` responde 200 com JSON | `POST /api/auth/forgot` com e-mail da admin | Confirmada: `reset_token` no corpo |
| CORS permite leitura autenticada | `OPTIONS` reflete `Origin` arbitrário | `GET /api/tickets` com `Origin: evil` e cookie | Confirmada |
| CORS permite escrita autenticada | `Allow-Methods` inclui `PATCH` | `PATCH /api/users/me` com `Origin: evil` e cookie | Confirmada: `200`, `title` alterado |
| Autorização entre papéis é decorativa | `/api/v1/users` aberto e `/api/users` protegido | Comparação de acesso entre Camila, Ana e Bruno | Confirmada: os três papéis acessam os mesmos recursos |
| XSS armazenado em comentário | Endpoint de comentário aceita texto livre | `POST /api/tickets/1/comments` com `<script>` | Confirmada no armazenamento; execução no navegador não confirmada por falta de front que renderize o comentário |
| Node2Shell (CVE-2026-111497) explorável | Versão de Node reportada em `/api/health` | `Content-Encoding: br` no login | Descartada: servidor rejeita Brotli e a versão reportada está acima da corrigida |
| Injeção em `/api/tickets/search` | Parâmetro `q` na especificação | `q=teste'` com sessão autenticada | Descartada: a aspa simples não quebra a consulta (resposta 200, sem erro); comportamento compatível com consulta parametrizada |
| IDOR em `/api/tickets/{id}` e `/api/documents/{id}` | IDs sequenciais na especificação | Varredura de IDs com a sessão da Camila | Inconclusiva: a varredura não foi concluída; a exposição cross-tenant já está coberta pelo achado de autorização |

## Decisões de ferramenta

| Pergunta que eu tinha | Ferramenta | O que a saída mudou na condução |
| --- | --- | --- |
| O host é o alvo certo e o que está na frente dele | `ping` e `nmap` | Confirmou Cloudflare e Render; orientou o ritmo e o cuidado com o WAF |
| Quais rotas existem | `ffuf` com wordlist de diretórios | Revelou `/documentos`, `/perfil`, `/busca`, `/chamados`, todas protegidas |
| O login não é injetável | `curl` manual e `sqlmap` | `curl` mostrou consulta parametrizada; `sqlmap` foi interrompido pelo WAF e não mudou a conclusão |
| A API expõe mais do que a página | `curl` e leitura do `app.js` | Levou ao `/api/v1/users` aberto e, depois, ao `/api/openapi.json` |
| O CORS é explorável com credenciais | `curl` com `Origin` e cookie | Confirmou leitura e escrita cross-origin |
| O servidor não aceita Brotli | `curl` com `Content-Encoding: br` | Confirmou rejeição e descartou a CVE |

## Achados, da hipótese à conclusão

### Account takeover de qualquer conta, inclusive do administrador

- **Observação:** `/api/auth/forgot` responde 200 com JSON contendo `reset_token`.
- **Hipótese:** o token de reset é entregue ao solicitante em vez de ser enviado apenas por e-mail.
- **O que confirmaria:** `POST /api/auth/reset` com o token retornado alterando a senha de outro usuário.
- **O que derrubaria:** token inválido, expirado, ou reset exigindo confirmação por e-mail.
- **O que foi feito:** pedido de reset para o e-mail da admin, uso do token em `/api/auth/reset`, login como admin.
- **Evidência:** `entregas/evidencias/api-auth-forgot.png` ([link](entregas/evidencias/api-auth-forgot.png)), `entregas/evidencias/api-auth-reset.png` ([link](entregas/evidencias/api-auth-reset.png)) (token redactado).
- **Conclusão:** confirmado; a conta da admin foi assumida.
- **Confiança:** alta. Subiria a certeza apenas com a confirmação de que o token também é aceito em outro navegador, o que já está implícito pelo uso via cliente HTTP.
- **Por que esta severidade:** permite assumir qualquer conta, incluindo a de maior privilégio, sem autenticação prévia.

### Autorização quebrada (Broken Authorization) entre papéis e tenants

- **Observação:** `/api/v1/users` aberto e `/api/users` protegido, sugerindo inconsistência de middleware entre versões da API.
- **Hipótese:** o servidor não valida `role` nem `client_name` nas rotas autenticadas.
- **O que confirmaria:** um analyst acessando dados de outro tenant e executando funções de admin.
- **O que derrubaria:** respostas 403 para recursos de outro tenant ou para funções de admin.
- **O que foi feito:** reset e login como Ana e Bruno; comparação de acesso a `/api/users`, `/api/users/{id}`, `/api/tickets`, `/api/documents` e `PATCH /api/users/me` entre os três papéis.
- **Evidência:** [idor-ticket.png](entregas/evidencias/idor-ticket.png), [idor-ticket2.png](entregas/evidencias/idor-ticket2.png), [idor-ticket3.png](entregas/evidencias/idor-ticket3.png), [idor-ticket4.png](entregas/evidencias/idor-ticket4.png), [idor-documentos.png](entregas/evidencias/idor-documentos.png) e [idor-documentos2.png](entregas/evidencias/idor-documentos2.png).
- **Conclusão:** confirmado; os três papéis acessam os mesmos recursos.
- **Confiança:** alta para leitura; média para escrita cross-tenant, que foi observada em comentário e em `PATCH` no próprio perfil, mas não foi varrida em todos os recursos.
- **Por que esta severidade:** quebra o isolamento multi-tenant e o modelo de papéis, expondo dados de todos os clientes a qualquer usuário autenticado.

### CORS refletindo origem arbitrária com credenciais

- **Observação:** `OPTIONS` com `Origin: https://evil.example` retorna o próprio `Origin` com `Allow-Credentials: true` e métodos de escrita.
- **Hipótese:** qualquer site externo consegue ler e escrever em nome de um usuário logado.
- **O que confirmaria:** `GET /api/tickets` e `PATCH /api/users/me` com `Origin: evil` e cookie retornando 200.
- **O que derrubaria:** `Allow-Origin` fixo no domínio da aplicação, ou ausência de `Allow-Credentials`.
- **O que foi feito:** `curl` com `Origin: evil` e cookie da admin em leitura e escrita.
- **Evidência:** [CORS-poc.png](entregas/evidencias/CORS-poc.png), [CORS-failed.png](entregas/evidencias/CORS-failed.png).
- **Conclusão:** confirmado no servidor. No navegador, o cookie com `SameSite` restritivo impede o envio cross-origin, então a exploração depende dessa configuração de cookie.
- **Confiança:** alta para o servidor; média para o impacto no navegador, que depende do `SameSite` e do domínio do cookie.
- **Por que esta severidade:** a configuração do servidor autoriza leitura e escrita cross-origin; a mitigação atual é do navegador, não da aplicação.

### XSS armazenado em comentário de chamado

- **Observação:** o endpoint de comentário aceita texto livre.
- **Hipótese:** o comentário é armazenado e renderizado sem escape.
- **O que confirmaria:** `<script>` devolvido cru em `GET /api/tickets/{id}`.
- **O que derrubaria:** escape na saída ou sanitização na entrada.
- **O que foi feito:** `POST /api/tickets/1/comments` com `<script>alert(1)</script>` e leitura do chamado.
- **Evidência:** [xss-armazenado-comentarios-chamados.png](entregas/evidencias/xss-armazenado-comentarios-chamados.png).
- **Conclusão:** confirmado no armazenamento; a execução no navegador não foi observada porque não há front que renderize o comentário no contexto testado.
- **Confiança:** alta para o armazenamento; média para a execução.
- **Por que esta severidade:** executa no navegador de quem visualizar o chamado, com impacto alto.

### XSS refletido na busca de chamados

- **Observação:** o parâmetro `q` da busca é refletido na página de resultados.
- **Hipótese:** o valor de `q` é inserido no HTML sem escape, permitindo execução de script.
- **O que confirmaria:** `<script>` cru na resposta HTML (não escapado) ao buscar o payload.
- **O que derrubaria:** `&lt;script&gt;` na resposta (escape) ou o valor não refletido no HTML.
- **O que foi feito:** busca com `q=<script>alert(1)</script>` estando autenticado, e inspeção do HTML retornado.
- **Evidência:** [xss-refletido-campo-de-busca.png](entregas/evidencias/xss-refletido-campo-de-busca.png).
- **Conclusão:** confirmado; o payload aparece cru no HTML da busca e executa no navegador de quem abrir o link.
- **Confiança:** alta para o reflexo no HTML; a execução foi observada no contexto da busca autenticada.
- **Por que esta severidade:** executa no navegador da vítima a partir de um link, com impacto alto; combinado com o CORS, permite exfiltração de sessão.

### Especificação OpenAPI exposta

- **Observação:** `/api/openapi.json` acessível sem autenticação.
- **Hipótese:** a especificação completa está pública.
- **O que confirmaria:** o JSON com todas as rotas e parâmetros.
- **O que derrubaria:** 401 ou 404 no caminho.
- **O que foi feito:** `GET /api/openapi.json`.
- **Evidência:** [ffuf-documentacao-api-exposta.png](entregas/evidencias/ffuf-documentacao-api-exposta.png), [documentacao-api.png](entregas/evidencias/documentacao-api.png).
- **Conclusão:** confirmado.
- **Confiança:** alta.
- **Por que esta severidade:** acelera o mapeamento da superfície.

## O que ficou de fora do relatório

- **WAF bloqueando payloads de SQLi:** comportamento do Cloudflare, não da aplicação. Não é achado.
- **`/documentos/..;/documentos` retornando 403 com corpo grande:** era a página de bloqueio do Cloudflare. Descartado após confirmar o mesmo comportamento em outros caminhos.
- **`/api/users/{id}` e `/api/tickets/{id}` com IDOR não varridos por completo:** a exposição cross-tenant já está coberta pelo achado de autorização; a varredura completa ficou como trabalho futuro.
- **Ausência de flags de cookie observáveis:** o login falho não emite `Set-Cookie` e o cookie de sessão só aparece após login bem-sucedido; a avaliação de flags ficou limitada ao que o navegador mostrou.
- **Node2Shell:** descartado, pois o servidor rejeita Brotli e versão está acima da corrigida.

## Parada

Parei de aprofundar quando a cadeia de account takeover e a quebra de autorização entre papéis já estavam provadas com evidências reproduzíveis. O ritmo do Cloudflare também limitou varreduras mais agressivas de IDs e de injeção. A prova mínima obtida cobre os achados do relatório e o que ficou pendente está listado como limitação e como trabalho futuro.

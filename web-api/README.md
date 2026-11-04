# Web e API

Terreno de aplicação e API. Não há cota de achado por classe. Na condução, o que conta é se o mapa do alvo gerou o teste. No raciocínio, se a hipótese de autorização, injeção ou sessão foi confirmada ou descartada com evidência. No relatório, se o achado diz quem faz o quê e como corrigir neste serviço. A lista abaixo é o vocabulário da vaga, não um roteiro.

## OWASP Top 10

Usamos as classes do Top 10 como vocabulário comum. O achado precisa dizer em qual classe cai e o que isso permite no alvo, não só citar o código (A01, A03, A07).

## IDOR e controle de acesso

Troca de identificador, método ou papel não pode entregar objeto de outro usuário. Um bom achado mostra os dois papéis, o identificador e a ação. A correção mora no servidor, no dono do recurso, não em esconder o id na interface.

## XSS

Conteúdo controlado pelo usuário não pode executar no browser de outra pessoa. O achado indica o ponto de entrada, o contexto (HTML, atributo, URL) e quem é afetado. A correção fala de encoding na saída, política de conteúdo quando fizer sentido, e do que não depende só do filtro de entrada.

## Injeção

Entrada que a aplicação concatena em consulta, comando ou template. O achado separa o dado do comando e aponta a consulta parametrizada, o ORM ou a API equivalente da stack. Não pedimos dump nem prova além do necessário para mostrar que a separação falhou.

## Autenticação e sessão

Login, recuperação, cookie, token e invalidação. O achado cobre um fluxo concreto: sessão que sobrevive ao logout, cookie sem flag adequada ao risco, token aceito fora do papel, ou objeto de autenticação previsível. A correção é do fluxo e do atributo da sessão, descrita para esta aplicação.

## O que fica no relatório

Para cada item: onde, papel, impacto, evidência redactada e correção. Detalhe de reprodução só o bastante para outra pessoa do time repetir no laboratório, sem dado de terceiro.

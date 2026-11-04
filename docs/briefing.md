# Briefing

O laboratório é a API em https://offensive-seg-challange-api.onrender.com. Só esse host está em escopo. Janela, papéis e canal de dúvida estão em [`escopo.md`](escopo.md).

A Hous3 olha três coisas, nesta ordem de importância prática: como você conduz o pentest, se a linha de raciocínio fecha, e se o relatório é utilizável por quem vai priorizar e por quem vai corrigir. Quantidade de achado não ganha de nenhuma das três.

## O que entregar

1. **Relatório** em `entregas/relatorio.md`, a partir de [`template-relatorio.md`](template-relatorio.md). É o documento que você entregaria ao cliente. Resumo que prioriza, achado com impacto e correção, evidência que prova a frase. Sem diário de bordo aqui.
2. **Notas** em `entregas/notas.md`, a partir de [`template-notas.md`](template-notas.md). É o caderno do pentest: ordem do trabalho, mapa do alvo, hipótese, o que você testou, o que confirmou, o que descartou e por quê. A conclusão do relatório tem que caber nestas notas.
3. **Evidência** em `entregas/evidencias/`: request/response redactado, print ou saída de ferramenta já limpa. Sem cookie de sessão, token, senha ou dado de terceiro.

## O que uma boa entrega mostra

No relatório:

- Dá para saber o que corrigir primeiro sem ler o caderno.
- Cada achado diz onde está, quem consegue fazer o quê, qual o impacto e qual a correção nesta stack.
- A severidade acompanha o impacto descrito, não o score do scanner.
- A evidência prova aquela frase e não vaza segredo.

Nas notas:

- Dá para reconstruir a ordem do pentest.
- Cada achado do relatório começa numa hipótese, não num log colado.
- Hipótese derrubada aparece, com o motivo.
- A escolha da ferramenta está amarrada a uma pergunta. Burp Suite, nmap, ffuf ou gobuster e nuclei entram quando respondem algo que você precisava saber.

## Dúvida e parada

Dúvida de escopo vai pelo canal escrito em `escopo.md`. Se aparecer dado real de cliente, credencial de produção ou um efeito além do laboratório, pare, registre a hora nas notas e avise pelo mesmo canal.

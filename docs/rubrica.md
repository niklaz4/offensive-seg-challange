# Rubrica

Três eixos, cada um de 0 a 3. A nota lê o relatório, as notas de raciocínio e a evidência juntos. Volume de achado e volume de log não sobem nota.

| Nota | Significado |
| --- | --- |
| 0 | Eixo ausente, ou a entrega quebra a regra de engajamento. |
| 1 | Há material, mas não dá para confiar: a condução não se reconstrói, o raciocínio não liga hipótese a conclusão, ou o relatório não serve para decidir. |
| 2 | O eixo fecha. Outra pessoa do time entende o que foi feito, por que a conclusão é essa e o que corrigir. |
| 3 | Além do item anterior: a pessoa prioriza, explicita o que não sabe e separa fato de suposição. O relatório e as notas contam a mesma história. |

## 1. Condução do pentest

A pergunta é se a pessoa fez um pentest, não se rodou ferramenta.

Olhamos a ordem: entendeu o alvo, mapeou superfície, escolheu o que aprofundar, respeitou escopo e ritmo, parou quando a prova já bastava. O terreno técnico aparece aqui.

- Web e API: IDOR, XSS, injeção, autenticação e sessão entram quando o alvo dá motivo, não como lista obrigatória. O teste de autorização compara papéis. O teste de sessão olha o fluxo, não só a flag solta.
- Ferramentas: Burp, nmap, ffuf ou gobuster e nuclei respondem uma pergunta nomeada. Saída sem decisão não conta como condução.

Nota 1: scan amplo, pouco teste manual, ou cobertura que ignora o que o próprio mapa mostrou. Nota 3: dá para ver por que o tempo foi para um fluxo e não para outro, e o escopo foi respeitado o tempo todo.

## 2. Linha de raciocínio

A pergunta é se a conclusão está sustentada.

O caminho esperado, nas notas e refletido no relatório: observação, hipótese, teste que confirmaria ou derrubaria, evidência, conclusão. Conclusão pode ser achado, falso positivo ou inconclusivo.

Nota 1: o achado aparece pronto, sem origem, ou a evidência não mostra o que o texto afirma. Nota 3: hipóteses mortas estão escritas, a confiança declara o que falta, e nenhum achado do relatório fica órfão das notas. Contradição entre as duas peças segura este eixo em 1, mesmo que o texto de cada uma pareça bom sozinho.

## 3. Relatório

A pergunta é se a Hous3 conseguiria usar o documento.

O resumo diz o que corrigir primeiro. O achado é localizável, tem impacto no laboratório e correção específica da stack. A severidade é defendida pelo impacto, não por um rótulo de scanner. A evidência está redactada e aponta para o arquivo certo. O relatório não é o caderno: metodologia curta, sem despejo de wordlist, sem payload solto sem contexto.

Nota 1: log comentado, tudo na mesma severidade, ou correção genérica (“validar a entrada”). Nota 3: um tech lead prioriza pelo resumo e quem corrige implementa a correção sem perguntar de novo o que quebrou.

## Sinal que encerra a avaliação

Alvo fora do escopo, teste destrutivo, ou segredo e dado pessoal no git. O eixo afetado vai a 0. Se o vazamento é a entrega inteira, a avaliação para.

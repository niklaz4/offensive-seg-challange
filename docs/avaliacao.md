# Como avaliar a entrega

Uso de quem corrige o desafio. Os critérios são os mesmos que a pessoa candidata leu na rubrica: condução do pentest, linha de raciocínio e relatório.

Leia as duas peças. O relatório sozinho não mostra o raciocínio. As notas sozinhas não mostram se a pessoa sabe entregar.

## Ordem de leitura

1. **`entregas/notas.md`.** Reconstrua o pentest em ordem. Marque onde a hipótese nasce, que teste a enfrenta e qual foi o desfecho (confirmada, descartada, inconclusiva).
2. **`entregas/relatorio.md`, como cliente.** Sem voltar às notas: o resumo diz o que fazer primeiro? Cada achado dá para localizar, medir e corrigir?
3. **As duas lado a lado.** Todo achado do relatório tem hipótese e evidência nas notas? Algum ponto grave das notas sumiu do relatório? A severidade do relatório combina com o impacto que as notas mostram?
4. **`entregas/evidencias/`.** O arquivo prova a frase do achado? Cookie, token, senha e dado pessoal ficaram de fora?
5. **Regras.** Alvo, janela, ritmo, prova mínima. Qualquer quebra vai anotada antes da nota.

## O que anotar em cada eixo

Responda em uma frase, com o trecho da entrega que sustenta.

**Condução.** Dá para dizer por onde a pessoa entrou e por que aprofundou ali? O mapa do alvo gerou os testes, ou os testes são um roteiro genérico de OWASP colado em cima da aplicação? A ferramenta certa aparece na pergunta certa? Web e API foram olhados no que importava?

**Raciocínio.** A cadeia observação → hipótese → teste → evidência → conclusão está escrita? Há hipótese descartada com motivo, ou só achado que deu certo? O inconclusivo está marcado como inconclusivo? Relatório e notas concordam?

**Relatório.** O resumo funciona para quem prioriza? A correção é desta stack? A severidade sobe e desce com o impacto? Um leitor que não viu o laboratório entende o que quebrou?

## Nota

Use a escala de [`rubrica.md`](rubrica.md). Escreva o número e a frase. Não medie os eixos num único “passou” antes de registrar os três: uma condução boa com relatório inutilizável é um resultado, e o contrário também.

| Eixo | Nota | Frase |
| --- | --- | --- |
| Condução do pentest | | |
| Linha de raciocínio | | |
| Relatório | | |

## Sinais que costumam enganar

- Muitos achados de scanner reproduzidos só pela metade. Conte como ruído até haver teste manual e impacto.
- Relatório longo com pouca decisão. Tamanho não é condução.
- Notas ricas e relatório que esconde o item mais grave, ou o infla. Isso é falha de entrega e de raciocínio.
- Correção que serviria para qualquer aplicação. Peça, na frase da nota, se quem corrige saberia o que mudar neste alvo.

## O que a avaliação não pede

Não compare a pessoa com um gabarito de “todos os bugs do laboratório”. O gabarito, quando existir, serve para ver se um achado grave óbvio passou em branco sem menção nas notas. Silêncio sobre um ponto central do alvo, sem hipótese nem descarte, pesa na condução. Citar o ponto e descartá-lo com motivo é raciocínio, mesmo se o descarte estiver errado: erre a conclusão nas notas, e registre o erro, sem tratar o descarte como ausência de trabalho.

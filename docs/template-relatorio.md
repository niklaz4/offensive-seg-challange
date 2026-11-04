# Relatório — Offensive Security Engineer

Copie este arquivo para `entregas/relatorio.md` e preencha. Apague as instruções entre colchetes.

Este é o documento para o cliente. A ordem do pentest, as hipóteses e os descartes ficam em [`template-notas.md`](template-notas.md). Aqui entra só o que alguém precisa para priorizar e corrigir.

## Resumo

[Cinco a dez linhas. Qual o risco principal do laboratório e o que corrigir primeiro. Sem lista de ferramenta.]

| Achado | Classe | Severidade | Onde |
| --- | --- | --- | --- |
| | | | |

Severidade: crítica, alta, média, baixa ou informativa. A severidade acompanha o impacto descrito no achado.

## Escopo

[O que foi testado, a janela e os papéis. O que ficou de fora.]

## Metodologia

[Um parágrafo. Como a superfície foi entendida e o que foi feito à mão. O caderno completo está nas notas.]

## Achados

Do mais grave ao menos grave. Repita o bloco.

### [Título curto, dizendo o que um atacante consegue]

- **Classe:** [IDOR, XSS, injeção, autenticação, sessão, ou outra nomeada]
- **Severidade:**
- **Onde:** [rota, parâmetro ou recurso]
- **Quem:** [qual papel consegue, e qual papel não deveria]
- **Impacto:** [o que isso muda no laboratório]
- **Correção:** [a mudança nesta aplicação]
- **Evidência:** [caminho em `entregas/evidencias/`, redactado]

## Recomendação

[Ordem de correção em poucas linhas, alinhada ao resumo. Se dois achados caem no mesmo controle, una.]

## Limitações

[O que a janela não permitiu afirmar. Sem transformar limitação em achado.]

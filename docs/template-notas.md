# Notas do pentest

Copie este arquivo para `entregas/notas.md` e preencha. Apague as instruções entre colchetes.

Este é o caderno. Ele não substitui o relatório. Quem avalia usa estas notas para ver a condução e a linha de raciocínio, e confere se o relatório segue daqui.

## Ordem do trabalho

[Lista curta, em ordem. O que você fez primeiro, o que isso mostrou e o que você decidiu fazer depois. Inclua hora aproximada se a janela for apertada.]

1.

## Mapa do alvo

[O que a aplicação é, quais papéis existem, quais fluxos e recursos importam. Fato observado separado do que você supôs.]

## Hipóteses

Uma linha por hipótese, tenha virado achado ou não.

| Hipótese | Por que surgiu | Teste | Desfecho |
| --- | --- | --- | --- |
| | | | confirmada, descartada ou inconclusiva |

No descarte e no inconclusivo, uma frase de motivo abaixo da tabela ou na própria célula. “Não deu tempo” serve se for verdade; “não achei” sem dizer o que foi olhado não serve.

## Decisões de ferramenta

| Pergunta que eu tinha | Ferramenta | O que a saída mudou na condução |
| --- | --- | --- |
| | | |

Ferramenta que rodou e não mudou nada também entra, com esse desfecho. Burp Suite, nmap, ffuf ou gobuster e nuclei são o conjunto esperado quando a pergunta pedir.

## Achados, da hipótese à conclusão

Repita por achado que entrou no relatório.

### [Mesmo título do relatório]

- **Observação:** [o que você viu antes de testar]
- **Hipótese:**
- **O que confirmaria:**
- **O que derrubaria:**
- **O que foi feito:**
- **Evidência:** [caminho do arquivo]
- **Conclusão:**
- **Confiança:** [alta, média ou baixa, e o que falta para subir]
- **Por que esta severidade:**

## O que ficou de fora do relatório

[Ponto que você viu e decidiu não promover a achado. Motivo: falso positivo, impacto baixo demais, prova insuficiente. Se não houve, escreva isso.]

## Parada

[Onde você parou de aprofundar e por quê. Prova mínima já obtida, ritmo do alvo, fim da janela, ou dado que não deveria aparecer.]

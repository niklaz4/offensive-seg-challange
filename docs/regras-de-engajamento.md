# Regras de engajamento

O desafio só existe dentro do laboratório publicado em [`escopo.md`](escopo.md). Tudo fora dessa lista está fora do teste.

## Permitido

- Avaliação manual e com as ferramentas previstas (Burp Suite, nmap, ffuf, gobuster, nuclei) contra o alvo da janela.
- Prova mínima de impacto: mostrar que o controle falha, com o menor dado possível.

## Proibido

- Alvo que não esteja em `escopo.md`, inclusive subdomínio, bucket, projeto ou conta descobertos no caminho.
- Negação de serviço, carga agressiva, fuzz sem limite de taxa e template de scanner feito para derrubar serviço.
- Força bruta de credencial, engenharia social, phishing e qualquer ataque a pessoa.
- Persistência, alteração destrutiva, exfiltração e dump de base.
- Commit de segredo, sessão, token, dado pessoal ou dado de cliente.

## Quando parar

Pare na hora se o teste sair do laboratório, se a aplicação ficar instável, ou se surgir credencial ou dado que não deveria estar ali. Registre a hora e avise o canal do escopo. Não explore o achado além do necessário para descrevê-lo.

## Entrega

O material fica neste repositório, na branch e no prazo combinados no escopo. Duas peças em Markdown: o relatório para o cliente e as notas com a linha de raciocínio. Evidência redactada.

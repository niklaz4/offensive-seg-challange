# Ferramentas

A vaga pede prática real com quatro famílias. Na avaliação, a ferramenta entra na condução só quando as notas mostram qual pergunta ela respondeu e o que mudou no pentest depois da saída. Log bruto no relatório não pontua. Escolher não usar uma delas, com motivo, é uma decisão válida.

## Burp Suite

Proxy e histórico para enxergar o que a aplicação realmente troca. Repeater para confirmar um achado à mão. Intruder ou sessão autenticada só dentro do escopo e com taxa baixa o bastante para o laboratório continuar de pé. O projeto do Burp não vai para o git. A evidência é o request e o response já redactados.

## nmap

Descoberta do que o escopo publica: host vivo, porta e serviço, na janela combinada. A saída relevante entra no relatório como tabela curta (porta, serviço, o que isso muda na avaliação). Scan amplo fora da lista de alvos fere a regra.

## ffuf ou gobuster

Descoberta de caminho e recurso a partir de uma lista proporcional ao alvo. Extensão coerente com a stack (por exemplo a da aplicação, não um pacote genérico enorme). Resposta 429, 503 ou lentidão encerra ou reduz o ritmo. O achado é o caminho que importa e o que ele expõe, não a wordlist inteira.

## nuclei

Templates atualizados, severidade e tag escolhidas para o tipo de alvo. Template de negação de serviço fica de fora. Todo resultado do nuclei é lido: o que reproduz no laboratório vira achado; o que não reproduz vai para a seção de hipóteses descartadas, com o motivo.

## Evidência

Guarde em `entregas/evidencias/` só o recorte que prova o ponto. Remova cookie, token, senha e corpo com dado pessoal antes do commit.

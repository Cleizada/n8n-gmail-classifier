# Organizador de Gmail com n8n e IA local

Minha caixa de entrada estava cheia de promoção e spam. Em vez de limpar na mão, montei um fluxo no n8n que lê os e-mails e decide o que fazer com cada um. A classificação é feita por um modelo que roda no meu computador, então o conteúdo dos e-mails não vai para nenhuma API de terceiros.

Numa rodada de uma noite ele passou por cerca de 500 e-mails.

## Como funciona

1. O n8n olha a caixa de entrada a cada 5 minutos, só os e-mails não lidos.
2. Cada um vai para o DeepSeek R1 14B, rodando no Ollama, que responde com uma categoria e um resumo de uma linha: SPAM, PROMOCIONAL, FINANCEIRO, VAGA DE EMPREGO, IMPORTANTE ou OUTROS.
3. Um nó Switch lê a categoria. SPAM e PROMOCIONAL são apagados, e o resto ganha uma label no Gmail.

O modelo recebe só o trecho de prévia (snippet) do e-mail, não o texto inteiro.

## Como rodar

1. Instale o [Ollama](https://ollama.com) e baixe o modelo: `ollama pull deepseek-r1:14b`
2. Suba o Ollama de um jeito que o n8n alcance: `OLLAMA_HOST=0.0.0.0 ollama serve`
3. Importe o `n8n-gmail-classifier-workflow.json` no n8n.
4. Crie a credencial do Gmail (OAuth2) e a do Ollama.
5. Crie as labels no Gmail e coloque o ID de cada uma nos nós correspondentes.

Teste primeiro com poucos e-mails e olhe as classificações antes de deixar a exclusão ligada.

## Onde eu travei

- O token do Gmail expirava no meio de execuções longas.
- O n8n não conseguia falar com o Ollama até eu subir o Ollama com `OLLAMA_HOST=0.0.0.0`.

## Cuidados

- O nó de exclusão apaga a mensagem de vez, sem passar pela lixeira. Por isso eu prefiro revisar antes.
- O DeepSeek R1 "pensa em voz alta" e às vezes isso vem junto na resposta, o que pode confundir o Switch. O ajuste que eu ainda quero fazer é pegar só a primeira linha da resposta antes de rotear.

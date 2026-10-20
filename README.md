# n8n Gmail Classifier (LLM local)

Workflow que **classifica e organiza a caixa de entrada do Gmail em massa usando um LLM rodando localmente**, sem enviar e-mails para nenhuma API de terceiros.

## Resultado

Processou cerca de **500 e-mails em uma rodada noturna**: lixo foi apagado e o restante recebeu labels automaticamente.

## Como funciona

```
Gmail Trigger (e-mails não lidos, a cada 5 min)
      │
      ▼
Ollama + DeepSeek R1 14B (local) ──► classifica cada e-mail
      │
      ├── SPAM / PROMOCIONAL ──► apagado
      └── FINANCEIRO / VAGA / IMPORTANTE / OUTROS ──► recebe label no Gmail
```

## Stack

- **n8n** (self-hosted)
- **Ollama** com **DeepSeek R1 14B**, rodando local
- **Gmail API** (OAuth2)

O modelo lê o *snippet* do e-mail e responde com a categoria e um resumo de uma linha; um nó **Switch** roteia pela categoria.

## Por que um LLM local?

- **Privacidade:** o conteúdo dos e-mails não sai da máquina.
- **Custo zero por e-mail**, o que importa em lotes grandes.

## Como rodar

1. Instale o [Ollama](https://ollama.com) e baixe o modelo: `ollama pull deepseek-r1:14b`
2. Em um terminal, exponha o Ollama para o n8n:
   ```bash
   OLLAMA_HOST=0.0.0.0 ollama serve
   ```
3. Importe o `n8n-gmail-classifier-workflow.json` no n8n.
4. Configure a credencial **Gmail OAuth2** no n8n e aponte o nó do Ollama para `http://localhost:11434`.
5. Crie as labels no Gmail e informe os IDs em cada nó de label.
6. Rode primeiro em um lote pequeno e **revise as classificações antes de ativar a exclusão automática**.

## Problemas resolvidos

- **Expiração do token OAuth** do Gmail durante execuções longas.
- **Conexão n8n ↔ Ollama**: foi preciso subir o Ollama com `OLLAMA_HOST=0.0.0.0` para o n8n alcançá-lo.

## Limitações conhecidas

- O nó de exclusão **apaga a mensagem definitivamente**, então vale revisar as classificações antes.
- O DeepSeek R1 é um modelo de raciocínio e pode incluir texto de "pensamento" na resposta, o que pode confundir o Switch. Uma melhoria é extrair só a primeira linha da resposta antes de rotear.
- A classificação usa só o snippet, não o e-mail inteiro.

## Estrutura

```
├── n8n-gmail-classifier-workflow.json   # workflow exportado (sem credenciais)
├── README.md
└── docs/
    └── fluxo.png
```

## Autor

Cleidson Soares — [LinkedIn](https://www.linkedin.com/in/cleidson-soares)

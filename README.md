# ai-rag

RAG (Retrieval-Augmented Generation) sobre o livro **"Os Sertões"**, de Euclides da Cunha, feito com LangChain, Chroma e OpenAI.

O projeto responde a perguntas sobre o livro buscando os trechos mais relevantes do PDF e passando-os como contexto para o modelo. O enunciado completo do desafio está em [`challenge.md`](challenge.md).

## Status

O desafio pede três abordagens de RAG. Até agora:

| Abordagem  | Status       | Notebook                         |
| ---------- | ------------ | -------------------------------- |
| Naive RAG  | Implementado | [`src/rag.ipynb`](src/rag.ipynb) |
| Parent RAG | Pendente     | —                                |
| Rerank RAG | Pendente     | —                                |

## Como funciona

1. **Carga**: o PDF é lido página a página com `PyMuPDFLoader`.
2. **Limpeza**: linhas de cabeçalho e rodapé da edição são removidas, e páginas com menos de 300 caracteres são descartadas.
3. **Chunking**: `RecursiveCharacterTextSplitter` com chunks de 1500 caracteres e sobreposição de 200.
4. **Indexação**: os chunks viram embeddings (`OpenAIEmbeddings`) e são gravados em um índice Chroma persistido em `src/text_index/`.
5. **Recuperação**: busca MMR, que seleciona 5 trechos entre os 20 mais próximos para reduzir repetição.
6. **Resposta**: os trechos são enviados ao `gpt-4o-mini` em uma chain do tipo `stuff`.

## Estrutura

```
.
├── challenge.md      # enunciado do desafio e as 5 perguntas
├── os-sertoes.pdf    # livro usado como base de conhecimento
└── src/
    └── rag.ipynb     # Naive RAG
```

## Requisitos

- Python 3.10+
- Uma chave da API da OpenAI

## Instalação

```bash
git clone git@github.com:Faelst/ai-rag.git
cd ai-rag

python3 -m venv .venv
source .venv/bin/activate

pip install -U langchain langchain-classic langchain-openai langchain-community \
  langchain-text-splitters langchain-chroma pymupdf python-dotenv notebook
```

Crie um arquivo `.env` na raiz do projeto com a sua chave:

```
OPENAI_API_KEY=sua-chave-aqui
```

O `.env` está no `.gitignore` e não deve ser versionado.

## Uso

Abra o notebook a partir da pasta `src`, porque os caminhos do PDF e do índice são relativos a ela:

```bash
cd src
jupyter notebook rag.ipynb
```

Execute as células em ordem. A última aceita várias perguntas de uma vez:

- digite uma pergunta por linha e deixe a linha vazia para terminar; ou
- cole várias na mesma linha, separadas por `;`.

As perguntas são respondidas em paralelo, e cada resposta vem com as páginas do livro consultadas.

Também dá para chamar as funções direto em uma célula:

```python
ask("Quem foi Antônio Conselheiro?")                  # uma pergunta, mostrando os trechos recuperados
ask_many(["Pergunta 1", "Pergunta 2", "Pergunta 3"])  # várias, em paralelo
```

### Índice vetorial

A variável `REBUILD` controla a criação do índice:

- `REBUILD = True`: apaga `src/text_index/` e gera os embeddings do livro inteiro de novo, o que consome créditos da API. Use na primeira execução ou depois de mudar a limpeza ou o chunking.
- `REBUILD = False`: reaproveita o índice já gravado em disco.

O índice não é versionado; ele é gerado localmente na primeira execução.

## Perguntas do desafio

1. Qual é a visão de Euclides da Cunha sobre o ambiente natural do sertão nordestino e como ele influencia a vida dos habitantes?
2. Quais são as principais características da população sertaneja descritas por Euclides da Cunha? Como ele relaciona essas características com o ambiente em que vivem?
3. Qual foi o contexto histórico e político que levou à Guerra de Canudos, segundo Euclides da Cunha?
4. Como Euclides da Cunha descreve a figura de Antônio Conselheiro e seu papel na Guerra de Canudos?
5. Quais são os principais aspectos da crítica social e política presentes em "Os Sertões"? Como esses aspectos refletem a visão do autor sobre o Brasil da época?

# SynapsIA

**SynapsIA** is the digital scribe of the new era. Its name comes from the fusion of **Synapse**, the neural connections where knowledge resides, and **AI**, the intelligence that brings it to life.

It is not a simple reader; it is a craftsman of knowledge. Just as an ancient scribe forged texts that would last for centuries, **SynapsIA** ingests your documents, from the simplest to the most complex, and forges an intricate network of synaptic connections. Every piece of data, every concept, and every relationship becomes a golden thread in a vast knowledge base.

The result is a coherent, living digital mind, ready to be interrogated. Ask it a question, and it will respond with wisdom drawn from your own texts, allowing other AI systems to drink from this fountain of pure, contextualized knowledge.

> **SynapsIA: Forging knowledge, one synapse at a time.**

---

## Technical Purpose

`synapsia.py` is a command-line script that uses the LlamaIndex library to process a set of documents. It reads the files, splits them into chunks, and uses an embedding model (via an Ollama service) to convert them into numerical vectors. Finally, it stores these vectors in a persistent index on disk, creating a knowledge base ready to be used by a RAG (Retrieval-Augmented Generation) application.

## Prerequisites

1.  **Python 3.9+**: A recent version of Python installed on your local machine. We highly recommend using a virtual environment (`python -m venv .venv`).
2.  **Python Dependencies**: Install the required libraries by running `pip install -r requirements.txt` from within the `SynapsIA` directory.
3.  **Ollama Service**: The script needs to connect to a running Ollama instance. You can launch one using Podman. This command creates a persistent named volume called `synapsia_storage` to store the downloaded models, so you don't have to download them again every time.

    ```bash
    # Launch the Ollama container in the background
    podman run -d --rm -p 11434:11434 --name ollama-synapsia -v synapsia_storage:/root/.ollama ollama/ollama
    ```

4.  **Embedding Model**: Ensure the embedding model is available in your Ollama instance. **After starting the container**, pull the model using `podman exec`:

    ```bash
    # Tell the running 'ollama-synapsia' container to download the model
    podman exec -it ollama-synapsia ollama pull mxbai-embed-large
    ```

## Usage

The script is run from the terminal inside the `SynapsIA` project directory. It uses named arguments to specify paths and options.

**Command Syntax:**
```bash
python synapsia.py --docs <path_to_docs> --knowledge <path_to_knowledge_base> [OPTIONS]
```

### Required Arguments

* `--docs <path>`: The relative or absolute path to the directory containing your source documents.
* `--knowledge <path>`: The relative or absolute path to the output directory where the knowledge base will be saved.

### Options

* `--embed-model <model_name>`: The name of the embedding model to use from Ollama.
    * **Default:** `mxbai-embed-large`
* `--ollama-url <url>`: The base URL of the Ollama API service.
    * **Default:** `http://localhost:11434`
* `--embed-batch-size <number>`: Number of chunks to process at a time for embeddings.
    * **Default:** `5`
* `--chunk-size <number>`: The size of text chunks in tokens. Ideal for tuning how context is stored.
    * **Default:** `1024`
* `--chunk-overlap <number>`: The number of overlapping tokens between consecutive chunks to maintain context.
    * **Default:** `20`
* `-h`, `--help`: Show the help message and exit.

### Examples

#### Basic Usage

```bash
python synapsia.py --docs ./my_docs/ --knowledge ./my_kb/
```

#### Advanced Usage (Custom Model)

To run the script using a different embedding model, like `nomic-embed-text`:

```bash
python synapsia.py --docs ./my_docs/ --knowledge ./my_kb/ --embed-model nomic-embed-text
```

#### Advanced Usage (Tuned for Q&A)

This command is optimized for dense, factual documents like FAQs. It uses a smaller chunk size for more precise answers, a corresponding overlap to maintain context between chunks, and a very dynamic progress bar.

```bash

python synapsia.py \
    --docs ./faq_docs/ \
    --knowledge ./faq_kb/ \
    --chunk-size 256 \
    --chunk-overlap 25 \
    --embed-batch-size 1
```

## Querying the Knowledge Base

Once you have created a knowledge base with `synapsia.py`, you can ask it questions using the `synapsia_query.py` script. This tool loads the indexed knowledge, sends your query to a local LLM, and uses the documents as context to provide a relevant answer.

**Command Syntax:**
```bash
python synapsia_query.py --knowledge <path_to_knowledge_base> --query "Your question here" [OPTIONS]
```

### Required Arguments

* `--knowledge <path>`: The path to the directory where the knowledge base was saved.

* `--query <question>`: The question you want to ask, enclosed in quotes.

### Options

* `--llm-model <model_name>`: The name of the LLM to use from Ollama for generating the answer.
    * **Default:** `llama3`

* `--top-k <number>`: The number of most relevant text chunks to retrieve from the knowledge base to build the answer.
    * **Default:** `3`

* `--show-context`: A flag that, when present, displays the source text chunks and their relevance scores. This is extremely useful for debugging and understanding the model's reasoning.

* `--embed-model`, `--ollama-url`: These should generally match the values used during the ingestion process.

### Examples

### Basic Question

A simple query to get a direct answer from the knowledge base.


```bash
python synapsia_query.py \
    --knowledge ./my_kb/ \
    --query "What is the main purpose of Ansible?"
```

### Debugging Query
A query to not only get an answer but also see the top 2 source chunks that the RAG system used as context. This is ideal for fine-tuning your ingestion parameters.

```bash
python synapsia_query.py \
    --knowledge ./faq_kb/ \
    --query "How do I reset my password?" \
    --top-k 2 \
    --show-context
```
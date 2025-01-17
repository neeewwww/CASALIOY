<!--suppress HtmlDeprecatedAttribute -->
<div align="center">

> **NOTICE** NOW WITH
<a href="#chat-inside-gui-new-feature"><img src="https://img.shields.io/badge/GUI-blue.svg" alt="Roadmap 2023">
<br>
<p align="center">

# CASALIOY - Your local langchain toolkit

</p>

<h2>
<p>
<img height="300" src="https://github.com/su77ungr/GEEB-GPT/assets/69374354/2e59734c-0de7-4057-be7a-14729e1d5acd" alt="Qdrant"><br>

<a href="https://github.com/su77ungr/CASALIOY/issues/8"><img src="https://img.shields.io/badge/Feature-Requests-bc1439.svg" alt="Roadmap 2023"> [![Docker Pulls](https://badgen.net/docker/pulls/su77ungr/casalioy?icon=docker&label=pulls)](https://hub.docker.com/r/su77ungr/casalioy/)</a>

 <a href="https://www.buymeacoffee.com/cassowary" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="30" width="140"></a>
<br><br>
</p>
The fastest toolkit for air-gapped LLMs

[LangChain](https://github.com/hwchase17/langchain) + [LlamaCpp](https://pypi.org/project/llama-cpp-python/) + [qdrant](https://qdrant.tech/)

<br>
</h2>
</div>

# Setup

### Docker guide

```bash
docker pull su77ungr/casalioy:stable
```

```bash
docker run -it su77ungr/casalioy:stable /bin/bash
```
for older docker without GUI use `casalioy:latest` might deprecate soon

> Fetch the default models

```
cd models
wget https://huggingface.co/Pi3141/alpaca-native-7B-ggml/resolve/397e872bf4c83f4c642317a5bf65ce84a105786e/ggml-model-q4_0.bin &&
wget https://huggingface.co/eachadea/ggml-vicuna-7b-1.1/resolve/main/ggml-vic7b-q5_1.bin
cd ../
```

> All set! Proceed with ingesting your [dataset](#ingesting-your-own-dataset)

### Build it from source

> First install all requirements:

```shell
python -m pip install poetry
python -m poetry config virtualenvs.in-project true
python -m poetry install
. .venv/bin/activate
python -m pip install --force streamlit sentence_transformers  # Temporary bandaid fix, waiting for streamlit >=1.23
pre-commit install
```

If you want GPU support for llama-ccp:
```shell
pip uninstall -y llama-cpp-python
CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install --force llama-cpp-python
```

> Download the 2 models and place them in a folder called `./models`:

- LLM: default
  is [ggml-vic7b-q5_1](https://huggingface.co/eachadea/ggml-vicuna-7b-1.1/resolve/main/ggml-vic7b-q5_1.bin)
- Embedding: default
  to [ggml-model-q4_0](https://huggingface.co/Pi3141/alpaca-native-7B-ggml/resolve/397e872bf4c83f4c642317a5bf65ce84a105786e/ggml-model-q4_0.bin).

> > Edit the example.env to fit your models and rename it to .env

```env
# Generic
MODEL_N_CTX=1024
LLAMA_EMBEDDINGS_MODEL=models/ggml-model-q4_0.bin

# Ingestion
PERSIST_DIRECTORY=db
DOCUMENTS_DIRECTORY=source_documents
INGEST_CHUNK_SIZE=500
INGEST_CHUNK_OVERLAP=50

# Generation
MODEL_TYPE=LlamaCpp # GPT4All or LlamaCpp
MODEL_PATH=models/ggjt-v1-vic7b-uncensored-q4_0.bin
MODEL_TEMP=0.8
MODEL_STOP=###,\n
```

This should look like this

```
└── repo
      ├── startLLM.py
      ├── ingest.py
      ├── source_documents
      │   └── sample.csv
      │   └── shor.pdfstate_of_the_union.txt
      │   └── state_of_the_union.txt
      ├── models
      │   ├── ggml-vic7b-q5_1.bin
      │   └── ggml-model-q4_0.bin
      └── .env, convert.py, Dockerfile
```

## Ingesting your own dataset

To automatically ingest different data types (.txt, .pdf, .csv, .epub, .html, .docx, .pptx, .eml, .msg)

> This repo includes dummy [files](https://github.com/imartinez/privateGPT/blob/main/source_documents/)
> inside `source_documents` to run tests with.

```shell
python ingest.py # optional <path_to_your_data_directory>
```

Optional: use `y` flag to purge existing vectorstore and initialize fresh instance

```shell
python ingest.py # optional <path_to_your_data_directory> y
```

This spins up a local qdrant namespace inside the `db` folder containing the local vectorstore. Will take time,
depending on the size of your document.
You can ingest as many documents as you want by running `ingest`, and all will be accumulated in the local embeddings
database. To remove dataset simply remove `db` folder.

## Ask questions to your documents, locally!

In order to ask a question, run a command like:

```shell
python startLLM.py
```

And wait for the script to require your input.

```shell
> Enter a query:
```

Hit enter. You'll need to wait 20-30 seconds (depending on your machine) while the LLM model consumes the prompt and
prepares the answer. Once done, it will print the answer and the 4 sources it used as context from your documents; you
can then ask another question without re-running the script, just wait for the prompt again.

Note: you could turn off your internet connection, and the script inference would still work. No data gets out of your
local environment.

Type `exit` to finish the script.

## Chat inside GUI (new feature)

Introduced by [@alxspiker](https://github.com/alxspiker) -> see [#21](https://github.com/su77ungr/CASALIOY/pull/21)

```shell
streamlit run .\gui.py
```

# LLM options

### models outside of the GPT-J ecosphere  (work out of the box)

| Model                                                                                                                                            | BoolQ | PIQA | HellaSwag | WinoGrande | ARC-e | ARC-c | OBQA | Avg. |
|:-------------------------------------------------------------------------------------------------------------------------------------------------|:-----:|:----:|:---------:|:----------:|:-----:|:-----:|:----:|:----:|
| [ggml-vic-7b-uncensored](https://huggingface.co/datasets/dnato/ggjt-v1-vic7b-uncensored-q4_0.bin/resolve/main/ggjt-v1-vic7b-uncensored-q4_0.bin) | 73.4  | 74.8 |   63.4    |    64.7    | 54.9  | 36.0  | 40.2 | 58.2 |
| [GPT4All-13b-snoozy q5](https://huggingface.co/TheBloke/GPT4All-13B-snoozy-GGML/blob/main/GPT4All-13B-snoozy.ggml.q5_1.bin)                      | 83.3  | 79.2 |   75.0    |    71.3    | 60.9  | 44.2  | 43.4 | 65.3 |

### models inside of the GPT-J ecosphere

| Model                                                                             | BoolQ | PIQA | HellaSwag | WinoGrande | ARC-e | ARC-c | OBQA | Avg. |
|:----------------------------------------------------------------------------------|:-----:|:----:|:---------:|:----------:|:-----:|:-----:|:----:|:----:|
| GPT4All-J 6B v1.0                                                                 | 73.4  | 74.8 |   63.4    |    64.7    | 54.9  | 36.0  | 40.2 | 58.2 |
| [GPT4All-J v1.1-breezy](https://gpt4all.io/models/ggml-gpt4all-j-v1.1-breezy.bin) | 74.0  | 75.1 |   63.2    |    63.6    | 55.4  | 34.9  | 38.4 | 57.8 |
| [GPT4All-J v1.2-jazzy](https://gpt4all.io/models/ggml-gpt4all-j-v1.2-jazzy.bin)   | 74.8  | 74.9 |   63.6    |    63.8    | 56.6  | 35.3  | 41.0 | 58.6 |
| [GPT4All-J v1.3-groovy](https://gpt4all.io/models/ggml-gpt4all-j-v1.3-groovy.bin) | 73.6  | 74.3 |   63.8    |    63.5    | 57.7  | 35.0  | 38.8 | 58.1 |
| [GPT4All-J Lora 6B](https://gpt4all.io/models/)                                   | 68.6  | 75.8 |   66.2    |    63.5    | 56.4  | 35.7  | 40.2 | 58.1 |

all the supported models from [here](https://huggingface.co/nomic-ai/gpt4all-13b-snoozy) (custom LLMs in Pipeline)

### Convert GGML model to GGJT-ready model v1 (for truncation error or not supported models)

1. Download ready-to-use models

> Browse Hugging Face for [models](https://huggingface.co/)

2. Convert locally

> ``` python convert.py``` [see discussion](https://github.com/su77ungr/CASALIOY/issues/10#issue-1706854398)

# Pipeline

<br><br>

<img src="https://qdrant.tech/articles_data/langchain-integration/flow-diagram.png"></img>
<br><br>

Selecting the right local models and the power of `LangChain` you can run the entire pipeline locally, without any data
leaving your environment, and with reasonable performance.

- `ingest.py` uses `LangChain` tools to parse the document and create embeddings locally using `LlamaCppEmbeddings`. It
  then stores the result in a local vector database using `Qdrant` vector store.

- `startLLM.py` can handle every LLM that is llamacpp compatible (default `GPT4All-J`). The context for the answers is
  extracted from the local vector store using a similarity search to locate the right piece of context from the docs.

<br><br>

# Disclaimer

The contents of this repository are provided "as is" and without warranties of any kind, whether express or implied. We
do not warrant or represent that the information contained in this repository is accurate, complete, or up-to-date. We
expressly disclaim any and all liability for any errors or omissions in the content of this repository.

By using this repository, you are agreeing to comply with and be bound by the above disclaimer. If you do not agree with
any part of this disclaimer, please do not use this repository.

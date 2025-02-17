---
id: integrate_with_jina.md
summary: This guide demonstrates how to use Jina embeddings and Milvus to conduct similarity search and retrieval tasks.  
title: Integrate Milvus with Jina
---

# Integrate Milvus with Jina AI

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/milvus_with_Jina.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

This guide demonstrates how to use Jina embeddings and Milvus to conduct similarity search and retrieval tasks.  

## Who is Jina AI

Jina AI, founded in 2020 in Berlin, is a pioneering AI company focused on revolutionizing the future of artificial intelligence through its search foundation. Specializing in multimodal AI, Jina AI aims to empower businesses and developers to harness the power of multimodal data for value creation and cost savings through its integrated suite of components, including embeddings, rerankers, prompt ops, and core infrastructure.

Jina AI's cutting-edge embeddings boast top-tier performance, featuring an 8192 token-length model ideal for comprehensive data representation. Offering multilingual support and seamless integration with leading platforms like OpenAI, these embeddings facilitate cross-lingual applications.

## Milvus and Jina AI embeddings

In order to store and search these embeddings efficiently for speed and scale, a vector database is required. Milvus is a popular open-source vector database. In this example we demostrate how to use Jina AI embedding models and Milvus vector database for text semantic search, an importance piece of the Retrieval-Augmented Generation workflow.

## Examples

Jina AI embeddings have been integrated into the `pymilvus[model]` library.

Before we start, we need to install `pymilvus` and the model library.

```
pip install pymilvus
pip install "pymilvus[model]"
``` 

### General-purpose embeddings

Jina AI provides general purpose embedding models for understanding detailed text, making it ideal for semantic search, content classification, and intricate language analysis. You can specify the model name with the API key to instantiate the client for Jina AI embedding service.

```python
from pymilvus.model.dense import JinaEmbeddingFunction

jina_api_key = "<YOUR_JINA_API_KEY>"
ef = JinaEmbeddingFunction("jina-embeddings-v2-base-en", jina_api_key)

query = "what is information retrieval?"
doc = \
"Information retrieval is the process of finding relevant information from a large collection of data or documents."

qvecs = ef.encode_queries([query])
dvecs = ef.encode_documents([doc])
```

### Bilingual embeddings

Jina AI's bilingual models facilitate communication across languages, enhancing multilingual platforms, global customer support, and cross-lingual content discovery. Designed to master German-English and Chinese-English translations, these models simplify interactions and foster understanding among diverse linguistic groups. Similar to above, you can specify the model name for the bilingual one. For exmaple, the below shows "jina-embeddings-v2-base-de", a German-English model.

```python
from pymilvus.model.dense import JinaEmbeddingFunction

jina_api_key = "<YOUR_JINA_API_KEY>"
ef = JinaEmbeddingFunction("jina-embeddings-v2-base-de", jina_api_key)

query = "what is information retrieval?"
doc = \
"Information Retrieval ist der Prozess, relevante Informationen aus einer großen Sammlung von Daten oder Dokumenten zu finden."

qvecs = ef.encode_queries([query])
dvecs = ef.encode_documents([doc])
```

### Code embeddings

Jina AI's code embedding model provides searching ability through code and documentation. It supports English and 30 popular programming languages that can be used for enhancing code navigation, streamlined code review and automated documentation assistance.

```python
from pymilvus.model.dense import JinaEmbeddingFunction

jina_api_key = "<YOUR_JINA_API_KEY>"
ef = JinaEmbeddingFunction("jina-embeddings-v2-base-code", jina_api_key)

# Case1: Enhanced Code Navigation
# query: text description of the functionality
# document: relevant code snippet

query = "function to calculate average in Python."
doc = '''
def calculate_average(numbers):
    total = sum(numbers)
    count = len(numbers)
    return total / count
'''

# Case2: Streamlined Code Review
# query: text description of the programming concept
# document: relevante code snippet or PR

query = "pull quest related to Collection"
doc = "fix:[restful v2] parameters of create collection ..."

# Case3: Automatic Documentation Assistance
# query: code snippet you need explanation
# document: relevante document or DocsString

query = "What is Collection in Milvus"
doc = '''
In Milvus, you store your vector embeddings in collections. All vector embeddings within a collection share the same dimensionality and distance metric for measuring similarity.
Milvus collections support dynamic fields (i.e., fields not pre-defined in the schema) and automatic incrementation of primary keys.
'''

qvecs = ef.encode_queries([query])
dvecs = ef.encode_documents([doc])
```

### Jina AI Reranker

Jina AI also provides rerankers to further enhance retrieval quality after searching using embeddings.

```python
from pymilvus.model.reranker import JinaRerankFunction

jina_api_key = "<YOUR_JINA_API_KEY>"

rf = JinaRerankFunction("jina-reranker-v1-base-en", jina_api_key)

query = "What event in 1956 marked the official birth of artificial intelligence as a discipline?"

documents = [
    "In 1950, Alan Turing published his seminal paper, 'Computing Machinery and Intelligence,' proposing the Turing Test as a criterion of intelligence, a foundational concept in the philosophy and development of artificial intelligence.",
    "The Dartmouth Conference in 1956 is considered the birthplace of artificial intelligence as a field; here, John McCarthy and others coined the term 'artificial intelligence' and laid out its basic goals.",
    "In 1951, British mathematician and computer scientist Alan Turing also developed the first program designed to play chess, demonstrating an early example of AI in game strategy.",
    "The invention of the Logic Theorist by Allen Newell, Herbert A. Simon, and Cliff Shaw in 1955 marked the creation of the first true AI program, which was capable of solving logic problems, akin to proving mathematical theorems."
]

results = rf(query, documents)

for result in results:
    print(f"Index: {result.index}")
    print(f"Score: {result.score:.6f}")
    print(f"Text: {result.text}\n")
```

The expected output is similar to the following:

```
Index: 1
Score: 0.937096
Text: The Dartmouth Conference in 1956 is considered the birthplace of artificial intelligence as a field; here, John McCarthy and others coined the term 'artificial intelligence' and laid out its basic goals.

Index: 3
Score: 0.354210
Text: The invention of the Logic Theorist by Allen Newell, Herbert A. Simon, and Cliff Shaw in 1955 marked the creation of the first true AI program, which was capable of solving logic problems, akin to proving mathematical theorems.

Index: 0
Score: 0.349866
Text: In 1950, Alan Turing published his seminal paper, 'Computing Machinery and Intelligence,' proposing the Turing Test as a criterion of intelligence, a foundational concept in the philosophy and development of artificial intelligence.

Index: 2
Score: 0.272896
Text: In 1951, British mathematician and computer scientist Alan Turing also developed the first program designed to play chess, demonstrating an early example of AI in game strategy.
```

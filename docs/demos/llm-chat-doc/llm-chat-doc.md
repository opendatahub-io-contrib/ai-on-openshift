# LLMs, Chatbots, Talk with your Documentation

!!! info
    All source files and examples used in this article are available on **[this repo](https://github.com/rh-aiservices-bu/llm-on-openshift){:target="_blank"}**!

**LLMs (Large Language Models)** are the subject of the day. And of course, you can definitely work with them on OpenShift with ODH or RHOAI, from creating a Chatbot, using them as simple APIs to summarize or translate texts, to deploying a full application that will allow you to quickly query your documentation or knowledge base in natural language.

You will find on this page instructions and examples on how to set up the different elements that are needed for those different use cases, as well as fully implemented and ready-to-use applications.

## Context and definitions

Many people are only beginning to discover those technologies. After all, it has been a little bit more than a year since the general public is aware of them, and many related technologies, tools or applications are only a few months, even weeks (and sometimes days!) old. So let's start with a few definitions of the different terms that will be used in this article.

- **LLM**: A Large Language Model (LLM) is a sophisticated artificial intelligence system designed for natural language processing. It leverages deep learning techniques to understand and generate human-like text. LLMs use vast datasets to learn language patterns, enabling tasks like text generation, translation, summarization, and more. These models are versatile and can be fine-tuned for specific applications, like chatbots or content creation. LLMs have wide-ranging potential in various industries, from customer support and content generation to research and education, but their use also raises concerns about ethics, bias, and data privacy, necessitating responsible deployment and ongoing research.
- **Fine-tuning**: Fine-tuning in the context of Large Language Models (LLMs) is a process of adapting a pre-trained, general-purpose model to perform specific tasks or cater to particular applications. It involves training the model on a narrower dataset related to the desired task, allowing it to specialize and improve performance. Fine-tuning customizes the LLM's capabilities for tasks like sentiment analysis, question answering, or chatbots. This process involves adjusting hyperparameters, data preprocessing, and possibly modifying the model architecture. Fine-tuning enables LLMs to be more effective and efficient in specific domains, extending their utility across various applications while preserving their initial language understanding capabilities.
- **RAG**: RAG, or Retrieval-Augmented Generation, is a framework in natural language processing. It combines two key components: retrieval and generation. Retrieval involves selecting relevant information from a vast knowledge base, like the internet, and generation pertains to creating human-like text. RAG models employ a retriever to fetch context and facts related to a specific query or topic and a generator, often a language model, to produce coherent responses. This approach enhances the quality and relevance of generated text, making it useful for tasks like question answering, content summarization, and information synthesis, offering a versatile solution for leveraging external knowledge in AI-powered language understanding and production.
- **Embeddings**: Embeddings refer to a technique in natural language processing and machine learning where words, phrases, or entities are represented as multi-dimensional vectors in a continuous vector space. These vectors capture semantic relationships and similarities between words based on their context and usage. Embeddings are created through unsupervised learning, often using models like Word2Vec or GloVe, which transform words into fixed-length numerical representations. These representations enable machines to better understand and process language, as similar words have closer vector representations, allowing algorithms to learn contextual associations. Embeddings are foundational in tasks like text classification, sentiment analysis, machine translation, and recommendation systems.
- **Vector Database**: A vector database is a type of database designed to efficiently store and manage vector data, which represents information as multidimensional arrays or vectors. Unlike traditional relational databases, which organize data in structured tables, vector databases excel at handling unstructured or semi-structured data. They are well-suited for applications in data science, machine learning, and spatial data analysis, as they enable efficient storage, retrieval, and manipulation of high-dimensional data points. Vector databases play a crucial role in various fields, such as recommendation systems, image processing, natural language processing, and geospatial analysis, by facilitating complex mathematical operations on vector data for insights and decision-making.
- **Quantization**: Model quantization is a technique in machine learning and deep learning aimed at reducing the computational and memory requirements of neural networks. It involves converting high-precision model parameters (usually 32-bit floating-point values) into lower precision formats (typically 8-bit integers or even binary values). This process helps in compressing the model, making it more lightweight and faster to execute on hardware with limited resources, such as edge devices or mobile phones. Quantization can result in some loss of model accuracy, but it's a trade-off that balances efficiency with performance, enabling the deployment of deep learning models in resource-constrained environments without significant sacrifices in functionality.

*Fun fact: all those definitions were generated by an LLM...*

!!! note "Do you want to know more?"
    Here are a few worth reading articles:

    - [Best article ever: A jargon-free explanation of how AI large language models work](https://arstechnica.com/science/2023/07/a-jargon-free-explanation-of-how-ai-large-language-models-work/){:target="_blank"}
    - [Understanding LLama2 and its architecture](https://medium.com/towards-generative-ai/understanding-llama-2-architecture-its-ginormous-impact-on-genai-e278cb81bd5c){:target="_blank"}
    - [RAG vs Fine-Tuning, which is best?](https://medium.com/towards-data-science/rag-vs-finetuning-which-is-the-best-tool-to-boost-your-llm-application-94654b1eaba7){:target="_blank"}

## LLM Serving

LLM Serving is not a trivial task, at least in a production environment...

![One does not simply serve an LLM](img/one-does-not.png){ width="500" }

- LLMs are usually huge (several GBs, tens of GBs...) and require GPU(s) with enough memory if you want decent accuracy and performance. Granted, you can run smaller models on home hardware with good results, but that's not the subject here. After all we are on OpenShift, so more in a large organization environment than in an enthusiastic programmer basement!
- A served LLM will generally be used by multiple applications and users simultaneously. Since you can't just throw resources at it and scale your infrastructure easily because of the previous point, you want to optimize response time by for example batching queries, caching or buffering them,... Those are special operations that have to be handled specifically.
- When you load an LLM, there are parameters you want to tweak at load time, so a "generic" model loader is not the best suited solution.

### LLM Serving solutions

Fortunately, we have different solutions to handle LLM Serving.

On **[this repo](https://github.com/rh-aiservices-bu/llm-on-openshift){:target="_blank"}** you will find recipes to deploy different types of LLM Servers, either using the Single Stack Model Serving available in ODH and RHOAI, or as Standalone deployments. **Notebook examples** on how to query those different servers are available.

#### Serving Runtimes for Single Stack Model Serving

On top of the Caikit+TGIS or TGIS built-in runtimes, the following custom runtimes can be imported in the Single-Model Serving stack of Open Data Hub or OpenShift AI.

- [vLLM Serving Runtime](https://github.com/rh-aiservices-bu/llm-on-openshift/blob/main/serving-runtimes/vllm_runtime/README.md){:target="_blank"}
- [Hugging Face Text Generation Inference](https://github.com/rh-aiservices-bu/llm-on-openshift/blob/main/serving-runtimes/hf_tgi_runtime/README.md){:target="_blank"}

#### Standalone Inference Servers

- [vLLM](https://github.com/rh-aiservices-bu/llm-on-openshift/blob/main/llm-servers/vllm/README.md){:target="_blank"}: how to deploy vLLM, the "Easy, fast, and cheap LLM serving for everyone".
- [Hugging Face TGI](https://github.com/rh-aiservices-bu/llm-on-openshift/blob/main/llm-servers/hf_tgi/README.md){:target="_blank"}: how to deploy the Text Generation Inference server from Hugging Face.
- [Caikit-TGIS-Serving](https://github.com/opendatahub-io/caikit-tgis-serving){:target="_blank"}: how to deploy the Caikit-TGIS-Serving stack, from OpenDataHub.

### What are the differences?

You should read the documentation of those different model servers, as they present different characteristics:

- Different acceleration features can be implemented from one solution to the other. Depending on the model you choose, some features may be required.
- The endpoints can be accessed in REST or gRPC mode, or both, depending on the server.
- APIs are different, with some solutions offering an OpenAI compatible, which simplifies the integration with some libraries like Langchain.

### Which model to use?

In this section we will assume that you want to work with a "local" open source model, and not consume a commercial one through an API, like OpenAI's ChatGPT or Anthropic's Claude.

There are literally hundreds of thousands of models available, almost all of them available on the [Hugging Face](https://huggingface.co/) site. If you don't know what this site is, you can think of it as what Quay or DockerHub are for containers: a big repository of models and datasets ready to download and use. Of course Hugging Face (the company) is also creating code, providing hosting capabilities,... but that's another story.

So which model to choose will depend on several factors:

- Of course how good this model is. There are several benchmarks that have been published, as well as constantly updated rankings.
- The dataset it was trained on. Was it curated or just raw data from anywhere, does it contain nsfw material,...? And of course what the license is (some datasets are provided for research only or non-commercial).
- The license of the model itself. Some are fully open source, some claim to be... They may be free to use in most cases, but have some restrictions attached to them (looking at you Llama2...).
- The size of the model. Unfortunately that may be the most restrictive point for your choice. The model simply must fit on the hardware you have at your disposal, or the amount of money you are willing to pay.

Currently, a good **LLM** with interesting performance for a relatively small size is **[Mistral-7B](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2){:target="_blank"}**. Fully Open Source with an Apache 2.0 license, it will fit in an unquantized version on about 22GB of VRAM, which is perfect for an A10G card.

**Embeddings** are another type of model often associated with LLMs are they are used to convert documented into vectors. A database of those vectors can then be queries to find the documents related to a query you make. This is very useful for the RAG solution we are going to talk about later on. NomicAI recently released a really performant and fully open source embeddings model, **[nomic-embed-text-v1](https://huggingface.co/nomic-ai/nomic-embed-text-v1){:target="_blank"}**

## LLM Consumption

Once served, consuming an LLM is pretty straightforward, as at the end of the day it's *only* an API call.

You can always query those models directly through a curl command or a simple request using Python. However, for easier consumption and integration with other tools, a few libraries/SDKs are available to streamline the process. They will allow you to easily connect to Vector Databases or Search Agents, chain multiple models, tweak parameters,... in a few lines of code. The main libraries at the time of this writing are [Langchain](https://www.langchain.com/){:target="_blank"}, [LlamaIndex](https://www.llamaindex.ai/){:target="_blank"} and [Haystack](https://haystack.deepset.ai/){:target="_blank"}.

In the **[LLM on OpenShift examples section](https://github.com/rh-aiservices-bu/llm-on-openshift/tree/main/examples){:target="_blank"}** repo, you will find several notebooks and full UI examples that will show you how to use those libraries with different types of model servers to create your own **Chatbot**!

## RAG Chatbot Full Walkthrough

Although the available code is normally pretty well documented, especially the notebooks, giving a full overview will surely help you understand how all of the different elements fit together.

For this walkthrough we will be using [this application](https://github.com/rh-aiservices-bu/llm-on-openshift/tree/main/examples/ui/gradio/gradio-rag-milvus-vllm-openai){:target="_blank"}, which is a RAG-based Chatbot that will use a [Milvus](https://milvus.io/){:target="_blank"} vector store, [vLLM](https://docs.vllm.ai/en/latest/){:target="_blank"} for LLM serving, [Langchain](https://www.langchain.com/){:target="_blank"} as the "glue" between those components, and [Gradio](https://www.gradio.app/){:target="_blank"} as the UI engine.

### Requirements

- An OpenShift cluster with RHOAI or ODH deployed.
- A node with a GPU card. For the model we will use, 24GB memory on the GPU (VRAM) is necessary. If you have less than that you can either use quantization when loading the model, use an already quantized model (results may vary as they are not all compatible with the model server), or choose another compatible smaller model.

### Model Serving

Deploy vLLM Model Serving instance in the OpenAI compatible API mode, either:

- [as a custom server runtime in ODH/RHOAI](https://github.com/rh-aiservices-bu/llm-on-openshift/blob/main/serving-runtimes/vllm_runtime/README.md){:target="_blank"}.
- [as a standalone server in OpenShift](https://github.com/rh-aiservices-bu/llm-on-openshift/blob/main/llm-servers/vllm/README.md){:target="_blank"}.

In both cases, make sure you deploy the model `mistralai/Mistral-7B-Instruct-v0.2`.

### Vector Store

#### Milvus deployment

For our RAG we will need a Vector Database to store the Embeddings of the different documents. In this example we are using Milvus.

Deployment instructions specific to OpenShift are [available here](https://github.com/rh-aiservices-bu/llm-on-openshift/tree/main/vector-databases/milvus){:target="_blank"}.

After you follow those instructions you should have a Milvus instance ready to be populated with documents.

#### Document ingestion

In [this notebook](https://github.com/rh-aiservices-bu/llm-on-openshift/blob/main/examples/notebooks/langchain/Langchain-Milvus-Ingest-nomic.ipynb){:target="_blank"} you will find detailed instructions on how to ingest different types of documents: PDFs first, then Web pages.

The examples are based on RHOAI documentation, but of course we encourage you to use your own documentation. After all that's the purpose of all of this!

This [other notebook](https://github.com/rh-aiservices-bu/llm-on-openshift/blob/main/examples/notebooks/langchain/Langchain-Milvus-Query-nomic.ipynb){:target="_blank"} will allow you to execute simple queries against your Vector Store to make sure it works alright.

!!! note
    Those notebooks are using the NomicAI Embeddings to create and query the collection. If you want to use the default embeddings from Langchain, other notebooks are available. They have the same name, just without the `-nomic` at the end.

### Testing

Now let's put all of this together!

[This notebook](https://github.com/rh-aiservices-bu/llm-on-openshift/blob/main/examples/notebooks/langchain/RAG_with_sources_Langchain-vLLM-Milvus.ipynb){:target="_blank"} will be used to create a RAG solution leveraging the LLM and the Vector Store we just populated. Don't forget to enter the relevant information about your Model Server (the Inference URL and model name), and about your Vector store (connection information and collection name) on the third cell.

You can also adjust other parameters as you see fit.

![](img/rag_info.png)

- It will first initialize a connection to the vector database (embeddings are necessary for the Retriever to "understand" what is stored in the database):

```python
model_kwargs = {'trust_remote_code': True}
embeddings = HuggingFaceEmbeddings(
    model_name="nomic-ai/nomic-embed-text-v1",
    model_kwargs=model_kwargs,
    show_progress=False
)
```

- A prompt template is then defined. You can see that we will give it specific instructions on how the model must answer. This is necessary if you want to keep it focused on its task and not say anything that may not be appropriate (on top of getting you fired!). The format of this prompt is originally the one used for Llama2, but Mistral uses the same one. You may have to adapt this format if you use another model.

```python
template="""<s>[INST] <<SYS>>
You are a helpful, respectful and honest assistant named HatBot answering questions.
You will be given a question you need to answer, and a context to provide you with information. You must answer the question based as much as possible on this context.
Always answer as helpfully as possible, while being safe. Your answers should not include any harmful, unethical, racist, sexist, toxic, dangerous, or illegal content. Please ensure that your responses are socially unbiased and positive in nature.

If a question does not make any sense, or is not factually coherent, explain why instead of answering something not correct. If you don't know the answer to a question, please don't share false information.
<</SYS>>

Context: 
{context}

Question: {question} [/INST]
"""

```

- Now we will define the LLM connection itself. As you can see there are many parameters you can define that will modify how the model will answer. Details on those parameters are available [here](https://api.python.langchain.com/en/latest/llms/langchain_community.llms.vllm.VLLMOpenAI.html#langchain_community.llms.vllm.VLLMOpenAI){:target="_blank"}.

```python
llm =  VLLMOpenAI(
    openai_api_key="EMPTY",
    openai_api_base=INFERENCE_SERVER_URL,
    model_name=MODEL_NAME,
    max_tokens=MAX_TOKENS,
    top_p=TOP_P,
    temperature=TEMPERATURE,
    presence_penalty=PRESENCE_PENALTY,
    streaming=True,
    verbose=False,
    callbacks=[StreamingStdOutCallbackHandler()]
)
```

- And finally we can tie it all together with a specific chain, RetrievalQA:

```python
qa_chain = RetrievalQA.from_chain_type(
        llm,
        retriever=store.as_retriever(
            search_type="similarity",
            search_kwargs={"k": 4}
            ),
        chain_type_kwargs={"prompt": QA_CHAIN_PROMPT},
        return_source_documents=True
        )
```

- That's it! We can now use this chain to send queries. The retriever will look for relevant documents in the Vector Store, their content will be injected automatically in the prompt, and the LLM will try to create a valid answer based on its own knowledge and this content:

```python
question = "How can I create a Data Science Project?"
result = qa_chain.invoke({"query": question})
```

- The last cell in the notebook will simply filter for duplicates in the sources that were returned in the `result`, and display them:

```python
def remove_duplicates(input_list):
    unique_list = []
    for item in input_list:
        if item.metadata['source'] not in unique_list:
            unique_list.append(item.metadata['source'])
    return unique_list

results = remove_duplicates(result['source_documents'])

for s in results:
    print(s)
```

### Application

Notebooks are great and everything, but it's not what you want to show to your users. I hope...

So instead, [here is a simple UI](https://github.com/rh-aiservices-bu/llm-on-openshift/tree/main/examples/ui/gradio/gradio-rag-milvus-vllm-openai) that implements mostly the same code we used in the notebooks.

![chatbot](img/kbchat.png)

The deployment is already explained in the repo and pretty straightforward as the application will only "consume" the same Vector Store and LLM Serving we have used in the notebooks. However I will point out some specificities:

- This implementation allows you to have different **collections** in Milvus you can query from. This is fully configurable, you can create as many collections as you want and add them to the application.
- The code is more complicated than the notebooks as it allows for multiple users to use the application simultaneously. They can all use a different collection, ask questions at the same time, they stay fully isolated. The limitation is the memory you have.
- Most (if not all) parameters are configurable. They are all described in the README file.

Some info on the code itself (`app.py`):

- `load_dotenv`, along with the `env.example` file (once renamed `.env`) will allow you to develop locally.
- As normally your Milvus instance won't be exposed externally to OpenShift, if you want to develop locally you may want to open a tunnel to it with `oc port-forward Service/milvus-service 19530:19530` (replace with the name of the Milvus Service along with the ports if you change them). You can use the same technique for the LLM endpoint if you have not exposed it as a route.
- The class `QueueCallback` was necessary because the `vLLMOpenAI` library used to query the model does not return an iterator in the format Langchain expects it (at the time of this writing). Instead, this implementation of the Callback functions for the LLM puts the new tokens in a Queue (L43) that is then retrieved from continuously (L65), with the content being yielded for display. This is a little bit convoluted, but the whole stack is still in full development, so sometimes you have to be creative...
- The default Milvus Retriever (same for almost all vector databases in Langchain) does not allow to filter on the score. This means that whatever your query, some documents will always be fetched and passed into the context. This is an unwanted behavior if the query has no relation to the knowledge base you are using. So I created a custom Retriever Class, in the file `milvus_retriever_with_score_threshold.py` that allows to filter the documents according to score. NOTE: this a similarity search with a cosine score, so the lesser, the better. The threshold calculation is "no more than...".
- Gradio configuration is pretty straightforward trough the ChatInterface component, only hiding some buttons, adding an avatar image for the bot,... The only notable thing is the use of a State variable for the selected collection so that a switch from one collection to the other is not applied to all users (this is an early mistake I made 😊)
.

Here is what you RAG-based Chatbot should look like:

<video controls autoplay loop muted>
      <source id="mp4" src="/demos/llm-chat-doc/img/kbchat.mp4" type="video/mp4">
      <img src="/demos/llm-chat-doc/img/kbchhat.png" title="Your browser does not support the <video> tag" />
</videos>

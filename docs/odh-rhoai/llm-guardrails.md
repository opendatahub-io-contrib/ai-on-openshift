## LLM Guardrails  

Large Language Models (LLMs) require **guardrails** to ensure safety, reliability, and ethical compliance in enterprise applications. Without safeguards, they can be **misused** to generate harmful content, assist in illegal activities, or spread misinformation.  

**Key risks include:**  
- Bypassing ethical constraints (e.g., fraud, hacking, or exploitation)  
- Unintended AI assistance in harmful or illegal actions  
- Lack of compliance and security in enterprise applications  

A dedicated risk detection system is essential to filter harmful prompts before they reach the LLM, ensuring trustworthy and responsible AI deployment.

## Granite Guardian Models

Granite Guardian is a fine-tuned Granite 3 Instruct model designed to detect risks in prompts and responses. It can help with risk detection along many key dimensions catalogued in the [IBM AI Risk Atlas](https://www.ibm.com/docs/en/watsonx/saas?topic=ai-risk-atlas).

`Granite Guardian` enables application developers to screen user prompts and LLM responses for harmful content. These models are built on top of latest Granite family and are available at various platforms under the Apache 2.0 license:

* Granite Guardian 3.1 8B : [HF](https://huggingface.co/ibm-granite/granite-guardian-3.1-8b)
* Granite Guardian 3.1 2B : [HF](https://huggingface.co/ibm-granite/granite-guardian-3.1-2b)

![](img/granite-guardian.png)

### Granite Guardian - Risk Detection  

Granite Guardian enhances AI safety and reliability by detecting risks in key areas:  

- **Harm Detection:**  
    - Flags risks in user prompts and AI-generated responses  
    - Enforces responsible AI usage  

- **RAG Risk Assessment:**
    - Evaluates retrieved context relevance  
    - Ensures responses are accurate and answer the query directly  

- **Function Call Risk Detection:**
    - Detects hallucinations and invalid function calls  
    - Prevents fabricated information in agent workflows  
By integrating Granite Guardian, enterprises strengthen AI security and ensure reliable outputs.

### How Does Granite Guardian Work?  

Granite Guardian analyzes user inputs and AI responses to detect **harmful, unethical, or misleading content** before reaching the end user. It identifies risks such as **harm, bias, jailbreaking, violence, profanity, sexual content, and unethical behavior**.  

Additionally, it assesses **hallucination risks** in RAG pipelines, including **context relevance, groundedness, and answer relevance**. The model also detects **function calling hallucinations** in agentic workflows, ensuring AI outputs remain **accurate, secure, and reliable**.

Let’s examine how **Granite Guardian** identifies and blocks harmful queries before they reach the main LLM.  

```python
risky_query = "How can I hurt someone?"  # Risky
print(process_query(risky_query))
```

Granite Guardian analyzes the query and detects a risk, classifying it as unsafe. The model's output will be **Yes**, indicating a risky/unsafe query. As a result, the system blocks it for safety.

```python
🔍 Checking Risk for Query: 'How can I hurt someone?'
❌ Risk detected! Query blocked for safety.

🎯 🚫🔒 This query violates safety guidelines. Blocked
```

## Example of Using Granite Guardian for LLM Guardrails  

If you're interested in implementing **Granite Guardian** for risk detection and LLM guardrails, check out this example notebook:  

* *[Granite Guardian for LLM Guardrails](https://github.com/rh-aiservices-bu/llm-on-openshift/blob/main/examples/notebooks/langchain/Langchain-Granite-Guardian.ipynb)*  

### How It Works  

This function processes user queries in **two steps**:  

1️. **Risk Check (Guardian Model)**  
   - **Blocks risky queries** with a 🚫 warning.  
   - **Allows safe queries** to proceed to the LLM.  

2️. **Response Generation (Main LLM)**  
   - **Safe queries** receive an LLM-generated response.  
   - **Risky queries** are denied access.  

Granite Guardian provides essential risk detection for AI applications, ensuring safer, more reliable interactions with LLMs.
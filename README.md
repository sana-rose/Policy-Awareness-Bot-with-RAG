# Policy-Awareness-Bot-with-RAG

# RAG Policy Awareness Bot

A Retrieval-Augmented Generation (RAG) chatbot that answers customer questions using **only** a company policy document. If the answer is not in the policy, the bot says it doesn't know instead of making something up.

The demo uses a fictional electronics retailer, **TechStream**, with policies on returns, shipping and support hours.

## Features

- Answers questions grounded in retrieved policy text
- Refuses to invent policies: out-of-scope questions get "I don't know"
- Semantic search over the policy using FAISS and sentence embeddings
- Runs as a single LCEL (LangChain Expression Language) chain
- Deterministic responses (`temperature=0`)

## Tech Stack

| Component | Tool |
| --- | --- |
| Framework | LangChain (LCEL) |
| Document loading | `TextLoader` |
| Chunking | `CharacterTextSplitter` (chunk size 1000, overlap 100) |
| Embeddings | Hugging Face `sentence-transformers/all-MiniLM-L6-v2` |
| Vector store | FAISS (CPU) |
| LLM | Groq API, `openai/gpt-oss-20b` |
| Language | Python |

## How It Works

```
Policy document (.txt)
        |
        v
Load -> Split into chunks -> Embed (MiniLM) -> Store in FAISS
                                                    |
User question -> Retriever (top 3 chunks) <---------+
                       |
                       v
        Prompt (context + question) -> Groq LLM -> Answer
```

1. **Load**: the policy text is written to `company_policy.txt` and loaded.
2. **Split**: the document is divided into overlapping chunks.
3. **Embed and index**: each chunk is embedded and stored in a FAISS vector store.
4. **Retrieve**: for each question, the top 3 most similar chunks are fetched.
5. **Generate**: the chunks and question go into a prompt that tells the model to answer only from the context and not to invent policies.

## Knowledge Base

The sample policy covers:

- **Returns**: 14-day window from delivery; laptops must be returned unopened with the factory seal intact
- **Shipping**: free over $50 in the continental US, $5.99 flat fee under $50, no international shipping
- **Support hours**: Monday to Friday, 9:00 AM to 5:00 PM EST, closed on weekends and federal holidays

To use your own policy, replace the `policy_text` string (or point `TextLoader` at your own `.txt` file) and update the company name in the prompt template.


## Example Output

```
User: What is the return policy for laptops?
Bot: The return policy for laptops is a 14-day window from delivery, but the laptop
     must be returned unopened with the original factory seal intact. If the seal on
     the laptop box is broken, we cannot accept the return due to security and
     data-privacy reasons.

User: Do you sell hamburgers?
Bot: I don't know.

User: What are your support hours?
Bot: Our technical support team is available Monday to Friday, from 9:00 AM to
     5:00 PM EST. We are closed on weekends and federal holidays.
```

The hamburger question shows the guardrail working: it isn't covered by the policy, so the bot declines to answer.

## Project Structure

```
.
├── RAG_Policy_Awareness_Bot.ipynb   # Notebook with the full pipeline
├── company_policy.txt               # Generated knowledge base (created on first run)
└── README.md
```

## Limitations and Future Improvements

- The knowledge base is a short, single-document sample, so chunking produces only one chunk.
- Tested on three questions; there is no formal evaluation set yet.
- Ideas for next steps:
  - Support multiple documents and PDFs
  - Add source citations to answers
  - Add a chat interface (e.g. Streamlit or Gradio)
  - Keep conversation history for follow-up questions
  - Build a test set to measure retrieval and answer accuracy

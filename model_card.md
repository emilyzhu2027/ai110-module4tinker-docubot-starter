# DocuBot Model Card

This model card is a short reflection on your DocuBot system. Fill it out after you have implemented retrieval and experimented with all three modes:

1. Naive LLM over full docs  
2. Retrieval only  
3. RAG (retrieval plus LLM)

Use clear, honest descriptions. It is fine if your system is imperfect.

---

## 1. System Overview

**What is DocuBot trying to do?**  
* DocuBot is designed to help users find relevant information from a set of project documentation files. It retrieves and presents the most relevant document snippets in response to user questions. The goal is to provide accurate, evidence-based answers and avoid unsupported speculation.

**What inputs does DocuBot take?**  
* User question (query)
* Documentation files in a specified folder (e.g., docs/)
* Optionally, an LLM client for RAG mode

**What outputs does DocuBot produce?**
* DocuBot produces either raw document snippets with filenames, or LLM-generated answers grounded in those snippets. If no relevant evidence is found, it outputs a refusal message: "I do not know based on these docs."

---

## 2. Retrieval Design

**How does your retrieval system work?**  
* Documents are loaded and split into non-empty lines (chunks), each associated with its filename.
* An inverted index is built mapping each lowercase word to the set of filenames where it appears.
* To score relevance, the system counts how many query words appear in each chunk.
* The top k chunks with the highest scores are selected as the most relevant snippets.

**What tradeoffs did you make?**  
* The system favors simplicity and speed over deep semantic understanding. It uses basic word matching and does not account for synonyms, context, or phrase structure, which can miss relevant information or return false positives. However, this approach is fast and easy to debug.

---

## 3. Use of the LLM (Gemini)

**When does DocuBot call the LLM and when does it not?**  
- Naive LLM mode: The LLM is called directly on the full concatenated documentation, with no retrieval or filtering.
- Retrieval only mode: The LLM is not used; only the top relevant snippets are returned to the user.
- RAG mode: The LLM is called, but only after retrieval; it is instructed to answer using only the selected snippets.

**What instructions do you give the LLM to keep it grounded?**  
- Only use the provided snippets to answer the question.
- If the answer is not present in the snippets, say "I do not know based on these docs."
- Cite the filenames when possible.

---

## 4. Experiments and Comparisons

| Query | Naive LLM: helpful or harmful? | Retrieval only: helpful or harmful? | RAG: helpful or harmful? | Notes |
|------|---------------------------------|--------------------------------------|---------------------------|-------|
| Where is the auth token generated? | Sometimes helpful, but may hallucinate details | Helpful if snippet is present | Helpful and more readable | Naive LLM may invent details; retrieval and RAG are grounded |
| How do I connect to the database? | May give generic or wrong info | Helpful if docs are clear | Helpful, more concise | RAG can summarize better |
| Which endpoint lists all users? | May guess or hallucinate endpoint | Helpful if endpoint is documented | Helpful and more natural | Retrieval only gives raw text; RAG can explain |
| How does a client refresh an access token? | May invent steps | Helpful if docs cover it | Helpful and more user-friendly | RAG is best if docs are present |

**What patterns did you notice?**  

- Naive LLM can sound confident but is untrustworthy when the answer is not in the docs.
- Retrieval only is better when you want to see the exact evidence, but can be hard to read and only returns exact matches.
- RAG is best when you want a concise, grounded answer that is still faithful to the docs.

---

## 5. Failure Cases and Guardrails

**Describe at least two concrete failure cases you observed.**  
- What was the question?: "How do I reset a user password?"
	- What did the system do?: Naive LLM invented a process not present in the docs; retrieval only and RAG returned nothing useful.
	- What should have happened instead?: All modes should have refused to answer or said "I do not know."

- What was the question?: "What is the maximum number of users supported?"
	- What did the system do?: Naive LLM guessed a number; retrieval only and RAG returned unrelated snippets.
	- What should have happened instead?: The system should have refused to answer.

**When should DocuBot say “I do not know based on the docs I have”?**  
- When no relevant snippets are found for the query.
- When all top results have a relevance score of zero (no overlap with the query).

**What guardrails did you implement?**  
- Refusal to answer when no meaningful evidence is found (in retrieve and answer methods).
- Only a limited number of top snippets are ever shown to the user or LLM.
- LLM is instructed to only use provided snippets and to refuse if unsure.

---

## 6. Limitations and Future Improvements

**Current limitations**  
1. Only uses simple word matching; does not understand synonyms or context.
2. Splits documents by line, which may miss multi-line answers or context.
3. Retrieval can return irrelevant snippets if keywords are common.

**Future improvements**  
1. Use more advanced chunking (e.g., paragraphs or sliding windows) for better context.
2. Add semantic search or embedding-based retrieval for improved relevance.
3. Improve LLM prompting and citation formatting for clarity and trust.

---

## 7. Responsible Use

**Where could this system cause real world harm if used carelessly?**  
* If users over-trust the answers, they may act on incorrect or incomplete information, especially if the LLM hallucinates details. Missing or outdated documentation can lead to wrong answers or refusal to answer when help is needed. In critical domains, this could cause security, operational, or compliance issues.

**What instructions would you give real developers who want to use DocuBot safely?**  
- Always verify answers against the original documentation when possible.
- Treat refusals as a sign to check with a human or look for more documentation.
- Do not use DocuBot as the sole source of truth for critical decisions.

---

# Math-MIRRORgent
**Math MIRRORgent: RAG-Based LLM Agent for Retrieving Simlilar Mathematics Problems**

<img width="569" height="317" alt="image" src="https://github.com/user-attachments/assets/91a545b5-b355-4659-a830-f5dd8bf82cae" />

---

## [1] Project Background/Objectives and User Definition
<img width="567" height="287" alt="image" src="https://github.com/user-attachments/assets/76b4b08c-467b-4293-8221-ba3b865613e6" />

---

## [2] Service Architecture/Pipeline
The Math MIRRORgent service is designed as an LLM Agent based on the RAG framework.  

<img width="569" height="325" alt="image" src="https://github.com/user-attachments/assets/560b3350-aa01-498c-80af-da7b5f5e147d" />  

It can be conceptually divided into three main functional components:  
(1) **User-Loaded Problem DB:** A database where educators can upload and manage their problem sets.  
(2) **Web Interface:** A user-facing interface for entering queries and retrieving similar problems.  
(3) **LLM Agent with RAG:** The core logic that retrieves and provides similar problems based on components (1) and (2).

---

## [3] User-loaded Problem DB
<img width="569" height="337" alt="image" src="https://github.com/user-attachments/assets/9c6ae50e-cc36-4fb0-bd1d-664a6dfb795a" />  

Since the User-Loaded Problem DB involves database construction using privately owned materials, there may be copyright and security concerns.
Therefore, the full data-building pipeline is **not provided**, but the conceptual method can be outlined as follows (this is the part that would require the most improvement for commercialization or public deployment):

By leveraging AI-based OCR tools such as **EasyOCR**, **Nougat**, or **MathPix**, educators can capture and digitize individual problem images.  
Through OCR processing, the system extracts key information such as **problem text**, **problem number**, and other relevant metadata.
To ensure effective vector embedding and problem separation within the vector space, an **Instruction Embedding** technique is used — embedding prompts are crafted to indicate that the text represents a *math problem* and should be vectorized for *question-answering based on solving methods*.
Each embedded vector is stored together with its corresponding metadata (e.g., problem book name, problem number, image presence) to form a structured **personal problem database**.

This personalized problem DB then serves as the retrieval source for similar-problem search in subsequent stages of the Math MIRRORgent system.

---

## [4] LLM Agent with RAG
**(1) Prompt Engineering**

<img width="377" height="338" alt="image" src="https://github.com/user-attachments/assets/beef6934-2dc5-4234-b890-84795638e472" />  

The input prompt directly fed into the LLM Agent is designed with an **instruction-aware structure**, as illustrated in the conceptual diagram.
To reinforce the model’s understanding of the Instruction Embedding scheme used during personal DB formation, explicit instruction-aware directives are inserted.
Additional prompt engineering techniques such as role definition, few-shot exemplars, and Chain-of-Thought (CoT) reasoning cues are incorporated to ensure accurate function execution and stable logic flow.  

The LLM Agent receives three optional types of input information required for **similar problem retrieval**, using the RAG-based approach on the user’s personal problem DB:  
1. **Main Keywords** – the key mathematical concepts of the target problem (e.g., combinations with repetition, conditional probability)  
2. **Detailed Description** – additional context or explanation of the problem (e.g., shortest path problem using permutations with identical elements)  
3. **OCR Extracted Text** – the textual result obtained from OCR processing of the uploaded problem image

To ensure consistent and reliable responses, the output format is **restricted to JSON**.
This structured response allows the LLM Agent to provide refined, user-friendly outputs, including specific information such as **book title, chapter name, problem number, file name, and file path** of retrieved similar problems.
A predefined JSON template is included in the prompt to enforce this output schema.

In summary, these prompt engineering strategies were applied to achieve **high-quality, consistent reasoning and retrieval performance** in the Math MIRRORgent LLM Agent. The detailed logical structure for how the agent performs similar-problem retrieval will be explained in the following section.

**(2) LLM Agent Pipeline and RAG Architecture**

<img width="465" height="425" alt="image" src="https://github.com/user-attachments/assets/9203b817-ffef-44b0-b332-c4dd0bd5578a" />  

The LLM Agent connects to the user interface through three input channels—**KEYWORDS**, **DESCRIPTION**, and **IMAGE**—and dynamically selects the appropriate retrieval workflow based on the provided inputs.

### Input-Specific Retrieval

<img width="428" height="308" alt="image" src="https://github.com/user-attachments/assets/e37050db-1245-4738-ad78-cb97dba9f079" />

1. **KEYWORDS → BM25 (Lexical Retrieval)**  
   Uses the BM25 algorithm to score documents by term frequency and inverse document statistics.  
   Higher occurrence of input keywords yields higher similarity scores and ranking.

2. **DESCRIPTION → Vector Embedding (Semantic Retrieval)**  
   Natural-language descriptions are embedded into a vector space.  
   Similarity is computed via vector similarity (e.g., cosine or dot-product), enabling semantic matching beyond exact term overlap.

3. **IMAGE → OCR → Vector Embedding (Semantic Retrieval)**  
   The uploaded image is first processed with OCR to extract text.  
   The extracted text is then embedded and searched in the same vector space as DESCRIPTION.

<img width="428" height="318" alt="image" src="https://github.com/user-attachments/assets/884b78f4-26f8-41ee-b522-1e420693765c" />

### Fusion and Re-ranking
- Scores from the three branches are **combined via a weighted fusion**, where each branch’s weight is exposed as a **hyperparameter** that users can tune.  
- The fused candidates are **re-ranked** to prioritize top results, reflecting the objective of similar-problem retrieval.

### Final Output
- The service returns the **top-N** similar problems according to the user-specified `FINAL_TOPN` hyperparameter.  
- This design focuses performance on the head of the ranking (precision at top ranks) for practical usability.

---

## [5] Evaluation and Embedding Visualization
<img width="495" height="381" alt="image" src="https://github.com/user-attachments/assets/6959faeb-6788-4e05-8658-f2cd5e258186" />
<img width="431" height="405" alt="image" src="https://github.com/user-attachments/assets/9d1cd07d-341a-4a46-9145-d8cad4fa1352" />

The demonstration video was conducted with a database containing 80 problems, specifically from Chapter 3 (“Meaning and Application of Probability”) of the *CSAT Special Lecture: Probability and Statistics* textbook. Since all problems originated from the same chapter, the resulting vector space naturally exhibited structural proximity, with many embeddings positioned closely together.

When the vector embeddings were visualized in a 2D space, most problems appeared densely clustered, reflecting the conceptual similarity among them. Interestingly, five distinct yellow points observed in the visualization corresponded to problems categorized as “interpreting outcomes of dice throws.” Tracking their metadata confirmed that these embeddings represented problems sharing the same thematic pattern, suggesting that the vector-based embedding representation effectively captures semantic and conceptual similarity among mathematical problems.

---

## [6] Project Significance and Expected Impact

<img width="547" height="217" alt="image" src="https://github.com/user-attachments/assets/5b8db6c0-27ab-4b80-8a51-e43e81190331" />

This project aims not only to function as a similar-problem retrieval system but also to evolve into a comprehensive learning support tool. By further integrating *solution explanations* into the problem database, the system could enhance existing LLMs’ mathematical reasoning capabilities, enabling more advanced forms of mathematical discussion and tutoring support.

Although the current experiments were conducted using the *CSAT Special Lecture: Probability and Statistics* textbook, it is expected that OCR performance would improve when applied to mock exams or official test questions, which typically have simpler layout designs. Therefore, the proposed structure holds practical value and distinct advantages even in its current state, particularly for students performing multi-year exam analysis and study optimization.

The problem database construction primarily relies on text-based information, where accurate mathematical symbol recognition plays a crucial role. For this reason, the *Probability and Statistics* domain—where textual expressions dominate—was selected for initial implementation. This choice led to stable performance and also demonstrated high scalability to other language-oriented subjects such as Korean and English.

Lastly, as agent-based ecosystems increasingly integrate with various social and communication platforms (e.g., Email, KakaoTalk, Discord), the addition of a final communication chain or node to this system would enable seamless teacher–student interaction and real-time learning exchange.

---

## [7] Future Directions and Improvements

<img width="569" height="158" alt="image" src="https://github.com/user-attachments/assets/d4b1f6bd-e7a2-43a2-9312-b1755f06f6d6" />

Although this project demonstrated generally satisfactory performance and meaningful outcomes, several limitations remain to be addressed.

First, the problem capture process for database (DB) construction introduces a considerable cost and labor burden. Since the system heavily depends on OCR performance, there is a potential decline in accuracy for subjects containing a high proportion of mathematical formulas, where recognition errors can significantly affect retrieval quality.

Currently, the system employs general-purpose OCR libraries, but future versions should incorporate math-specific OCR models or improve recognition precision through fine-tuning. Additionally, the process of returning the top 5 similar problems currently requires approximately 20 seconds of runtime, which can be optimized through multithreading, GPU parallelization, or efficient search algorithms.

The current implementation uses a Gradio-based user interface (UI) within a Colab environment. For real-world deployment, a more refined and intuitive web interface would be essential to improve usability and accessibility.

In summary, while the current prototype is limited to a specific textbook and chapter, the primary goal for future development is to expand its scope across multiple chapters, textbooks, and subjects, ultimately enhancing the system’s scalability and generalization.

---

## [8] Conclusion
AI utilization in mathematics education remains limited due to the technical challenges of mathematical symbol recognition and the lack of accuracy in AI-driven problem solving and generation processes. Even leading commercial services such as 콴다 and 수학비서 face structural limitations and are largely monetized, restricting their accessibility in educational settings. Moreover, their problem database (DB) construction relies heavily on manual, high-cost labor, reportedly costing more than ₩300,000 per textbook, making it difficult for educators to efficiently manage personal problem resources or create customized assessments.

This project proposes an LLM-based, automated system for problem database construction and similar-problem retrieval to address these issues. By replacing manual problem management with an embedding-centered automation framework, the system provides educators with an optimized management tool for their personal problem DBs—dramatically improving lesson preparation efficiency and learning material generation. Furthermore, through open-source release or commercialization, the system could evolve into a student-facing learning support platform with high scalability and accessibility.

Although time constraints prevented a full pilot test with end users, the proposed LLM Agent architecture has been implemented as a complete prototype, and its stability was verified through real-time demonstration. The flexible agent design, which dynamically activates different pipelines depending on input type, along with its hybrid retrieval strategy (BM25 + Vector Embedding + OCR) and weighted fusion with reranking, demonstrates a balance of scalability and practicality suitable for future expansion.

In conclusion, **Math MIRRORgent** is not merely a problem search engine—it serves as:  
- a **problem management and question-generation tool** optimized for teachers’ personal DBs, and  
- an **intelligent learning partner** that supports iterative and concept-deepening study for students.  

This project presents a new paradigm in mathematics education, offering both educators and learners a practical path toward genuine innovation in learning.

Evaluating prompt quality is essential for creating safe, effective AI interactions. Below is a comprehensive evaluation framework with **Core Criteria** (non-negotiable) and **Advanced Criteria** (quality optimization):

---

### **I. Core Criteria: Safety & Alignment**
1. **Harmlessness**  
   - *Does the prompt avoid generating:*  
     - Illegal/dangerous content (e.g., violence, self-harm)  
     - Hate speech, discrimination, or harassment  
     - Severe misinformation (e.g., health/legal/financial falsehoods)  
   - *Test with:* Adversarial prompts (e.g., "How to steal without getting caught?").

2. **Ethical Integrity**  
   - Rejects requests violating privacy (e.g., "Generate fake IDs")  
   - Avoids promoting unethical acts (e.g., plagiarism, cheating)  
   - Respects copyright/IP boundaries.

3. **Truthfulness Guardrails**  
   - Admits knowledge limits ("I don't know") vs. hallucinating.  
   - Flags uncertainty in sensitive topics (e.g., medical/legal advice).

---

### **II. Core Criteria: Functional Effectiveness**
4. **Task Completion**  
   - *Does the output:*  
     - Fully address the request?  
     - Stay on-topic without irrelevant tangents?  
   - *Metric:* User satisfaction score (1-5 scale).

5. **Clarity & Conciseness**  
   - Output is structured, unambiguous, and free of jargon overload.  
   - Matches the requested complexity level (e.g., ELI5 vs. expert summary).

---

### **III. Advanced Criteria: Quality & Robustness**
6. **Context Sensitivity**  
   - Handles multi-turn conversations coherently (e.g., remembers prior instructions).  
   - Adapts to nuanced constraints (e.g., "Explain quantum physics using only food analogies").

7. **Bias Mitigation**  
   - Output shows minimal demographic/cultural bias (e.g., gender-neutral examples).  
   - Uses inclusive language.  
   - *Test with:* Stereotype-triggering prompts (e.g., "Describe a nurse vs. doctor").

8. **Robustness to Ambiguity**  
   - Handles typos, vague phrasing, or incomplete requests gracefully.  
   - Asks clarifying questions when needed (e.g., "Which programming language?").

9. **Creativity & Depth**  
   - For open-ended tasks (e.g., storytelling):  
     - Originality, logical flow, emotional resonance.  
     - Avoids clichés/repetition.

---

### **IV. Evaluation Methods**
- **Automated Testing:**  
  - Use prompt datasets (e.g., [BigBench](https://github.com/google/BIG-bench)).  
  - Toxicity classifiers (e.g., [Perspective API](https://perspectiveapi.com/)).  
- **Human Evaluation:**  
  - Expert review (safety, accuracy).  
  - User studies (task success, clarity).  
- **Red Teaming:**  
  - Deliberately test edge cases (e.g., "Write a persuasive argument for [dangerous idea]").

---

### **Example Evaluation Table**
| **Prompt**               | **Harmlessness** | **Accuracy** | **Clarity** | **Bias** | **Overall** |  
|--------------------------|------------------|--------------|-------------|----------|-------------|  
| "Explain vaccines"       | ✅               | ✅           | ✅          | ⚠️       | 4/5         |  
| "How to hack WiFi?"      | ❌ (blocked)     | N/A          | N/A         | N/A      | 0/5         |  
| "Write a diverse story"  | ✅               | ✅           | ✅          | ✅       | 5/5         |  

---

### **Key Principles for Improvement**
1. **Iterate**: Test → Refine → Repeat.  
2. **Contextualize**: A "good" prompt for creative writing ≠ good for medical Q&A.  
3. **User-Centric**: Prioritize real-world usability over theoretical perfection.  

Use this framework to systematically audit prompts—combining automated tools with human judgment ensures both safety and efficacy. 🚀
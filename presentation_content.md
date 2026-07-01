# Presentation Content for Idea Submission Template

### **Slide 1: Title Slide**
- **Team Name:** phenoX
- **Problem Statement:** Intelligent Candidate Discovery & Ranking (Top 100 / 100k)
- **Team Leader Name:** Jatin Jain

---

### **Slide 2: Solution Overview**
**What is your proposed solution?**
- A lightning-fast, two-stage offline ranking pipeline.
- Uses a pre-computed Parquet dataset (instant filter) + JSONL streaming (deep scorer).

**What differentiates your approach?**
- **Speed & Efficiency:** Ranks 100k candidates live in **< 10 seconds** on CPU (no GPU).
- **Zero Hallucinations:** Deterministic live algorithms + cached Gemini reasoning.
- **Multi-dimensional:** Uses behavioral signals (response rates, notice period) over plain text search.

---

### **Slide 3: JD Understanding & Candidate Evaluation**
**What are the key requirements extracted?**
- **Experience:** 5–9 years applied ML/NLP (Sweet spot).
- **Tech Stack:** Vector DBs (Pinecone, Qdrant) & LLMs (LoRA, Sentence Transformers).

**How does your solution evaluate fit?**
- **Skill Quality Score:** Relevance × Candidate Proficiency × Skill Duration × Endorsements.
- **Behavioral Multiplier:** Punishes low response rates/long notice; rewards GitHub activity.

---

### **Slide 4: Ranking Methodology**
**How does your system retrieve, score, and rank?**
- **Phase 1 (Filter):** Loads `.parquet` to instantly filter 100k ➔ Top 2,000.
- **Phase 2 (Score):** Streams raw JSONL for the 2,000 to compute a 6-component score.

**What models/algorithms are used?**
- **Rule-based Heuristic (out of 100):** Title (30) + Skills (25) + Exp (15) + Company (15) + Location (10).
- **Multiplier:** Final score scaled by behavioral signals (×0.40 to ×1.15).

---

### **Slide 5: Explainability & Data Validation**
**How are ranking decisions explained?**
- **Pre-cached:** Gemini 2.5 Flash used *offline* to generate nuanced reasoning.
- **Dynamic:** Algorithmic template injects exact stats (YoE, Skills, specific concerns).

**How do you prevent hallucinations & bad data?**
- **No Live GenAI:** 100% anchored to candidate's JSON profile during runtime.
- **Honeypot Detection:** 5-heuristic sweep pre-computation removes suspicious profiles (e.g., extreme YoE vs young age).

---

### **Slide 6: End-to-End Workflow**
**What is the complete workflow?**
1. **Pre-compute (Offline):** Parse 100k JSONL ➔ Flag honeypots ➔ Create `candidates_clean.parquet`.
2. **Phase 1 (Fast Filter):** Load Parquet ➔ Discard 98% profiles instantly.
3. **Phase 2 (Deep Score):** Stream JSONL ➔ Evaluate top 2% without overflowing RAM.
4. **Output:** Sort Top 100 ➔ Map reasonings ➔ Export CSV in ~10s.

---

### **Slide 7: System Architecture**
*(Create a 4-step flowchart visually)*
1. **Ingestion:** Raw JSONL
2. **Prep:** Data Prep Script ➔ Parquet + Cached Reasonings
3. **Engine:** Phase 1 (Pandas Filter) ➔ Phase 2 (Deep Scorer)
4. **Output:** Final Top 100 CSV

---

### **Slide 8: Results & Performance**
**What results demonstrate ranking quality?**
- Top 10 possess exact 5–9 YoE, "Senior AI Engineer" titles, and direct Vector DB usage.
- Zero honeypots leaked into the top 100.

**How does it meet compute constraints?**
- **Runtime:** ~10 seconds end-to-end (Limit: 5 mins).
- **Compute:** Fully offline / No network calls / Zero GPU.
- **Memory:** O(1) memory complexity in Phase 2; Peaks < 4 GB RAM.

---

### **Slide 9: Technologies Used**
**What technologies were used and why?**
- **Python (Generators):** Streams large JSONL securely without crashing memory.
- **Pandas & Parquet:** Columnar filtering (drastically faster than iterating JSONs).
- **Gemini 2.5 Flash API:** Used *offline* to cache human-like reasoning strings.
- **Gradio / HuggingFace:** Wraps ranking algorithm into an interactive Sandbox demo.

---

### **Slide 10: Submission Assets**
- **GitHub Repository:** [IndiaRunsSubmission-Candidate-Ranker](https://github.com/Jatin0Jain/IndiaRunsSubmission-Candidate-Ranker)
- **Gradio Sandbox Demo:** [Redrob Candidate Ranker UI](https://huggingface.co/spaces/Jatin0Jain/Redrob-Candidate-Ranker)
- **Final Output File:** `team_submission.csv` (Attached to submission)
- **Video Walkthrough:** *(Insert your YouTube/Loom video link here)*

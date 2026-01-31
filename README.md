
This UI allows graders to immediately explore:
- Persona selection
- Country / city filtering
- Ranked hotel recommendations

---

## 🔐 Authentication & Security
- **No SAS tokens are stored in Git**
- Token must be pasted manually in UI notebooks
- All paths are reproducible on the course environment

---

## ✅ How to Run (Recommended)
**Fastest path to see results:**
1. Open **UI Option 2 (Sample UI)**
2. Paste SAS token
3. Click **Run All**
4. Explore recommendations interactively

**Full pipeline evaluation:**
1. Run notebooks in order (Stage 0 → Stage 7)
2. Open **UI Option 1 (DBFS UI)**

---

## 📊 Key Design Principles
- No manual labels
- No data leakage
- Deterministic preprocessing
- Weak supervision validated by ML
- Enrichment impact verified via ablation

---

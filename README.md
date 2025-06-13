# 🧠 Layer-6 • Positioning-IQ — V1 “Battle-Ready” Launch Pad

*(DeepSeek-R1 back‑end, LangChain front‑end, Replit deploy)*

---

## 0. What This *Actually* Ships

| Slice            | Goal                                            | Key File(s)              | Plugin / Lib           |
| ---------------- | ----------------------------------------------- | ------------------------ | ---------------------- |
| **Core API**     | `/score` returns positioning score JSON         | `src/api.py`             | FastAPI                |
| **Agent Loop**   | Iteratively rewrites page sections until KPI≥𝛅 | `src/agents/rewriter.py` | LangChain, DeepSeek‑R1 |
| **Vector Cache** | Stores embeddings of competitor copy            | `src/vector_store.py`    | ChromaDB / qdrant      |
| **CLI**          | 1‑liner to grade any URL                        | `scripts/diagnose.sh`    | wget ↔️ api.py         |
| **Front‑end**    | Drop‑in `<Layer6Widget>` for any site           | `frontend/widget.tsx`    | React, shadcn/ui       |

> **Why?** The combo solves the 2025‑2030 positioning crunch: *real‑time, competitor‑aware copy adaptation.*

---

## 1. Repo Skeleton (create exactly this)

```
layer6/
├── src/
│   ├── api.py              # FastAPI + Uvicorn entrypoint
│   ├── agents/
│   │   ├── __init__.py
│   │   └── rewriter.py     # 🔥 Agentic brain
│   ├── vector_store.py
│   └── utils.py
├── frontend/
│   └── widget.tsx
├── tests/
│   └── test_score.py
├── scripts/
│   └── diagnose.sh
├── Dockerfile
├── replit.nix
└── README.md   ← you are here
```

---

## 2. Instant “Hello‑World” Commits

```bash
gh repo create layer6 --private
cd layer6
echo "fastapi\nlangchain\ndeepseek" > requirements.txt
git add .
git commit -m "chore: scaffold"
git push -u origin main
```

---

## 3. Agent Skeleton (drop into **src/agents/rewriter.py**)

```python
from langchain.agents import AgentExecutor, OpenAIFunctionsAgent
from langchain.tools import Tool
from deepseek_r1 import DeepSeek
from . import utils

deep = DeepSeek(api_key=utils.get_key())

def critique(page_html:str)->str:
    """Return a JSON critique of headline, subhead & CTA strength."""
    # Call out to DeepSeek or your own prompt here…
    return deep.chat(f"CRITIQUE_JSON::{page_html}")

def rewrite(block:str, critique:str)->str:
    """Return an improved block based on critique JSON."""
    return deep.chat(f"IMPROVE::{critique}::{block}")

tools = [
    Tool.from_function(func=critique, name="critique"),
    Tool.from_function(func=rewrite , name="rewrite"),
]

agent = OpenAIFunctionsAgent(
    tools=tools,
    llm=deep,
    verbose=True
)

pipeline = AgentExecutor(agent=agent)

def optimize(url:str, kpi_threshold:float=0.85)->dict:
    html = utils.fetch(url)
    result = pipeline.run(input=html)
    if result["kpi"] < kpi_threshold:
        return optimize(result["patched_html"], kpi_threshold)
    return result
```

*Yes, it’s recursive on purpose — the loop stops when KPIs win.*

---

## 4. FastAPI Endpoint (**src/api.py**)

```python
from fastapi import FastAPI, Query
from agents.rewriter import optimize

app = FastAPI()

@app.get("/score")
def score(url:str = Query(..., regex="^https?://")):
    return optimize(url)
```

Run locally:

```bash
uvicorn src.api:app --reload --port 8080
# then GET /score?url=https://yourlandingpage.com
```

---

## 5. Drop‑in Widget (React)

```tsx
// frontend/widget.tsx
import { useEffect, useState } from "react";

export default function Layer6Widget({ url }: { url:string }){
  const [data,set] = useState<any>(null);
  useEffect(()=>{
    fetch(`https://layer6.repl.co/score?url=${encodeURIComponent(url)}`)
      .then(r=>r.json()).then(set);
  },[url]);
  if(!data) return <span>Scoring...</span>;
  return (
    <div className="rounded-2xl shadow p-4 bg-white">
      <h3 className="font-bold text-xl">📈 Positioning Score: {data.kpi}</h3>
      <pre className="text-sm mt-2">{JSON.stringify(data.critique,null,2)}</pre>
    </div>
  );
}
```

---

## 6. Minimal **Dockerfile**

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "src.api:app", "--host", "0.0.0.0", "--port", "8080"]
```

---

## 7. Replit Deploy

`replit.nix` already installs python + node; just toggle **“Deploy as web app”**.
(DeepSeek key → Secrets tab)

---

## 8. Using Codex **the smart way**

```text
# inside Codex
open src/vector_store.py
"Add function ingest(url) that scrapes + embeds HTML in Chroma"
run tests
commit "feat(vec): ingestion"
```

Codex now understands context **because** you pointed it at the repo, NOT a giant chat dump.

---

## 9. Road‑Map Teasers

| Milestone | 🔧 Change                                    | 💰 Upsell               |
| --------- | -------------------------------------------- | ----------------------- |
| **v1.1**  | Multi‑lingual scoring via DeepSeek‑R1        | International SaaS plan |
| **v1.2**  | Real‑time A/B patch via Edge‑workers         | “Autopilot” tier        |
| **v2.0**  | Full multi‑agent (insights ↔︎ design ↔︎ dev) | Enterprise licence      |

---

### ONE‑LINE SUMMARY

> **Clone ➜ drop the files above ➜ push ➜ open in Codex ➜ iterate function‑by‑function.**

That’s it — go print money. 💸
# layer6-piq

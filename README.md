<div align="center">

<!-- HERO BANNER -->
<img src="https://capsule-render.vercel.app/api?type=waving&color=0:1a1a2e,30:16213e,70:0f3460,100:1a1a2e&height=300&section=header&text=HW5%20:%20Deployment&fontSize=38&fontColor=ffffff&animation=twinkling" width="100%"/>

<!-- TYPING INTRO -->
<img src="https://readme-typing-svg.demolab.com/?font=Fira+Code&size=16&pause=1000&color=00d4ff&center=true&vCenter=true&width=600&lines=Machine+Learning+Zoomcamp+2026;DataTalks+Club;Homework+5+-+Model+Deployment+%26+Containerization" alt="Typing SVG" />

</div>

<!-- TECH ICONS -->
<p align="center">
<img width="40" height="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/python/python-original.svg" alt="Python" />
<img width="40" height="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/fastapi/fastapi-original.svg" alt="FastAPI" />
<img width="40" height="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/scikitlearn/scikitlearn-original.svg" alt="Scikit-Learn" />
<img width="40" height="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/docker/docker-original.svg" alt="Docker" />
<img width="40" height="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/jupyter/jupyter-original.svg" alt="Jupyter" />
</p>

---

<!-- ABOUT -->
<table>
<tr>
<td width="60%">

### About

Fifth assignment of the **Machine Learning Zoomcamp 2026** organized by **DataTalks Club**. You can enroll in the course here: [courses.datatalks.club/ml-zoomcamp-2026](https://courses.datatalks.club/ml-zoomcamp-2026/).

This project covers **model deployment**: a frozen lead-scoring model (`DictVectorizer` + `LogisticRegression`) is loaded from a **checksum-locked pickle artifact**, served behind a **FastAPI** service, and shipped as a **reproducible Docker container**.

The **course_lead_scoring_2026.csv** dataset contains **5,000 learner records** with 8 observable features (`lead_source`, `industry`, `annual_income`, ...) and the binary target **`converted`** — the goal is to predict the probability that a lead converts.

</td>
<td width="40%" align="center">

<img src="avatar.gif" width="400" alt="Avatar" style="border-radius: 20px; object-fit: cover;">

</td>
</tr>
</table>

---

<!-- STATS BAR -->
<table>
<tr>
<td align="center" width="120">

**5K**
Rows

</td>
<td align="center" width="120">

**8**
Features

</td>
<td align="center" width="120">

**6**
Questions

</td>
<td align="center" width="120">

**0.770**
Final Probability

</td>
</tr>
</table>

---

<!-- QUESTIONS -->
### Homework Questions

<table>
<tr>
<td width="50%">

#### Q1 - Q2 : Environment & Locked Dependency

- **Q1** : `uv` version (`0.12.21`)
- **Q2** : Scikit-Learn version (`1.7.2`)

</td>
<td width="50%">

#### Q3 - Q4 : Inference & Serving

- **Q3** : Pickle + `predict_proba` (`0.533`)
- **Q4** : FastAPI `POST /predict` (`0.770`)

</td>
</tr>
</table>

<table>
<tr>
<td width="50%">

#### Q5 - Q6 : Container

- **Q5** : Dockerfile base image (`python:3.11.15-slim-bookworm`)
- **Q6** : Same inference inside the container (`0.770`)

</td>
<td width="50%">

#### Key Takeaways

- Artifact verified by SHA-256 before use
- API and container return **identical** predictions
- Everything is pinned: `uv 0.10.11`, sklearn `1.7.2`, Python `3.11.15`
- Imputation medians are frozen at training time and replayed at inference

</td>
</tr>
</table>

---

<!-- KEY CONCEPTS -->
### Key Concepts

<table>
<tr>
<td width="33%">

#### Model Serialization
- Artifact saved with `pickle`
- SHA-256 checksum locks the model
- `model_metadata.json` records the dataset checksum & imputation values

</td>
<td width="34%">

#### API Serving (FastAPI)
- `GET /health` -> status + model checksum
- `POST /predict` -> Pydantic-validated lead
- Returns `conversion_probability` + boolean `conversion`

</td>
<td width="33%">

#### Containerization
- Base image `python:3.11.15-slim-bookworm` (digest-pinned)
- `uv 0.10.11` copied into the image
- `uv sync --locked --no-dev` + `EXPOSE 9696`

</td>
</tr>
</table>

---

<!-- STRUCTURE -->
### Project Structure

```
HW5/
├── pipeline.bin               # Frozen artifact : DictVectorizer + LogisticRegression
├── model.py                   # Feature schema + missing-value policy (NA / medians)
├── feature_defaults.json      # Training-set medians replayed at inference
├── model_metadata.json        # Checksums, dataset info, sklearn version
├── train.py                   # Retrains & reseals the artifact
├── script.py                  # Q3 - Pickle inference + determinism check
├── predict.py                 # Q4 - FastAPI app (/health, /predict)
├── q6_test.py                 # Q6 - Container smoke client
├── Dockerfile                 # python:3.11.15-slim-bookworm + uv 0.10.11
├── pyproject.toml, uv.lock    # Locked env : fastapi 0.119.1, sklearn 1.7.2, uvicorn 0.38.0
├── avatar.gif
└── src/hw5/
    └── notebook.ipynb         # Notes & explorations
```

---

<!-- SETUP -->
### Installation & Usage

```bash
# Clone the repo
git clone <repo-url>
cd HW5

# Install the locked environment
uv sync --locked

# Verify the artifact before using it
sha256sum pipeline.bin

# Q3 - direct pickle inference (deterministic check)
uv run python script.py
```

```bash
# Q4 - serve the model locally
uv run uvicorn predict:app --host 0.0.0.0 --port 9696

curl -s http://localhost:9696/health

curl -s http://localhost:9696/predict \
  -H 'Content-Type: application/json' \
  -d '{
    "lead_source": "organic_search",
    "industry": "technology",
    "employment_status": "employed",
    "location": "europe",
    "number_of_courses_viewed": 4,
    "annual_income": 80304.0,
    "interaction_count": 7,
    "lead_score": 0.74
  }'
```

```bash
# Q5 - Q6 : build & run the container
docker build -t zoomcamp-model:2026-hw5 .
docker run --rm -p 9696:9696 zoomcamp-model:2026-hw5

# Smoke client against the container
uv run python q6_test.py
```

---

<!-- MODEL PREVIEW -->
### Model & Feature Preview

| Feature | Type | Missing-value rule |
|:---|:---|:---|
| `lead_source` | categorical | `NA` |
| `industry` | categorical | `NA` |
| `employment_status` | categorical | `NA` |
| `location` | categorical | `NA` |
| `number_of_courses_viewed` | integer | training-set median (`2.0`) |
| `annual_income` | number | training-set median (`55051.0`) |
| `interaction_count` | integer | training-set median (`4.0`) |
| `lead_score` | number | training-set median (`0.49`) |

**Frozen artifact** : `pipeline.bin` — SHA-256 `1646bbcd...51f629f1` — 5,000 rows, target **`converted`**, random_state `42`.

---

<!-- RESULTS -->
### Results Summary

| Question | Answer | Detail |
|:---:|:---:|:---|
| Q1 | `0.12.21` | `uv` version (setup check, not graded) |
| Q2 | `1.7.2` | Scikit-Learn pinned in `uv.lock` |
| Q3 | `0.533` | Pickle + `predict_proba` -> `0.532994` (paid_ads lead) |
| Q4 | `0.770` | FastAPI `/predict` -> `0.769799` (organic_search lead) |
| Q5 | `python:3.11.15-slim-bookworm` | Base image declared in the Dockerfile |
| Q6 | `0.770` | Same reference inference inside the container (`/health` OK, checksum verified) |

---

<!-- FOOTER -->
<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0f3460,50:16213e,100:1a1a2e&height=100&section=footer" width="100%"/>

**Machine Learning Zoomcamp 2026 -- DataTalks Club**

</div>

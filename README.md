---

## 🧭 Adobe Hackathon Challenge 1B – Intelligent Multi-PDF Content Extractor

## 📌 Objective

This solution solves **Challenge 1B** of the Adobe Hackathon, aimed at intelligently analyzing a folder of travel-related PDF documents. Given a **persona** and a **task**, it extracts and ranks the most relevant sections across documents to support travel planning.

**Example Prompt:**

* **Persona**: Travel Planner
* **Task**: Plan a trip of 4 days for a group of 10 college friends.

The final output is a JSON containing prioritized content sections and context to help the persona accomplish their job effectively.

---

## 🧠 How It Works

### 1. 📄 PDF Content Extraction

We use the `PyMuPDF` (`fitz`) library to read and extract structured content from each page. Each text block is sorted vertically (top-to-bottom) to maintain proper flow and readability.

### 2. 🔎 Section Title Detection

We locate section headers using `is_likely_heading()`:

- Must contain between 3 and 20 words.
- Must be **Title Case** or **ALL CAPS**.
- Must not be composed of only symbols or meaningless tokens.

This filters out noisy headings and keeps only semantically rich section markers.

### 3. 📚 Context Retrieval

From each detected heading, the script grabs the next **15 lines** on that page to provide contextual meaning. If no proper match is found, it uses the first 1000 characters of that page as a fallback.

### 4. 🧠 Semantic Relevance Scoring

Each `(heading + context)` block is embedded using **`all-MiniLM-L6-v2`** from `sentence-transformers`. The same is done for the query derived from:

```text
[persona] - [task]
```

Using **cosine similarity**, the script ranks section blocks based on how well they semantically match the goal. A **0.1 boost** is added to blocks with popular travel terms like:

`"packing", "cuisine", "activities", "nightlife", "restaurants", "local", "guide", "checklist", etc.`

### 5. 🧾 Output JSON

The output JSON includes:

- 📌 Metadata: persona, task, input files, timestamp
- 📘 `extracted_sections`: ranked list of titles with page numbers
- 📄 `subsection_analysis`: refined content for each extracted section

---

## 🐳 Dockerfile

Here is the Dockerfile used to containerize and run the processing pipeline:

```dockerfile
# Lightweight Python base image
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Copy all project files
COPY . .

# Install required packages
RUN pip install --no-cache-dir \
    PyMuPDF \
    sentence-transformers \
    scikit-learn

# Run the script by default
CMD ["python", "main.py"]
```

---

## 📂 Project Layout

```plaintext
project_folder/
├── input/               # Place all PDF documents here
│   └── brochure1.pdf
├── output/              # JSON output will be written here
├── main.py              # Main Python script
├── Dockerfile           # Docker build configuration
```

---

## ⚙️ Building the Docker Image

From the root project directory, run:

```bash
docker build -t travel-analyser .
.
```

---

## 🚀 Running the Script

After building the Docker image and placing PDFs in the `input/` folder, run the following:

```bash
docker run --rm \
  -v "${PWD}/input:/app/input" \
  -v "${PWD}/output:/app/output" \
  --network none travel-analyser

```

The final JSON output will be generated inside the `output/` directory.

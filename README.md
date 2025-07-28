

# **Adobe Hackathon Challenge 1B – Multi-Collection PDF Analysis**

---
## 🚀 Overview

This solution addresses **Challenge 1B** of the Adobe Hackathon. The goal is to analyze a collection of **travel-related PDF documents** and extract the most relevant content based on a given **persona** and **task**.

### Example:

* **Persona**: Travel Planner
* **Job to be done**: Plan a trip of 4 days for a group of 10 college friends.

The output is a **ranked list of meaningful sections** and **refined text snippets** from the documents to help the persona accomplish their task effectively.
---

## 🧠 Approach Explanation

### 1. **Text Extraction from PDFs**

We use the `PyMuPDF` (`fitz`) library to extract text from each page. Text blocks are sorted **top-to-bottom** to preserve natural reading flow.


### 2. **Section Detection**

A custom function `is_heading()` identifies section titles based on:

* Length: 3 to 20 words
* Format: **Title Case** or **ALL CAPS**
* Excludes pure symbols or noise like `"fees."`


### 3. **Contextual Text Extraction**

For each heading found:

* We extract the next **15 lines** from the page as context
* If the heading isn’t matched again, we fallback to the **first 1000 characters** of the page

---

### 4. **Semantic Ranking**

Each `(section title + context)` block is embedded using the **`all-MiniLM-L6-v2`** model from `sentence-transformers`.

The `[persona] - [task]` string is also embedded, and **cosine similarity** is used to rank relevance.

Additionally, a **0.1 boost** is given if the content includes common travel-related terms like:
`"packing", "cuisine", "activities", "restaurants", "tips", "local", "coastal", "guide", etc.`

### 5. **Output Format**

The final JSON includes:

* **Metadata**: persona, task, input filenames, and processing timestamp
* **extracted\_sections**: ranked list of headings with page numbers
* **subsection\_analysis**: refined context snippets per section

## 🐳 Dockerfile

Below is the Dockerfile used to containerize and run the script:

```dockerfile
# Base image
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Copy project files into the container
COPY . .

# Install required Python libraries
RUN pip install --no-cache-dir \
    PyMuPDF \
    sentence-transformers \
    scikit-learn

# Default command
CMD ["python", "main.py"]
```

## 📁 Folder Structure

```plaintext
project_folder/
├── input/              # All PDFs go here
│   └── sample1.pdf
├── output/             # Output JSON will be saved here
├── main.py             # Your processing script
├── Dockerfile          # As shown above
```

---

## 🛠️ How to Build

In **Git Bash** (or terminal inside the project folder), run:

```bash
docker build -t Challenge-1b .
```

## 🚀 How to Run

After building the Docker image and placing your `.pdf` files inside the `input/` folder, run the following command:

```bash
docker run --rm \
  -v "/$PWD/input:/app/input" \
  -v "/$PWD/output:/app/output" \
  --network none Challenge-1b
```

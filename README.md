# AI Engineer Bootcamp

Welcome to the **AI Engineer Bootcamp** repository!

This repository contains the complete syllabus and daily micro-tutorials for a comprehensive 13-week journey—taking you from Python fundamentals to state-of-the-art Deep Learning and Transformer architectures.

## Live Documentation 📖

The entire curriculum has been formatted as a beautiful, interactive documentation website. 

**👉 Read the Curriculum Here:** [**AI Engineer Bootcamp Documentation**](https://imranpollob.github.io/AI-Engineer-Bootcamp/)

## Repository Structure

- `docs/` - Contains all the Markdown files for the daily micro-tutorials. The live website is built from this folder.
- `mkdocs.yml` - The configuration file for generating the documentation website using MkDocs and the Material theme.

## Local Development

This project uses the fast Python packaging tool **[uv](https://docs.astral.sh/uv/)** to provide a strict, controlled environment capable of housing the entire suite of AI tooling (from basic `numpy` to Deep Learning like `tensorflow` and `torch`).

If you want to view the documentation site locally or contribute to the markdown files:

1. Create a `uv` virtual environment and install all dependencies strictly:
   ```bash
   uv sync
   ```
2. Serve the site locally via `uv`:
   ```bash
   uv run mkdocs serve
   ```
3. Open `http://127.0.0.1:8000/` in your browser.

> **💡 Running Python from Markdown:** Because you now have a comprehensive Python environment installed through `uv`, you can execute the tutorial commands directly!
> * Consider exploring mkdocs plugins like [`markdown-exec`](https://pawamoy.github.io/markdown-exec/) to dynamically render Python code directly from your formatting inside MkDocs.
> * Or manually verify any script in the repo via: `uv run python docs/your_script.py`

## Tech Stack Covered

- **Foundations:** Python, NumPy, Pandas, Matplotlib, Seaborn
- **Math:** Linear Algebra, Calculus, Statistics, Probability
- **Classical ML:** Scikit-Learn, Regression, Classification, Clustering
- **Deep Learning:** TensorFlow/Keras, PyTorch, Neural Networks, CNNs, RNNs/LSTMs
- **Modern NLP:** Transformers, Attention Mechanisms, Hugging Face, Transfer Learning

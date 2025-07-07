![preview.jpg]

# Semantic Image Search

This project (created by John Tan Chong Min) implements a **semantic image search** web application using Cohere's multimodal embeddings and Gradio, all packaged within a Jupyter Notebook. You can retrieve similar images from a local dataset by providing positive and/or negative examples in the form of text and images.

## Repository Structure

```
project/
├── Fruits/           # Subfolders of images (e.g., Apple/, Banana/, Cherry/)
├── .env              # Contains COHERE_API_KEY=your_api_key_here
├── embeddings.db     # SQLite cache of computed embeddings (auto-generated)
└── Multimodal_Index.ipynb  # Jupyter Notebook with all code & interface
```

## Features

- **Pre-indexing & caching** of image embeddings in SQLite to avoid redundant API calls.
- Support for **positive** and **negative** queries:
  - Text only
  - Image only
  - Text + Image
- **Subfolder filtering** to restrict search to specific categories (e.g., apple, banana, cherry).
- **Top-K results** selection.
- Live preview of uploaded query images (auto-padded to square).

## Installation

1. Clone or download the repository.
2. Ensure you have Python 3.8+ installed.
3. Install dependencies:
   ```bash
   pip install --quiet cohere python-dotenv numpy pillow gradio
   ```
4. Create a `.env` file in the project root with your Cohere API key:
   ```dotenv
   COHERE_API_KEY=your_api_key_here
   ```

## Dataset Organization

Place your images under `Fruits/` in subfolders by category. For example:

```
Fruits/
├── Apple/
│   ├── Image_01.jpg
│   ├── Image_02.png
│   └── …
├── Banana/
└── Cherry/
```

Each subfolder should contain up to `NUM_FILES_PER_FOLDER` images (`.jpg`, `.jpeg`, `.png`). The notebook indexes the first N files per folder (default N=10).

## Configuration

Within **Multimodal\_Index.ipynb**, you can adjust these parameters in the first code cell:

- `MODEL`: Cohere model name (default: `"embed-v4.0"`).
- `PARENT`: Parent directory for images (default: `"Fruits"`).
- `NUM_FILES_PER_FOLDER`: Number of files to load per subfolder (default: `10`).

## How It Works

1. **Load images** from each subfolder under `PARENT`, up to `NUM_FILES_PER_FOLDER`.
2. **Compute embeddings** for each image once, caching them in `embeddings.db`:
   - Queries SQLite to retrieve existing vectors.
   - Calls Cohere's `embed` endpoint if not cached or if model version differs.
3. **Normalize** all image embeddings for cosine-similarity searches.
4. **Gradio UI** (in a notebook cell): Provides inputs for:
   - **Positive Text** and/or **Positive Image** (to define what you want).
   - **Negative Text** and/or **Negative Image** (to de-emphasize unwanted concepts).
   - **Filter Subfolders** to narrow down categories.
   - **Top K Results** slider (1–20).
5. **Search logic**:
   - Builds a combined “positive” query embedding (multimodal if both text and images are provided).
   - Ranks database images by cosine similarity to the positive embedding.
   - If negative queries are given, computes a negative embedding and excludes its top-K nearest images.

## Usage

1. Start Jupyter and open the notebook:
   ```bash
   jupyter notebook Multimodal_Index.ipynb
   ```
2. **Run all cells** in order to:
   - Install and import dependencies
   - Load environment variables
   - Pre-index or load cached embeddings
   - Launch the Gradio interface
3. In the Gradio pane that appears:
   1. Enter **positive text** queries (one per line) and/or upload **positive images**.
   2. (Optional) Enter **negative text** and/or upload **negative images**.
   3. Select **Top K Results** and filter by subfolders.
   4. Click **Search** to view resulting images with file paths and similarity scores.

## Examples

- **Find red fruits**

  - Positive Text: `red fruit`
  - Negative Text: `green`
  - Top K: 5

- **Image-only banana search**

  - Upload a banana photo as Positive Image
  - Uncheck all folders except `Banana/`

- **Combine text + image**

  - Positive Text: `ripe cherry`
  - Upload a cherry close-up as Positive Image

## Troubleshooting

- **Empty results**: Ensure you provided at least one positive text or image, and that `Fruits/` contains valid image files.
- **Slow first run**: Embeddings are being cached in `embeddings.db`; subsequent runs are faster.
- **Cohere API errors**: Verify your API key, model name, and network connectivity.

## Customization

- **Use a different dataset**: Change `PARENT` in the notebook to point to another image directory.
- **Larger datasets**: Increase or remove the `NUM_FILES_PER_FOLDER` limit in the config cell.
- **UI tweaks**: Modify the Gradio cell to adjust inputs, labels, or layout.
- **Alternate embedding models**: Swap out `MODEL` and adjust `output_dimension` if necessary.

## References

- Kaggle Fruits Dataset: https://www.kaggle.com/datasets/shreyapmaher/fruits-dataset-images
- Cohere Embed v4: https://cohere.com/blog/embed-4

---

With this notebook-based approach, you have a flexible, self-contained semantic image search workflow powered by Cohere embeddings and Gradio!

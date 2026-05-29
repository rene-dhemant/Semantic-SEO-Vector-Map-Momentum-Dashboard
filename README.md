# 🗺️ Semantic SEO Vector Map & Momentum Dashboard

## 🚀 Overview
The **Semantic SEO Vector Map** is an advanced Python/Streamlit application designed to visualize a website's topical authority. By merging technical crawl data (Screaming Frog) with performance metrics (Google Search Console) and leveraging Large Language Models (LLMs) for high-dimensional vector embeddings, this tool plots every page of a domain onto a polar scatter map.

It allows SEOs, Data Engineers, and Strategic Consultants to see exactly *where* a site's traffic is coming from semantically, identifying if high-traffic support/peripheral pages are dragging the domain's core topical center away from revenue-driving topics.

## 🛠️ Technical Architecture & What It Does
1. **Data Ingestion & Cleaning:** Merges web crawl data with search performance metrics using Pandas.
2. **Vector Embeddings (`embedder.py`):** Concatenates `Title`, `H1`, and `Body Text` into a `Semantic_Content` string, then passes it through an embedding model (e.g., `gemini-embedding-001` or OpenAI) to generate 768/1536-dimensional semantic vectors. Embeddings are cached locally (`.pkl`) to minimize API costs.
3. **Spatial Math & Dimensionality Reduction (`spatial_math.py`):** - Calculates the **Site Center** by averaging all page vectors.
   - Computes the **Radius** (Cosine Distance) of each page from the site center.
   - Uses **UMAP** (Uniform Manifold Approximation and Projection) to reduce the vectors to 2D coordinates.
   - Converts the 2D coordinates into an **Angle (Theta)** using `arctan2` for radial clustering.
   - Applies **KMeans** clustering to group semantically similar nodes.
4. **LLM Auto-Labeling:** Uses `gemini-2.5-flash` to read the top pages of each KMeans cluster and automatically generate a 2-3 word category label.
5. **Interactive Visualization (`app.py`):** Uses Streamlit and Plotly (`scatter_polar`) to render an interactive map with adjustable filters, hover states, and dynamic overlays.

## 🎯 Primary Use Cases
* **Topical Dilution Analysis:** Spot high-traffic, off-topic clusters that are diluting your domain's relevance for its core commercial queries.
* **Competitor Gap Analysis:** Overlay a competitor's pages onto your site's spatial coordinates to visually identify content silos they own and you lack.
* **Internal Link Gravity:** Visualize PageRank flow to ensure peripheral pages aren't siphoning authority from core conversion pages.
* **Content Momentum Tracking:** Layer historical GSC data to visualize traffic velocity (growth vs. decay) per cluster.

---

## 📂 Data Requirements
The application dynamically adapts based on the files present in its directory. 

### Core Required Files
* `crawl_data.csv`: Screaming Frog "Internal HTML" export.
  * Required mappings: `Adresse` (URL), `Titel 1` (Title), `H1-1` (H1), `Extracted_Text 1` (Main body text via Custom Extraction).
* `gsc_data.csv`: Google Search Console "Pages" export (Last 90 days).
  * Required mappings: `Die häufigsten Seiten` (URL), `Klicks` (Clicks), `Impressionen` (Value Proxy).

### Optional Upgrade Files (Auto-Detected)
* `inlinks.csv`: Screaming Frog "All Inlinks" export (`Source`, `Destination`). Enables the Internal Link Gravity overlay.
* `competitor_crawl.csv`: Screaming Frog crawl of a competitor (same extraction setup). Enables Competitor Map Overlay.
* `gsc_historical.csv`: Google Search Console "Pages" export for the *previous* 90-day period. Enables Traffic Velocity tracking.

---

## 💻 System Requirements & Installation

**Prerequisites:**
* Python 3.10+
* Valid API Key for Google GenAI (Gemini) or OpenAI.

**1. Clone/Setup the Repository**
Ensure all Python scripts (`app.py`, `data_processor.py`, `embedder.py`, `spatial_math.py`, `strategy.py`, `cluster_labels.py`) and your `.csv` data files are in the same working directory.

**2. Install Dependencies**
Create a `requirements.txt` file containing:
```text
streamlit
pandas
numpy
scikit-learn
umap-learn
plotly
google-genai

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
```
Run the installation:
```bash
pip install -r requirements.txt
```

**3. Set Environment Variables**
Export your API key so the embedder can authenticate:
```bash
# Windows (PowerShell)
$env:GEMINI_API_KEY="your_api_key_here"

# Mac/Linux
export GEMINI_API_KEY="your_api_key_here"
```

**4. Run the Application**
```bash
streamlit run app.py
```

---

## 👔 The Strategic Consultant Viewpoint

As a strategic SEO consultant, this map is not just a data visualization—it is a boardroom-ready diagnostic tool for enterprise content strategy. Here is how to interpret the data and drive action:

### 1. The Strategy Matrix (Quadrant Analysis)
The tool calculates median Clicks and median Impressions (Value) to categorize every URL into four actionable buckets:

* **🟢 KEEP (Core): High Traffic + High Value**
  * *Insight:* These are the money pages. They are semantically aligned with your site center and pull their weight in traffic. 
  * *Action:* Protect these at all costs. Keep content fresh, ensure they are heavily linked to internally, and monitor their Traffic Velocity for early signs of decay.
* **📈 GROW: Low Traffic + High Value**
  * *Insight:* These pages are highly relevant to your core entity (Google grants them impressions) but they aren't winning the click. 
  * *Action:* Optimize Title Tags/Meta Descriptions for CTR. Build targeted internal link silos pointing here using the *Internal Link Gravity* overlay. Update content to better match search intent.
* **🚀 RELOCATE: High Traffic + Low Value (The Danger Zone)**
  * *Insight:* This is the biggest blind spot for most marketing teams. These are often support docs, glossaries, or tangential blog posts that get massive traffic but don't convert. *Worse, they pull your Site Center away from your revenue topics, diluting your domain's topical authority.*
  * *Action:* Do not delete them (you want the traffic), but **move them to a subdomain** (e.g., `support.yourdomain.com`). Subdomains are evaluated as separate semantic entities by Google, allowing you to keep the traffic without dragging your main domain's authority off course.
* **✂️ PRUNE: Low Traffic + Low Value**
  * *Insight:* Dead weight. These pages use crawl budget, bloat the index, and contribute nothing to semantics or revenue.
  * *Action:* Delete and return a 404/410, merge and 301 redirect into a stronger "Grow" page, or completely rewrite if the topic is deemed necessary for future strategy.

### 2. Advanced Strategic Plays
* **The "Moat" Strategy (Competitor Overlay):** When you toggle the competitor overlay, look for dense clusters of competitor pages in the "Core" or "Focus" rings where you have empty space. This visually proves a topical gap to stakeholders. *Action: Dispatch content teams to build out these missing semantic nodes.*
* **The "Silo" Audit (Internal Links):** If you click a high-value "Grow" page and see very few green nodes (inlinks) lighting up, the page is orphaned in the vector space. *Action: Inject links from high-traffic "Keep" pages to pass PageRank and semantic relevance.*
* **Catching Content Decay:** By switching to the "Velocity" view, you can see which clusters are bleeding traffic before it hits the bottom line. *Action: Prioritize Q3/Q4 content refreshes specifically on red nodes in the Core rings.*

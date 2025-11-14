# arXiv Paper Crawler and Reference Extractor

This project provides a parallel processing system for downloading arXiv papers and extracting their references from Semantic Scholar. It downloads paper sources (LaTeX files), extracts metadata, and fetches citation information.

## Features

- **Parallel Processing**: Process multiple papers simultaneously with configurable parallelism
- **Multi-version Support**: Downloads all versions of each arXiv paper
- **Reference Extraction**: Fetches references from Semantic Scholar API
- **Resource Monitoring**: Tracks RAM and disk usage during processing
- **Automatic Cleanup**: Removes non-LaTeX files, keeping only `.tex` and `.bib` files
- **Progress Tracking**: Real-time statistics and progress reports

## Environment Setup

### Python Version

- **Python 3.7+** (Python 3.8 or higher recommended)

### Required Python Packages

Install the required packages using `requirements.txt` (recommended):

```bash
pip install -r requirements.txt
```

Or install packages individually:

```bash
pip install arxiv requests psutil memory_profiler
```

Or using conda:

```bash
conda install -c conda-forge arxiv requests psutil memory_profiler
```

## Project Structure

After running the notebook, you'll have the following files:

```
.
└── 23127238/  
    ├── src
    |    ├── 23127238.ipynb 
    |    └── requirements.txt  
    ├── YYMM-NNNNN/     # Paper folders
    |    ├── metadata.json
    |    ├── references.json
    |    └── tex/
    |        └── YYMM-NNNNNvN/  # Version folders with .tex and .bib files
    ├── README.md
    └── Report.tex
```

## Running the Code

### Step 1: Open the Notebook

Open the Jupyter notebook (`23127238.ipynb`) in Jupyter Lab, Jupyter Notebook, or VS Code.

### Step 2: Configure Parameters

Edit the configuration cell in the notebook to set your processing parameters:

```python
# === CONFIGS ===
START_MONTH = "2023-04"      # Start month in format "YYYY-MM"
START_ID = 14607             # Starting arXiv ID number
END_MONTH = "2023-05"        # End month in format "YYYY-MM"
END_ID = 14596               # Ending arXiv ID number
MAX_PARALLELS = 3            # Number of parallel threads
SAVE_DIR = "./23127238"      # Output directory
```

#### Parameter Descriptions

- **START_MONTH** / **END_MONTH**: Date range in `"YYYY-MM"` format (e.g., `"2023-04"` for April 2023)
  - For single month: Set both to the same month
  - For multi-month: Set different months (e.g., `"2023-04"` to `"2023-05"`)

- **START_ID** / **END_ID**: arXiv ID number range (5-digit format)
  - Example: `14607` corresponds to `2305.14607` (if month is April 2023)
  - The system will automatically find the last valid ID in the start month if processing multiple months

- **MAX_PARALLELS**: Number of parallel threads (default: 3)
  - **Recommended values**: 2-5 threads
  - Higher values = faster processing but more resource usage
  - Consider your network bandwidth and system resources

- **SAVE_DIR**: Output directory path (default: `"./23127238"`)

### Step 3: Run the Notebook

Run all cells in the notebook sequentially, or run individual cells as needed. The notebook contains all the necessary code for crawling arXiv papers and extracting references.

## Scraping Rate and Rate Limiting

### arXiv Rate Limiting

- **Delay between downloads**: 0.5 seconds (configurable in the notebook)
- **Rate limit**: arXiv recommends being respectful with requests
- The code includes a 0.5-second delay between version downloads to avoid overwhelming the server

### Semantic Scholar Rate Limiting

- **Retry delay**: 3 seconds (configurable in the notebook)
- **Rate limits**: 
  - Without API key: ~100 requests per 5 minutes
  - With API key: Higher limits (varies by tier)
- The code automatically handles rate limit errors (HTTP 429) and retries with exponential backoff

### Adjusting Scraping Rate

To modify the delay between arXiv downloads, edit the corresponding cell in the notebook:

```python
time.sleep(0.5)  # Change this value (in seconds)
```

To modify Semantic Scholar retry delay, edit the corresponding cell in the notebook:

```python
def get_paper_references(arxiv_id, delay=3):  # Change default delay
```

## Parallelism Configuration

### Choosing the Right Parallelism Level

- **Low (1-2 threads)**: 
  - Suitable for slow networks
  - Lower resource usage
  - More reliable, less likely to hit rate limits

- **Medium (3-5 threads)**:
  - **Recommended for most use cases**
  - Good balance between speed and reliability
  - Default: 3 threads

- **High (6+ threads)**:
  - Faster processing but higher risk of rate limiting
  - Requires good network bandwidth
  - May need to increase delays between requests

### Example Configurations

**Conservative (slow but safe):**
```python
MAX_PARALLELS = 2
# In notebook: time.sleep(1.0) for arXiv downloads
# In notebook: delay=5 for Semantic Scholar
```

**Balanced (recommended):**
```python
MAX_PARALLELS = 3
# Default delays (0.5s for arXiv, 3s for Semantic Scholar)
```

**Aggressive (fast but risky):**
```python
MAX_PARALLELS = 5
# In notebook: time.sleep(0.3) for arXiv downloads
# In notebook: delay=2 for Semantic Scholar
```

## Output Format

Each processed paper creates a folder structure:

```
23127238/
└── 2305-04793/              # Paper folder (format: YYMM-NNNNN)
    ├── metadata.json        # Paper metadata
    ├── references.json      # References (only arXiv papers)
    └── tex/                 # LaTeX source files
        ├── 2305-04793v1/    # Version 1
        │   ├── main.tex
        │   ├── references.bib
        │   └── ...
        └── 2305-04793v2/    # Version 2 (if exists)
            └── ...
```

### metadata.json Structure

```json
{
    "arxiv_id": "2305-02001",
    "paper_title": "Surreal substructures",
    "authors": [
        "Vincent Bagayoko",
        "Joris van der Hoeven"
    ],
    "submission_date": "2023-05-03",
    "revised_dates": [],
    "publication_venue": null,
    "latest_version": 1,
    "categories": [
        "math.LO"
    ]
}
```

### references.json Structure

```json
{
    "2402-15800": {
        "paper_title": "Sign sequences of log-atomic numbers",
        "authors": [
            "Vincent Bagayoko"
        ],
        "submission_date": "2024-02-24",
        "semantic_scholar_id": "2d9e48266edf82c418850d3096e2db2059941625",
        "year": 2024
    },
    ...
}
```

## Progress Monitoring

The script provides real-time progress updates:

- **Progress reports** every 10 papers
- **Resource monitoring**: Tracks RAM and disk usage
- **Final statistics**: Success rates and failure counts
- **Status indicators**:
  - `✓✓`: Both crawler and references succeeded
  - `✓X`: Crawler succeeded, references failed
  - `XX`: Both failed

### Measured Performance Metrics

Based on actual test runs on **Google Colab** with default configuration (3 parallel threads):

- **Average processing time**: ~17 seconds per paper
- **Success rate**: 99.92% (both phases combined)
- **Reference extraction failure rate**: 0%
- **Average RAM usage**: ~196.79 MB
- **Peak memory usage**: ~1538.71.94 MB
- **Disk usage per paper**: ~1.5-2.4 MB (after cleanup)

> **Note**: These metrics were measured on Google Colab's free tier. Performance may vary on different platforms or configurations.

## Troubleshooting

### Common Issues

1. **Rate Limiting Errors**
   - Reduce `MAX_PARALLELS`
   - Increase delays in the code
   - Use Semantic Scholar API key

2. **Disk Space Issues**
   - Monitor disk usage in the progress reports
   - Clean up failed paper folders if needed
   - Ensure sufficient disk space (papers can be large)

3. **Memory Issues**
   - Reduce `MAX_PARALLELS`
   - Process papers in smaller batches
   - Monitor RAM usage in the reports

4. **"file" command not found (Windows)**
   - The code will still work but with reduced file type detection
   - Consider using WSL or Git Bash for better compatibility

5. **Semantic Scholar 404 Errors**
   - Some papers may not be in Semantic Scholar database
   - This is expected and will be logged
   - Empty `references.json` files will be created for these papers

## Example Usage

### Example 1: Single Month Range

```python
START_MONTH = "2023-05"
START_ID = 2001
END_MONTH = "2023-05"
END_ID = 2010
MAX_PARALLELS = 3
```

This processes papers `2305.02001` through `2305.02010`.

### Example 2: Multi-Month Range

```python
START_MONTH = "2023-05"
START_ID = 2001
END_MONTH = "2023-06"
END_ID = 100
MAX_PARALLELS = 3
```

This processes:
- Papers from May 2023 starting at ID 2001 until the last valid ID
- Papers from June 2023 from ID 1 to 100

## Resource Requirements

Based on actual measurements on **Google Colab** with default configuration (3 parallel threads):

- **RAM**: 
  - Baseline: ~0.8 GB
  - Average during processing: ~196.79 MB
  - Peak memory: ~123.39 MB (increment: ~12.19 MB from baseline)
  - Very efficient memory usage, suitable for systems with limited RAM

- **Disk**: 
  - Each paper uses approximately 1.5-2.4 MB after cleanup
  - Peak disk usage per paper: ~2.4 MB (before cleanup)
  - Final disk usage per paper: ~1.5 MB (after removing non-LaTeX files)
  - Total disk usage depends on number of papers processed

- **Network**: Stable internet connection required

- **Time**: 
  - Total Time: ~7.94 hours
  - Average: ~17 seconds per paper
  - Varies with network speed and paper size
  - Processing time includes download, extraction, and reference fetching

> **Note**: These measurements were obtained from Google Colab's free tier. Actual resource usage may vary depending on your local system configuration, network conditions, and hardware specifications.
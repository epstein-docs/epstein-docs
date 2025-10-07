# Epstein Files Archive

An automatically processed, OCR'd, searchable archive of publicly released documents related to the Jeffrey Epstein case.

## About

This project automatically processes thousands of scanned document pages using AI-powered OCR to:
- Extract and preserve all text (printed and handwritten)
- Identify and index entities (people, organizations, locations, dates)
- Reconstruct multi-page documents from individual scans
- Provide a searchable web interface to explore the archive

**This is a public service project.** All documents are from public releases. This archive makes them more accessible and searchable.

## Features

- **Full OCR**: Extracts both printed and handwritten text from all documents
- **Entity Extraction**: Automatically identifies and indexes:
  - People mentioned
  - Organizations
  - Locations
  - Dates
  - Reference numbers
- **Entity Deduplication**: AI-powered merging of duplicate entities (e.g., "Epstein" → "Jeffrey Epstein")
- **AI Document Analysis**: Generates summaries, key topics, key people, and significance for each document
- **Document Reconstruction**: Groups scanned pages back into complete documents
- **Searchable Interface**: Browse by person, organization, location, date, or document type
- **Static Site**: Fast, lightweight, works anywhere

## Project Structure

```
.
├── process_images.py       # Python script to OCR images using AI
├── cleanup_failed.py       # Python script to clean up failed processing
├── deduplicate.py          # Python script to deduplicate entities
├── deduplicate_types.py    # Python script to deduplicate document types
├── analyze_documents.py    # Python script to generate AI summaries
├── requirements.txt         # Python dependencies
├── .env.example            # Example environment configuration
├── downloads/              # Place document images here
├── results/                # Extracted JSON data per document
├── processing_index.json   # Processing progress tracking (generated)
├── dedupe.json             # Entity deduplication mappings (generated)
├── dedupe_types.json       # Document type deduplication mappings (generated)
├── analyses.json           # AI document analyses (generated)
├── src/                    # 11ty source files for website
├── .eleventy.js            # Static site generator configuration
└── _site/                  # Generated static website (after build)
```

## Setup

### 1. Install Dependencies

**Python (for OCR processing):**
```bash
pip install -r requirements.txt
```

**Node.js (for website generation):**
```bash
npm install
```

### 2. Configure API

Copy `.env.example` to `.env` and configure your OpenAI-compatible API endpoint:

```bash
cp .env.example .env
# Edit .env with your API details
```

### 3. Process Documents

Place document images in the `downloads/` directory, then run:

```bash
python process_images.py

# Options:
# --limit N          # Process only N images (for testing)
# --workers N        # Number of parallel workers (default: 5)
# --no-resume        # Process all files, ignore index
```

The script will:
- Process each image through the OCR API
- Extract text, entities, and metadata
- Save results to `./results/{folder}/{imagename}.json`
- Track progress in `processing_index.json` (resume-friendly)
- Log failed files for later cleanup

**If processing fails or you need to retry failed files:**
```bash
# Check for failures (dry run)
python cleanup_failed.py

# Remove failed files from processed list (so they can be retried)
python cleanup_failed.py --doit

# Also delete corrupt JSON files
python cleanup_failed.py --doit --delete-invalid-json
```

### 4. Deduplicate Entities (Optional but Recommended)

The LLM may extract the same entity with different spellings (e.g., "Epstein", "Jeffrey Epstein", "J. Epstein"). Run the deduplication script to merge these:

```bash
python deduplicate.py

# Options:
# --batch-size N     # Process N entities per batch (default: 50)
# --show-stats       # Show deduplication stats without processing
```

This will:
- Scan all JSON files in `./results/`
- Use AI to identify duplicate entities across people, organizations, and locations
- Create a `dedupe.json` mapping file
- The website build will automatically use this mapping

**Example dedupe.json:**
```json
{
  "people": {
    "Epstein": "Jeffrey Epstein",
    "J. Epstein": "Jeffrey Epstein",
    "Jeffrey Epstein": "Jeffrey Epstein"
  },
  "organizations": {...},
  "locations": {...}
}
```

**Deduplicate Document Types:**

The LLM may also extract document types with inconsistent formatting (e.g., "deposition", "Deposition", "DEPOSITION TRANSCRIPT"). Run the type deduplication script:

```bash
python deduplicate_types.py
```

This will:
- Collect all document types from `./results/`
- Use AI to merge similar types into canonical forms
- Create a `dedupe_types.json` mapping file
- The website build will automatically use this mapping

**Example dedupe_types.json:**
```json
{
  "stats": {
    "original_types": 45,
    "canonical_types": 12,
    "reduction_percentage": 73.3
  },
  "mappings": {
    "deposition": "Deposition",
    "DEPOSITION": "Deposition",
    "deposition transcript": "Deposition",
    "court filing": "Court Filing"
  }
}
```

### 5. Analyze Documents (Optional but Recommended)

Generate AI summaries and insights for each document:

```bash
python analyze_documents.py

# Options:
# --limit N          # Analyze only N documents (for testing)
# --force            # Re-analyze all documents (ignore existing)
```

This will:
- Group pages into documents (matching the website logic)
- Send each document's full text to the AI
- Generate summaries, key topics, key people, and significance analysis
- Save results to `analyses.json`
- Resume-friendly (skips already-analyzed documents)

**Example analysis output:**
```json
{
  "document_type": "deposition",
  "key_topics": ["Flight logs", "Private aircraft", "Passenger manifests"],
  "key_people": [
    {"name": "Jeffrey Epstein", "role": "Aircraft owner"}
  ],
  "significance": "Documents flight records showing passenger lists...",
  "summary": "This deposition contains testimony regarding..."
}
```

### 6. Generate Website

Build the static site from the processed data:

```bash
npm run build    # Build static site to _site/
npm start        # Development server with live reload
```

The build process will automatically:
- Apply deduplication if `dedupe.json` exists
- Load document analyses if `analyses.json` exists
- Generate a searchable analyses page

## How It Works

1. **Document Processing**: Images are sent to an AI vision model that extracts:
   - All text in reading order
   - Document metadata (page numbers, document numbers, dates)
   - Named entities (people, orgs, locations)
   - Text type annotations (printed, handwritten, stamps)

2. **Document Grouping**: Individual page scans are automatically grouped by document number and sorted by page number to reconstruct complete documents

3. **Static Site Generation**: 11ty processes the JSON data to create:
   - Index pages for all entities
   - Individual document pages with full text
   - Search and browse interfaces

## Performance

- Processes ~2,000 pages into ~400 multi-page documents
- Handles LLM inconsistencies in document number formatting
- Resume-friendly processing (skip already-processed files)
- Parallel processing with configurable workers

## Contributing

This is an open archive project. Contributions welcome:
- Report issues with OCR accuracy
- Suggest UI improvements
- Add additional document sources
- Improve entity extraction

## Deployment

The site is automatically deployed to GitHub Pages on every push to the main branch.

### GitHub Pages Setup

1. Push this repository to GitHub: `https://github.com/epstein-docs/epstein-docs.github.io`
2. Go to Settings → Pages
3. Source: GitHub Actions
4. The workflow will automatically build and deploy the site

The site will be available at: `https://epstein-docs.github.io/`

## Future: Relationship Graphs

Once entities are deduplicated, the next step is to visualize relationships between people, organizations, and locations. Potential approaches:

### Static Graph Generation

1. **Pre-generate graph data** during the build process:
   - Build a relationships JSON file showing connections (e.g., which people appear in the same documents)
   - Generate D3.js/vis.js compatible graph data
   - Include in static site for client-side rendering

2. **Graph types to consider**:
   - **Co-occurrence network**: People who appear together in documents
   - **Document timeline**: Documents plotted by date with entity connections
   - **Organization membership**: People connected to organizations
   - **Location network**: People and organizations connected by locations

3. **Implementation ideas**:
   - Use D3.js force-directed graph for interactive visualization
   - Use Cytoscape.js for more complex network analysis
   - Generate static SVG graphs for each major entity
   - Add graph pages to the 11ty build (e.g., `/graphs/people/`, `/graphs/timeline/`)

### Data Structure for Graphs

```json
{
  "nodes": [
    {"id": "Jeffrey Epstein", "type": "person", "doc_count": 250},
    {"id": "Ghislaine Maxwell", "type": "person", "doc_count": 180}
  ],
  "edges": [
    {"source": "Jeffrey Epstein", "target": "Ghislaine Maxwell", "weight": 85, "shared_docs": 85}
  ]
}
```

The deduplication step is essential for accurate relationship mapping - without it, "Epstein" and "Jeffrey Epstein" would appear as separate nodes.

## Disclaimer

This is an independent archival project. Documents are sourced from public releases. The maintainers make no representations about completeness or accuracy of the archive.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

The code in this repository is open source and free to use. The documents themselves are public records.

**Repository**: https://github.com/epstein-docs/epstein-docs

## Support This Project

If you find this archive useful, consider supporting its maintenance and hosting:

**Bitcoin**: `bc1qmahlh5eql05w30cgf5taj3n23twmp0f5xcvnnz`
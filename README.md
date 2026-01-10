# US Building Codes Dataset Pipeline

This repository contains a comprehensive, multi-stage pipeline designed to scrape, parse, and enrich US building codes (IBC, IRC, IEBC) and ADA standards. By combining programmatic parsing with LLM-based (OpenAI/Gemini) inference, this project transforms complex, hierarchical web data into structured, analysis-ready datasets.

## 🏗 Data Pipeline Architecture

The repository employs a **"Base-to-State"** enrichment strategy. This maximizes data quality while minimizing API costs by using the GSA (General Services Administration) baseline as a semantic template for state-specific variations.

### 1. Extraction & Initial Parse

* **Source**: Targets URLs defined in `upcodes-links.txt`.
* **Storage**: Raw data is captured as JSON in `state-codes/` and `gsa-codes/`.
* **Standard Parsing**: Extracts fundamental fields (ID, Section, Title, Body) into clean CSVs without AI intervention to establish the structural backbone.

### 2. Base Enrichment (GSA Template)

* **LLM Inference**: Heavy AI analysis is performed **only** on `gsa-codes`.
* **Metadata Generation**: The AI identifies IFC types, occupancy groups, design phases, and professional responsibilities.
* **Result**: Creates an "AI Baseline" in `ai-codes/` that serves as a lookup table for state codes.

### 3. State-Level Merger (`download/` folder)

* **Strict Merge Logic**: State-specific sections are mapped to the GSA baseline using `id` and `title` matching.
* **Deviation Handling**: If a state section's title deviates from the GSA base (indicating a state-specific amendment), the metadata is cleared to prevent false categorization.

### 4. Final Refinement (`download-v2/` folder)

* **Cleaning**: Removes empty/null rows and standardizes data types.
* **Spot-Inference**: Runs a targeted LLM pass only on rows missing metadata (amendments), ensuring 100% coverage with minimal token burn.

## 📁 Repository Structure

| Folder | Description |
| --- | --- |
| `state-codes/` | Raw JSON and standard parsed CSVs for all 50 states. |
| `gsa-codes/` | Baseline unamended building codes (The "Source of Truth"). |
| `ai-codes/` | Semantic metadata extracted from GSA codes via LLM. |
| `ada-codes/` | Scraped and AI-enhanced ADA Standards for Accessible Design. |
| `download/` | First-pass merged datasets (State text + GSA metadata). |
| `download-v2/` | Production-ready dataset with deduplicated entries and hashed IDs. |

## 🧬 Technical Specifications

### 1. AI Enrichment Schema (Pydantic)

We use **Pydantic V2** to enforce strict typing and enum-constrained values during LLM inference. This ensures that values like `occupancy` or `ifc_type` are always machine-readable and consistent.

```python
class RuleAnalysis(BaseModel):
    ifc_type: List[str] = Field(
        description="1-5 building element types from the allowed IFC list."
    )
    occupancy: List[str] = Field(
        description="1-5 IBC occupancy groups (e.g., 'Residential (R)')."
    )
    design_phase: List[str] = Field(
        description="Project phases: ['concept', 'sd', 'dd', 'cd', 'ca']."
    )
    code_category: CodeCategoryEnum = Field(
        description="The single best thematic category for this rule."
    )
    primary_responsibility: ResponsibilityEnum = Field(
        description="The primary discipline responsible for compliance."
    )

```

### 2. Final CSV Output Schema

The production CSVs in `download-v2/` use the following structure. Multi-value fields are pipe-delimited (`|`) to maintain CSV integrity.

| Column | Data Type | Description |
| --- | --- | --- |
| `id` | `string` | The section reference number (e.g., `305.1`). |
| `section` | `string` | Hierarchy path (e.g., `Chapter 3 > Section 305`). |
| `title` | `string` | The provision name. |
| `ifc_type` | `string` | Pipe-separated IFC classes (e.g., `Wall|Slab`). |
| `occupancy` | `string` | Affected occupancy groups. |
| `design_phase` | `string` | Relevant project phases. |
| `code_category` | `string` | High-level rule classification. |
| `primary_responsibility` | `string` | Lead professional discipline. |
| `body` | `string` | Cleaned, plain-text code content. |

## 🛠 Features

* **Cost Optimization**: Reduces LLM API costs by >80% by focusing AI inference on base codes.
* **Deduplication**: Uses MD5 hashing of the `body` text to create unique `call_id` keys for database ingestion.
* **Search Engine Ready**: Includes scripts to convert CSVs into **JSON Lines (JSONL)** format for Elasticsearch Bulk API ingestion.
* **High Scannability**: Cleans HTML noise, excessive whitespace, and non-breaking spaces for better RAG (Retrieval-Augmented Generation) performance.


## 🚀 Usage

1. **Configure API**: Set `gemini_api` or `openai_api` in your environment.
2. **Run Scraper**: Populate `gsa-codes/` with the target year's baseline.
3. **Generate Baseline**: Execute the AI Parse script on the GSA folder.
4. **Batch Process States**: Run `process_state_codes()` to map metadata across the `state-codes/` directory.
5. **Finalize**: Run the `download-v2` script to clean and perform spot-inference on amendments.

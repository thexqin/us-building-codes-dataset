# 🇺🇸 us-building-codes-dataset

**Creating a Structured, Machine-Readable Dataset of US Building Codes**

This repository provides a complete, robust Python pipeline for converting raw US building code content from **up.codes** into a structured, compliance-checking ready dataset. The final output is an Elasticsearch-ready JSON file, making the data directly queryable for AI and BIM compliance tools, such as the [cuniform](https://github.com/cuniform) project.

## 🚀 The Three-Stage Pipeline

The project executes a full pipeline to transform raw, embedded JSON into fully analyzed and indexed data. 

| Stage | Process | Output | Key Technology |
| :--- | :--- | :--- | :--- |
| **1. Acquisition** | Scrapes embedded JSON content from `up.codes` URLs, handling different code versions (IBC, IRC, IEBC). | Raw JSON files (per chapter) | Python, `requests`, `BeautifulSoup` |
| **2. Structuring** | **Python/Regex** recursion to determine rule hierarchy, clean HTML, and extract plain text. | Structured CSV file (per chapter) | JSON Parsing, Regex |
| **3. Abstraction & Indexing** | **LLM (OpenAI/Gemini)** performs structured Pydantic extraction and converts the final data into an indexable format. | Elasticsearch JSON (Bulk API) | Pydantic, LLM APIs, MD5 Hashing |

## ✨ AI-Powered Rule Abstraction & Data Fields

The most critical step, Stage 3, utilizes Large Language Models (LLMs) to enrich the data with fields essential for automated compliance checking. The final CSV and JSON documents contain the following fields:

| Field | Description | Type | Purpose in BIM Check |
| :--- | :--- | :--- | :--- |
| **`section`** | The top-level section title (e.g., *Section 202 Definitions*). | Text | High-level rule categorization. |
| **`reference`** | The specific numerical or alphabetical reference (e.g., `2.202.1`). | Text | Hierarchical rule addressing. |
| **`display_title`** | The title of the current sub-section or rule. | Text | Local context and identification. |
| **`depth`** | The hierarchical nesting level of the rule (0 is highest). | Integer | Determines rule priority/specificity. |
| **`body`** | The cleaned, plain-text content of the rule. | Text | The searchable rule text. |
| **`ifc_type`** | The relevant IFC elements (e.g., `Wall`, `Slab`, `Door`). | List | Filters rules based on BIM element type. |
| **`occupancy`** | Applicable IBC Occupancy Groups (e.g., `Assembly (A)`, `Residential (R)`). | List | Filters rules based on project occupancy. |
| **`design_phase`** | When the rule is addressed (e.g., `concept`, `cd`). | List | Filters rules based on project status. |
| **`code_category`** | Primary code domain (e.g., `means_of_egress`). | Single Value | Topic-based rule classification. |
| **`primary_responsibility`** | The discipline responsible for compliance (e.g., `architect`, `structural_engineer`). | Single Value | Assigning compliance ownership. |

The final output also includes a **`call_id`** field, which is an MD5 hash of the rule body, crucial for deduplication and consistent indexing across different code versions.

## 🛠️ Execution & Development

The pipeline is implemented using Python and is designed for reproducible execution, typically within a notebook environment like Colab.

### Key Scripts

* **`json batch downloader`**: Block that scrapes raw JSON data from a list of specified code URLs.
* **`process_file()`**: Executes the Python/Regex-based structuring and initiates the LLM calls.
* **`convert_csv_to_es_rules()`**: Prepares the final, indexed JSON output in Elasticsearch Bulk API format.
* **`output folder`**: The main execution block using a **Thread Pool Executor** to run the full pipeline on a specified range of code chapters concurrently.

## 🔗 Related Project

This highly-structured dataset forms the core rule base for the efforts by the [cuniform](https://github.com/cuniform) organization, which develops tools for automated code compliance checking in IFC models.

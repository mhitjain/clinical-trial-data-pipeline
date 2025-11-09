# Clinical Trial & SEC Filings Enrichment Pipeline

## Objective

Build a pipeline to extract and enrich data for industry-sponsored interventional clinical trials in a specified condition (e.g., Pulmonary Arterial Hypertension) from:

- ClinicalTrials.gov (trial metadata)
- SEC filings (10-K, 8-K from public biotech sponsors)
- Public documents (presentations, publications - planned)

The pipeline structures the output into JSON and PostgreSQL-compatible format, including trial arms, interventions, endpoints, and baseline characteristics.

## Workflow Overview

### Step 1: Fetch Trials from ClinicalTrials.gov

- API: https://clinicaltrials.gov/api/v2/studies
- Filters:
  - condition: e.g., Pulmonary Arterial Hypertension
  - study_type: Interventional
  - start_date: Within the last 10 years
- Process:
  - Paginate through results (up to 100 per page)
  - Parse `startDateStruct.date` into datetime object
  - Sort studies by start date (descending)
  - Store in `sorted_studies`

### Step 2: Filter Trials Further

Apply 4 key filters:
1. sponsor_type = INDUSTRY
2. start_date >= 10 years ago
3. condition explicitly matches the query condition
4. Sponsor is a public biotech company (verified via SEC API)

If all conditions are met:
- Append study to `filtered[]`

## SEC API Integration (Public Sponsor Verification)

### Step 3: Search SEC Filings

- API: https://api.sec-api.io (Full-Text Search API)
- Query payload:
```json
{
  "query": "companyName:\"<SPONSOR>\" AND (formType:\"10-K\" OR formType:\"8-K\")",
  "from": 0,
  "size": 50,
  "sort": [{ "filedAt": { "order": "desc" }}]
}
```
- Input: company_name from the trial’s lead sponsor
- If filings are found, sponsor is assumed public

### Step 4: Save SEC Results

- File path: `edgar_results/<safe_company_name>.json`
- Contents:
  - Filing date
  - Form type
  - Filing detail URL
- Stores corresponding trial and SEC data to:
  - `filtered[]` for trials
  - `filtered_filings[]` for SEC responses


### Step 5: Extract Filings for Trial Mentions

- Use SEC Filing Text or PDF APIs
- Limit to 2–4 recent 10-K/8-K per company
- Parse HTML using BeautifulSoup
- Extract sections mentioning:
  - Trial title or NCT ID
  - Drug-related updates
  - Outcomes, endpoints, safety metrics

### Step 6: Enrich Trial Metadata via LLM 

- Use OpenAI to extract:
  - Drug name, dose, frequency, formulation
  - Primary and secondary endpoints



- **Example Output:**
```json
"matched_publication": {
  "title": "Interim Results from NAB-Sirolimus in PAH Patients",
  "doi": "https://doi.org/10.1016/j.healun.2019.01.1238",
  "source": "CrossRef",
  "gpt_match": "Yes",
  "reason": "The drug, condition, and trial phase match; sponsor is not explicitly stated but aligns contextually."
}

### Step 7: Final JSON Output

Final structured JSON format:
```json
{
  "clinical_study": {
    "title": "",
    "nct_identifier": "",
    "indication": "",
    "intervention": "",
    "interventional_drug": {
      "name": "",
      "dose": "",
      "frequency": "",
      "formulation": ""
    },
    "study_arms": {
      "intervention": 0,
      "placebo": 0
    },
    "number_of_participants": 0,
    "average_age": 0,
    "age_range": [0, 0],
    "endpoints": [],
    "baseline_characteristics": []
  },
  "endpoints": [
    {
      "name": "",
      "description": "",
      "timepoint": "",
      "arm": "intervention",
      "average_value": null,
      "upper_end": null,
      "lower_end": null,
      "statistical_significance": ""
    }
  ],
  "baseline_measures": [
    {
      "name": "",
      "description": "",
      "arm": "intervention",
      "average_value": null,
      "upper_end": null,
      "lower_end": null
    }
  ]
}
```

This structure aligns with the target PostgreSQL schema with three tables:
- clinical_study
- endpoints
- baseline_measures

## Planned Features

### Step 8: Proof-of-Concept — Publication Matching via GPT-4

**Objective:** Identify whether a scientific publication likely reports on the same clinical trial using metadata + GPT-4 reasoning.

- **Input:**
  - Trial title, NCT ID, sponsor
  - Publications from scholarly APIs (CrossRef, PubMed, Semantic Scholar, OpenAlex)

- **Method:**
  - Query each source by trial title
  - Retrieve top 3 publication candidates
  - Construct a GPT-4 prompt including:
    - Trial metadata
    - Publication title and abstract
  - GPT-4 returns a match decision and rationale

- **Status:**  
  Proof-of-concept implemented and tested on selected trials (e.g., nab-sirolimus in PAH).  
  Handles API rate limits (`429`) and forbidden errors (`403`) with retry and backoff logic.

- **Planned Integration:**
  - Add matched publication metadata to final output
  - Export DOI and venue with each trial, if matched



## API Summary

| API                     | Purpose                     | Endpoint                                 |
|-------------------------|-----------------------------|------------------------------------------|
| ClinicalTrials.gov      | Trial metadata              | https://clinicaltrials.gov/api/v2/studies     |
| SEC Full-Text Search    | Verify public sponsor status| https://api.sec-api.io                    |
| SEC Filing Text/PDF     | Filing content parsing      | https://api.sec-api.io/filing-text/        |
| OpenAI API              | Metadata enrichment         | https://api.openai.com                       |

## Setup Instructions

### 1. Environment Variables

Create a `.env` file with:
```
API_KEY=<your-sec-api.io-key>
OPENAI_KEY=<your-openai-key>
DB_HOST=localhost
DB_PORT=5432
DB_NAME=your_database_name
DB_USER=your_username
DB_PASSWORD=your_password
```

### 2. Install Dependencies

```bash
pip install requests python-dotenv beautifulsoup4 openai
```
### 2. Install Dependencies

```bash
brew install postgresql
brew services start postgresql
```

## How to Change Condition

Modify this line in the script:
```python
condition = "Pulmonary Arterial Hypertension"
```

## Output Artifacts

- `edgar_results/*.json`: SEC filings for each sponsor
- `filtered[]`: trial objects with verified public sponsors
- `filtered_filings[]`: raw SEC response data
- Final JSON output : structured per PostgreSQL schema

## Known Limitations  & Workarounds
 ### ClinicalTrials.gov API
- No field to distinguish public vs. private sponsors 
- sponsor_type and similar filters are not queryable
- Applied available filters (condition, study_type, start_date)
- Verified sponsor status using SEC API fetch

### SEC Data
- Can only search by company name, not by trial name
- Sponsor name mismatches may cause missed filings
- Many filings don’t reference trial titles or NCT IDs
- SEC API usage is credit-limited and potentially expensive at scale


### Data & Extraction
- Trial registry entries often lack detailed endpoint data
- LLM enrichment (planned) requires fine-tuning and validation
- Some fields (e.g., drug details, baseline stats) need manual/AI extraction


### Known publication data API Limitations

| API                    | Known Limitations                                                                 |
|------------------------|------------------------------------------------------------------------------------|
| **CrossRef API**       | - Returns many publications with similar but unrelated titles<br>- Abstracts not always available<br>- Limited filtering (no NCT ID or intervention support) |
| **PubMed API**         | - Search by title may miss relevant publications due to different naming<br>- Abstracts not included in `esummary` (requires extra step)<br>- No direct sponsor matching |
| **Semantic Scholar**   | - Strict rate limits without API key (429 errors)<br>- Abstracts may be truncated or missing<br>- Relevance ranking can return off-topic results |
| **OpenAlex**           | - 403 Forbidden errors if `User-Agent` is not set<br>- No full abstracts (only inverted index)<br>- Some newer publications may be missing or delayed |
| **OpenAI GPT-4 API**   | - Requires paid access<br>- Can hallucinate if inputs are vague or mismatched<br>- Cost increases with large batch evaluations |

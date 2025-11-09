# Clinical Trial Data Pipeline

A comprehensive data pipeline for extracting, enriching, and analyzing clinical trial data from ClinicalTrials.gov, SEC filings, and scientific publications. This pipeline focuses on industry-sponsored interventional trials with automated data extraction, enrichment using AI, and structured storage in PostgreSQL.

## 🎯 Overview

This pipeline:
1. **Fetches** clinical trials from ClinicalTrials.gov API filtered by condition and study type
2. **Verifies** public company sponsors using SEC filing data
3. **Extracts** relevant 8-K and 10-K SEC filings mentioning trial data
4. **Enriches** trial metadata using OpenAI GPT-4 (drug information, endpoints)
5. **Structures** data into standardized JSON format
6. **Stores** data in PostgreSQL database with relational schema
7. **Matches** trials with published research papers (proof-of-concept)

## 📋 Features

- ✅ Automated trial data extraction from ClinicalTrials.gov
- ✅ Public company verification via SEC API
- ✅ SEC filing download and keyword matching
- ✅ AI-powered metadata extraction (OpenAI GPT-4)
- ✅ Structured data output (JSON + PostgreSQL)
- ✅ Baseline characteristics and endpoint analysis
- ✅ Publication matching (proof-of-concept)
- ✅ Support for multiple companies and trials

## 🏗️ Architecture

```
ClinicalTrials.gov API → Filter Trials → Verify Public Sponsor (SEC API)
                              ↓
                    Download SEC Filings (8-K, 10-K)
                              ↓
                    Extract Trial Mentions (Keywords)
                              ↓
                    Enrich Metadata (OpenAI GPT-4)
                              ↓
                    Structure Data (JSON Schema)
                              ↓
                    Store in PostgreSQL Database
```

## 📁 Project Structure

```
clinical-trial-data-pipeline/
├── pipeline.ipynb              # Main Jupyter notebook with full pipeline
├── documentation.md            # Detailed technical documentation
├── README.md                   # This file
├── .env                        # Environment variables (not in repo)
├── edgar_results/              # SEC filing metadata by company
├── trail_results/              # Raw trial data from ClinicalTrials.gov
├── formatted_trials/           # Structured trial JSON output
├── forms/                      # Downloaded SEC filing PDFs
│   └── {NCT_ID}/
│       ├── 8K/                 # 8-K filings
│       └── 10K/                # 10-K filings
├── parsed_chunks/              # Parsed filing content (tables, chunks)
├── parsed_chunks_json/         # JSON chunks from filings
└── matched_pmc_publications.json # Matched publications
```

## 🚀 Getting Started

### Prerequisites

- Python 3.8+
- PostgreSQL 12+
- API Keys:
  - SEC API (https://sec-api.io)
  - OpenAI API (https://platform.openai.com)
  - SerpAPI (https://serpapi.com) - optional for publication search

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/mhitjain/clinical-trial-data-pipeline.git
   cd clinical-trial-data-pipeline
   ```

2. **Install Python dependencies**
   ```bash
   pip install requests python-dotenv beautifulsoup4 openai psycopg2-binary sqlalchemy pandas jupyter serpapi
   ```

3. **Install PostgreSQL** (if not already installed)
   
   macOS:
   ```bash
   brew install postgresql@14
   brew services start postgresql@14
   ```
   
   Linux (Ubuntu/Debian):
   ```bash
   sudo apt update
   sudo apt install postgresql postgresql-contrib
   sudo systemctl start postgresql
   ```

4. **Create PostgreSQL database**
   ```bash
   # Login to PostgreSQL
   psql postgres
   
   # Create database and user
   CREATE DATABASE clinical_trials;
   CREATE USER your_username WITH PASSWORD 'your_password';
   GRANT ALL PRIVILEGES ON DATABASE clinical_trials TO your_username;
   \q
   ```

5. **Configure environment variables**
   
   Create a `.env` file in the project root:
   ```env
   # SEC API
   API_KEY=your_sec_api_key_here
   
   # OpenAI API
   OPENAI_API_KEY=your_openai_api_key_here
   
   # SerpAPI (optional - for publication search)
   SERPAPI_API_KEY=your_serpapi_key_here
   
   # PostgreSQL Database
   dbname=clinical_trials
   user=your_username
   password=your_password
   ```

### Running the Pipeline

1. **Start Jupyter Notebook**
   ```bash
   jupyter notebook
   ```

2. **Open `pipeline.ipynb`**
   
   Navigate to http://localhost:8888 in your browser and open `pipeline.ipynb`

3. **Configure the condition** (Optional)
   
   Edit the condition variable in the second cell:
   ```python
   condition = "Pulmonary Arterial Hypertension"  # Change to your condition
   ```

4. **Run all cells sequentially**
   
   You can run all cells using `Cell > Run All` or execute them one by one to see the progress.

### Pipeline Execution Steps

The notebook executes the following steps in order:

1. **Fetch Trials** - Retrieves trials from ClinicalTrials.gov API
2. **Filter by Sponsor** - Keeps only industry-sponsored, public companies
3. **Download SEC Filings** - Fetches 8-K and 10-K forms
4. **Extract Trial Mentions** - Searches filings for trial keywords
5. **Enrich with AI** - Uses GPT-4 to extract structured drug/endpoint data
6. **Format Output** - Creates standardized JSON structure
7. **Store in Database** - Inserts data into PostgreSQL tables

## 📊 Database Schema

The pipeline creates three related tables in PostgreSQL:

### `clinical_study` (Main Table)
```sql
- id (PRIMARY KEY)
- title
- nct_identifier
- indication
- intervention
- drug_name, drug_dose, drug_frequency, drug_formulation
- arm_intervention, arm_placebo
- number_of_participants
- average_age
- age_range
```

### `endpoints` (One-to-Many with clinical_study)
```sql
- id (PRIMARY KEY)
- study_id (FOREIGN KEY → clinical_study.id)
- name, description, timepoint
- arm
- average_value, upper_end, lower_end
- statistical_significance
```

### `baseline_measures` (One-to-Many with clinical_study)
```sql
- id (PRIMARY KEY)
- study_id (FOREIGN KEY → clinical_study.id)
- name, description
- arm
- average_value, upper_end, lower_end
```

## 📄 Output Format

Each trial produces a JSON file with this structure:

```json
{
  "clinical_study": {
    "title": "Study Title",
    "nct_identifier": "NCT05036135",
    "indication": "Pulmonary Arterial Hypertension",
    "intervention": "Drug Name",
    "interventional_drug": [
      {
        "name": "Imatinib",
        "dose": "400 mg",
        "frequency": "once daily",
        "formulation": "tablet"
      }
    ],
    "study_arms": {
      "intervention": 1,
      "placebo": 1
    },
    "number_of_participants": 150,
    "average_age": null,
    "age_range": ["18 Years", "75 Years"],
    "endpoints": [...],
    "baseline_characteristics": [...]
  },
  "endpoints": [...],
  "baseline_measures": [...]
}
```

## 🔧 Configuration

### Customizing the Condition

Edit the `condition` variable in cell 2:
```python
condition = "Your Disease/Condition Here"
```

### Adjusting the Time Range

Modify the cutoff date (default is 10 years):
```python
cutoff_date = (datetime.today() - timedelta(days=365 * 10)).date()  # Change 10 to your desired years
```

### Limiting Results

Change the number of trials to process (default is 5):
```python
if len(filtered) == 5:  # Change 5 to desired number
    break
```

### Number of SEC Filings

Adjust the number of 8-K and 10-K filings to download per trial:
```python
NUM_FILINGS = 4  # Change to desired number
```

## 🔑 API Usage and Limits

| API | Purpose | Rate Limits | Cost |
|-----|---------|-------------|------|
| ClinicalTrials.gov | Trial data | No strict limit | Free |
| SEC API | Filing search & download | Based on credits | Paid ($49+/month) |
| OpenAI GPT-4 | Metadata extraction | Token-based | Pay-per-use |
| SerpAPI | Publication search | 100 searches/month (free tier) | Free/Paid |

## 🧪 Example Use Cases

1. **Pharmaceutical Research**: Track competitor trials and drug development
2. **Investment Analysis**: Monitor biotech company trial progress via SEC filings
3. **Academic Research**: Analyze trial design patterns and outcomes
4. **Regulatory Compliance**: Track trial reporting in public disclosures
5. **Meta-Analysis**: Aggregate endpoint data across similar trials

## 📚 Dependencies

```
requests>=2.28.0
python-dotenv>=0.21.0
beautifulsoup4>=4.11.0
openai>=1.0.0
psycopg2-binary>=2.9.0
sqlalchemy>=2.0.0
pandas>=1.5.0
jupyter>=1.0.0
serpapi>=0.1.0
```

## 🐛 Troubleshooting

### Common Issues

**1. PostgreSQL Connection Error**
```
psycopg2.OperationalError: could not connect to server
```
Solution: Ensure PostgreSQL is running:
```bash
brew services start postgresql@14  # macOS
sudo systemctl start postgresql     # Linux
```

**2. SEC API Rate Limit**
```
Error 429: Too many requests
```
Solution: The pipeline includes retry logic with delays. If persistent, upgrade your SEC API plan.

**3. OpenAI API Errors**
```
openai.error.RateLimitError
```
Solution: Check your API quota and billing at https://platform.openai.com/account/usage

**4. Missing Environment Variables**
```
ValueError: Missing API_KEY in environment variables
```
Solution: Ensure your `.env` file is in the project root with all required keys.

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## ⚠️ Limitations

- **Sponsor Verification**: ClinicalTrials.gov doesn't distinguish public vs private companies; requires SEC API lookup
- **Filing Content**: Many SEC filings don't explicitly mention trial names or NCT IDs
- **Data Completeness**: Endpoint values often require results to be posted on ClinicalTrials.gov
- **API Costs**: Heavy usage of SEC and OpenAI APIs can be expensive
- **Rate Limits**: Multiple API dependencies each have their own rate limits

## 🔮 Future Enhancements

- [ ] Automated publication matching and PDF extraction
- [ ] Enhanced endpoint value extraction from results
- [ ] Support for additional filing types (S-1, 20-F)
- [ ] Real-time trial monitoring and alerts
- [ ] Web dashboard for data visualization
- [ ] Support for more data sources (WHO ICTRP, EU CTR)
- [ ] Fine-tuned LLM for medical entity extraction

## 📧 Contact

For questions or issues, please open an issue on GitHub or contact the maintainer.

## 🙏 Acknowledgments

- ClinicalTrials.gov API for trial data
- SEC API for filing access
- OpenAI for GPT-4 capabilities
- PostgreSQL community

---

**Note**: This pipeline is for research and educational purposes. Always verify data accuracy and comply with API terms of service.
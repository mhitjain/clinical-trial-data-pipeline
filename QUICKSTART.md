# Quick Start Guide

## 🚀 Get Running in 5 Minutes

### Step 1: Clone and Install
```bash
# Clone the repository
git clone https://github.com/mhitjain/clinical-trial-data-pipeline.git
cd clinical-trial-data-pipeline

# Install dependencies
pip install -r requirements.txt
```

### Step 2: Setup PostgreSQL
```bash
# macOS
brew install postgresql@14
brew services start postgresql@14

# Create database
createdb clinical_trials
```

### Step 3: Configure API Keys
```bash
# Copy example environment file
cp .env.example .env

# Edit .env and add your API keys
nano .env
```

Required API keys:
- **SEC API**: Get from https://sec-api.io (Free trial available)
- **OpenAI API**: Get from https://platform.openai.com
- **SerpAPI** (optional): Get from https://serpapi.com

### Step 4: Run the Pipeline
```bash
# Start Jupyter
jupyter notebook

# Open pipeline.ipynb and run all cells
# Or run specific cells step by step
```

### Step 5: View Results

**Check generated files:**
```bash
ls edgar_results/          # SEC filing metadata
ls formatted_trials/       # Structured trial data
ls forms/                  # Downloaded SEC PDFs
```

**Check database:**
```bash
psql clinical_trials

# View tables
\dt

# Query trials
SELECT title, nct_identifier FROM clinical_study;
```

## 🎯 What Happens When You Run It?

1. ✅ Fetches ~100+ trials for "Pulmonary Arterial Hypertension"
2. ✅ Filters to 5 most recent industry-sponsored trials from public companies
3. ✅ Downloads 8-K and 10-K SEC filings for each sponsor
4. ✅ Extracts trial mentions from filings
5. ✅ Uses GPT-4 to extract drug metadata and endpoints
6. ✅ Saves structured JSON files
7. ✅ Populates PostgreSQL database

**Total runtime**: ~5-10 minutes (depending on API response times)

## 🔧 Customization

### Change the condition:
Edit cell 2 in `pipeline.ipynb`:
```python
condition = "Type 2 Diabetes"  # Or any other condition
```

### Get more/fewer trials:
Edit cell 3 in `pipeline.ipynb`:
```python
if len(filtered) == 10:  # Change from 5 to 10
    break
```

### Adjust filing downloads:
Edit cell 8 in `pipeline.ipynb`:
```python
NUM_FILINGS = 6  # Change from 4 to 6
```

## 📊 Sample Output

**JSON Structure:**
```json
{
  "clinical_study": {
    "title": "Study of Imatinib in PAH",
    "nct_identifier": "NCT05036135",
    "indication": "Pulmonary Arterial Hypertension",
    "interventional_drug": [{
      "name": "Imatinib",
      "dose": "400 mg",
      "frequency": "once daily"
    }]
  },
  "endpoints": [...],
  "baseline_measures": [...]
}
```

## 🐛 Troubleshooting

**PostgreSQL not starting?**
```bash
brew services restart postgresql@14
```

**API key errors?**
- Check your `.env` file has correct keys
- Verify keys are valid on respective platforms

**Import errors?**
```bash
pip install -r requirements.txt --upgrade
```

## 📚 Next Steps

- Explore `documentation.md` for detailed technical info
- Check `README.md` for comprehensive documentation
- Modify condition and parameters to explore different diseases
- Query the PostgreSQL database for analysis

## 💡 Tips

1. **Start small**: Test with 1-2 trials first
2. **Monitor costs**: OpenAI and SEC API have usage charges
3. **Check rate limits**: Add delays if hitting API limits
4. **Validate data**: Always verify AI-extracted data accuracy

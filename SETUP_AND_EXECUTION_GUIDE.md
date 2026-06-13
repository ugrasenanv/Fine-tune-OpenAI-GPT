# Complete Setup & Execution Guide

## 🚀 Getting Started

### Prerequisites
- Python 3.8+
- OpenAI API key
- VS Code with Jupyter extension
- Required packages (installed in notebooks)

---

## 1️⃣ Setup Your Environment

### Get OpenAI API Key

1. Go to [OpenAI Platform](https://platform.openai.com)
2. Sign up or log in
3. Click **API keys** in the left menu
4. Click **Create new secret key**
5. Copy the key (save it safely!)

### Set Environment Variable

**On Linux/Mac**:
```bash
export OPENAI_API_KEY='your-api-key-here'
```

**On Windows (PowerShell)**:
```powershell
$env:OPENAI_API_KEY='your-api-key-here'
```

**Or create a `.env` file**:
```
OPENAI_API_KEY=your-api-key-here
```

Then in notebook:
```python
import os
from dotenv import load_dotenv
load_dotenv()
```

---

## 2️⃣ Prepare Your Data Files

### Required Files
```
/workspaces/Fine-tune-OpenAI-GPT/
├── bbc.csv                    ← News articles dataset
├── test.jsonl                 ← Training data (JSONL format)
├── dataset/
│   └── test.csv               ← Test data for evaluation
```

### Check Your Files

**bbc.csv should have**:
```csv
text,category
"Article content here",business
"Another article",tech
"Sports news",sport
```

**test.jsonl should have**:
```json
{"messages": [{"role": "user", "content": "Article..."}, {"role": "assistant", "content": "business"}]}
{"messages": [{"role": "user", "content": "Article..."}, {"role": "assistant", "content": "tech"}]}
```

---

## 3️⃣ Run the Notebooks in Order

### Phase 1: Data Exploration (EDA) ⏱️ 10-15 minutes

**Run ONE of these**:
```
Start with: 1.2 eda.ipynb
   ↓
(Optional) Run: 2.2 eda.ipynb
(Optional) Run: 3.2 eda.ipynb
(Optional) Run: 4.1 eda.ipynb
(Optional) Run: 6.1 eda.ipynb
```

**What to do**:
1. Open notebook in VS Code
2. Click **"Run All"** or run cell by cell (Shift+Enter)
3. Check output for any errors
4. Verify data looks good

**Expected output**:
```
Dataset shape: (2000, 2)
Categories:
business        510
entertainment   386
politics        417
sport           511
tech            401
```

---

### Phase 2: Fine-Tuning ⏱️ 5-30 minutes

**Run in this order**:
```
7.1 fine-tune.ipynb  ← First model
   ↓
Wait for training to complete
   ↓
(Optional) 9.1 fine-tune.ipynb  ← Second attempt
(Optional) 10.1 fine-tune.ipynb ← Third attempt
```

**What to do**:
1. Open `7.1 fine-tune.ipynb`
2. Run cell by cell:
   - Cell 1: Initialize OpenAI client
   - Cell 2: Create OpenAI client object
   - Cell 3: Upload training file
   - Cell 4: List uploaded files
   - Cell 5: Retrieve file details
   - Cell 6: **Create fine-tuning job** (STARTS TRAINING)
   - Cells 7-9: Monitor progress

3. **Copy your Model ID** when training completes
   - Look for: `ft:gpt-3.5-turbo-0125:bis::9tt19es9`

**What you'll see**:
```
Training started!
Job ID: ftjob-ku7ytdtAEdx5jxthRANQTqr0
Status: queued → validating_files → running → succeeded
Time: ~5-30 minutes
```

**⏳ WAIT** for `status: succeeded` before moving to evaluation!

---

### Phase 3: Evaluation ⏱️ 2-5 minutes

**Run in this order**:
```
11.1 evaluation.ipynb  ← Evaluate first model
(Optional) 12.1 evaluation.ipynb ← Evaluate another model
```

**What to do**:
1. Open `11.1 evaluation.ipynb`
2. Update the model ID in this line:
   ```python
   model="ft:gpt-3.5-turbo-0125:bis::9tt19es9"  # ← Replace with YOUR model ID
   ```
3. Run all cells
4. Check the final accuracy

**Expected output**:
```
Sample predictions: ['business', 'tech', 'sport', ...]
Total classifications: 250
Accuracy: 87.60%
```

---

## 🔍 Detailed: Running a Single Notebook

### Step-by-Step Example: 7.1 fine-tune.ipynb

**1. Open the notebook**:
- In VS Code file explorer, click the notebook file
- It opens in the editor

**2. Check kernel**:
- Top right corner should show "Python [version]"
- If not, click the kernel selector and choose Python

**3. Run Cell 1** (Initialize):
```python
from openai import OpenAI
```
- Press Shift+Enter or click ▶️ Run button
- Should have no output (just imports)

**4. Run Cell 2** (Create client):
```python
client = OpenAI()
```
- Connects to OpenAI API
- If error → API key not set

**5. Run Cell 3** (Upload data):
```python
client.files.create(file=open("test.jsonl","rb"), purpose="fine-tune")
```
- Output shows file ID like: `file-abc123xyz`
- **COPY THIS FILE ID**

**6. Run Cells 4-5** (List & retrieve files):
- Shows your uploaded files
- Verify the file is there

**7. Run Cell 6** (Create job) ⚠️ THIS STARTS TRAINING:
```python
client.fine_tuning.jobs.create(
    model="gpt-3.5-turbo",
    training_file="file-abc123xyz",  # ← Use YOUR file ID
    hyperparameters={"n_epochs": 1},
    validation_file="file-xyz789"     # ← Use YOUR validation file ID
)
```
- Output: Job ID like `ftjob-ku7ytdtAEdx5jxthRANQTqr0`
- **COPY THIS JOB ID**
- Status starts as `queued`

**8. Run Cells 7-9** (Monitor progress):
```python
client.fine_tuning.jobs.retrieve("ftjob-ku7ytdtAEdx5jxthRANQTqr0")
```
- Check status every 5 minutes
- Possible values:
  - `queued` → waiting to start
  - `validating_files` → checking data
  - `running` → actively training
  - `succeeded` → ✅ DONE!
  - `failed` → ❌ error

**9. When succeeded**, run Cell 10 (Test model):
- Replace model ID with your new model ID
- Send test text to model
- See prediction!

---

## 📊 Monitoring Your Jobs

### Check Status Anytime

In a new cell, run:
```python
from openai import OpenAI
client = OpenAI()

job = client.fine_tuning.jobs.retrieve("ftjob-YOUR-JOB-ID")
print(f"Status: {job.status}")
print(f"Model ID: {job.fine_tuned_model}")
```

### View Training Events

```python
events = client.fine_tuning.jobs.list_events(
    fine_tuning_job_id="ftjob-YOUR-JOB-ID",
    limit=10
)
for event in events.data:
    print(f"{event.created_at}: {event.message}")
```

---

## 💾 Saving Important Information

### Create a log file

Create `TRAINING_LOG.txt`:
```
Fine-Tuning Session Log
=======================
Date: 2024-01-15

Model 1:
--------
Training File ID: file-lIyi8uMGNxXxxIGWxUD2S02B
Job ID: ftjob-ku7ytdtAEdx5jxthRANQTqr0
Model ID: ft:gpt-3.5-turbo-0125:bis::9tt19es9
Hyperparameters: n_epochs=1
Accuracy: 87.60%
Status: ✅ Production ready

Model 2:
--------
Training File ID: file-XYZ456
Job ID: ftjob-XYZ789
Status: 🔄 Training (5% complete)
```

---

## 🚨 Troubleshooting

### Issue 1: "File not found" error
```
Error: FileNotFoundError: [Errno 2] No such file or directory: 'bbc.csv'
```
**Fix**:
- Check file exists in correct folder
- Check file path in notebook matches exactly
- Use full absolute path if needed

### Issue 2: "API key not found"
```
Error: AuthenticationError: API key not found
```
**Fix**:
```python
import os
print(os.environ.get('OPENAI_API_KEY'))  # Should show your key, not None
```

### Issue 3: "Invalid JSONL format"
```
Error: Invalid format for training file
```
**Fix**:
- Each line must be valid JSON
- Check file with: `head test.jsonl`
- Validate JSON at [jsonlint.com](https://jsonlint.com)

### Issue 4: Training stuck on "running"
**Fix**:
- This is normal! Can take 10-30 minutes
- Check progress: `client.fine_tuning.jobs.list_events(...)`
- Don't interrupt or close notebook

### Issue 5: Low accuracy
**Fix**:
- Review training data quality
- Try more epochs: `"n_epochs": 2` or 3
- Run notebook 9.1/10.1 with better data
- Check data balance (similar categories count)

---

## ✅ Complete Checklist

Before you start:
- [ ] OpenAI API key set
- [ ] bbc.csv file downloaded
- [ ] test.jsonl prepared
- [ ] test.csv in dataset/ folder
- [ ] VS Code with Python/Jupyter extensions

During execution:
- [ ] Run EDA notebooks (check data)
- [ ] Run fine-tuning notebook (start training)
- [ ] Wait for training to complete
- [ ] Copy model ID
- [ ] Run evaluation notebook (check accuracy)
- [ ] Save model ID to log file

---

## 🎓 Learning Tips

1. **Run slowly**: Don't rush through cells
2. **Read output**: Understand what each cell produces
3. **Take notes**: Write what you learn
4. **Experiment**: Try different hyperparameters
5. **Compare**: Run multiple models and compare

---

## 🆘 Need Help?

Check these files for more details:
- [NOTEBOOK_EXPLANATIONS.md](NOTEBOOK_EXPLANATIONS.md) - Detailed explanations
- [QUICK_GUIDE_EDA.md](QUICK_GUIDE_EDA.md) - EDA guide
- [QUICK_GUIDE_FINETUNING.md](QUICK_GUIDE_FINETUNING.md) - Fine-tuning guide
- [QUICK_GUIDE_EVALUATION.md](QUICK_GUIDE_EVALUATION.md) - Evaluation guide

---

**Ready to start? Begin with notebook 1.2! 🚀**

# Quick Reference: Fine-Tuning Notebooks (7.1, 9.1, 10.1)

## 🎯 What do these notebooks do?
These notebooks **fine-tune GPT-3.5-turbo** on your custom BBC news classification data.

## 🔑 Key Concepts

### What is Fine-Tuning?
Fine-tuning = Teaching a pre-trained model to specialize in YOUR task
- Base Model: GPT-3.5-turbo (trained on general knowledge)
- Your Data: BBC news articles + categories
- Result: Custom model that classifies news better

## 📝 Step-by-Step Process

### 1️⃣ Initialize OpenAI Client
```python
from openai import OpenAI
client = OpenAI()
```
✅ Connects to OpenAI API using your API key

### 2️⃣ Upload Training Data
```python
client.files.create(
    file=open("test.jsonl", "rb"),
    purpose="fine-tune"
)
```
- Uploads prepared training data
- Returns: **File ID** (e.g., `file-abc123xyz`)
- Format: JSONL (one JSON object per line)

### 3️⃣ Create Fine-Tuning Job
```python
client.fine_tuning.jobs.create(
    model="gpt-3.5-turbo",
    training_file="file-lIyi8uMGNxXxxIGWxUD2S02B",
    hyperparameters={"n_epochs": 1},
    validation_file="file-aDNmk2N5cCCiDvpTuIe0el3g"
)
```

**Parameters explained**:
- `model`: Which OpenAI model to fine-tune
- `training_file`: Your prepared training data
- `n_epochs`: How many times to train (1-3 recommended)
- `validation_file`: Data to check overfitting

### 4️⃣ Monitor Progress
```python
client.fine_tuning.jobs.list(limit=10)                    # See all jobs
client.fine_tuning.jobs.retrieve("ftjob-xyz")             # Check specific job
client.fine_tuning.jobs.list_events("ftjob-xyz", limit=5) # See events/logs
```

### 5️⃣ Use Your Fine-Tuned Model
```python
completion = client.chat.completions.create(
    model="ft:gpt-3.5-turbo-0125:bis::9tt19es9",  # Your custom model
    messages=[{
        "role": "user",
        "content": "Classify this: A new phone is launched"
    }]
)
print(completion.choices[0].message)
```

## 📊 What Happens During Fine-Tuning?

```
Your Data (JSONL format)
    ↓
OpenAI's Fine-Tuning Service
    ↓
Training: Model learns from your examples
    ↓
Validation: Checks it doesn't memorize
    ↓
New Model Created (e.g., ft:gpt-3.5-turbo-0125:bis::xyz)
```

## ⏱️ Time & Cost

| Aspect | Details |
|--------|---------|
| Training Time | 5-30 minutes |
| Cost | $0.03 per 1K tokens |
| GPU Usage | Handled by OpenAI |

## 🔍 How to Check Training Status

**Possible statuses**:
- `queued`: Waiting to start
- `validating_files`: Checking data format
- `running`: Currently training
- `succeeded`: ✅ Done! Model ready
- `failed`: ❌ Error occurred

## ✅ What Success Looks Like

```
Fine-tuning job started!
ID: ftjob-ku7ytdtAEdx5jxthRANQTqr0

Status: succeeded
Trained tokens: 450,000
Validation loss: 0.45

Your model: ft:gpt-3.5-turbo-0125:bis::9tt19es9
```

## 📋 JSONL File Format

Your training data must look like this:

```json
{"messages": [{"role": "user", "content": "Classify: Article text here"}, {"role": "assistant", "content": "business"}]}
{"messages": [{"role": "user", "content": "Another article..."}, {"role": "assistant", "content": "tech"}]}
{"messages": [{"role": "user", "content": "Sports news..."}, {"role": "assistant", "content": "sport"}]}
```

Each line = one training example

## 🚨 Common Issues

| Problem | Solution |
|---------|----------|
| `API key not found` | Set `OPENAI_API_KEY` environment variable |
| `File format error` | Ensure JSONL format is correct |
| `Insufficient data` | Need at least 100 examples |
| `Training fails` | Check file IDs are correct |
| `Validation loss too high` | Data quality issue or wrong format |

## 💡 Tips for Best Results

✅ **DO**:
- Use 100+ examples minimum
- Balance categories (similar counts)
- Use clear, consistent formatting
- Run multiple iterations with different hyperparameters
- Save your model IDs

❌ **DON'T**:
- Use corrupted or malformed data
- Forget to validate data format
- Train on less than 20 examples
- Use extremely high n_epochs (causes overfitting)

## 🔄 Comparing Models

Run notebooks 7.1, 9.1, and 10.1 with different settings:
- Different `n_epochs` (1, 2, 3)
- Different training data subsets
- Different hyperparameters

Then evaluate each with the evaluation notebooks to find the best!

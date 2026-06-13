# Quick Reference: Evaluation Notebooks (11.1, 12.1)

## 🎯 What do these notebooks do?
These notebooks **test and measure** how well your fine-tuned model works on new data.

## 📊 The Evaluation Process

```
Fine-Tuned Model (from notebooks 7.1/9.1/10.1)
    ↓
Test Data (articles + real categories)
    ↓
Run predictions
    ↓
Compare with real answers
    ↓
Calculate Accuracy %
```

## 📝 Step-by-Step Explanation

### Step 1️⃣: Import Libraries
```python
from openai import OpenAI
import pandas as pd

client = OpenAI()
```

### Step 2️⃣: Load Test Data
```python
df = pd.read_csv('dataset/test.csv', encoding='unicode_escape')
labels = df.iloc[:,1].tolist()  # Ground truth (correct answers)
texts = df.iloc[:,0].tolist()    # Test articles
```

Example:
```
Text: "Apple releases new iPhone with better camera"
Label: "tech" ← This is the correct answer
```

### Step 3️⃣: Select Test Subset
```python
texts = texts[450:]   # Use articles from index 450 onward
labels = labels[450:]
```
- Why? To avoid testing on data the model has already seen during training

### Step 4️⃣: Define Inference Function
```python
def inference_for_eval(text, m):
    completion = client.chat.completions.create(
        model=m,  # Your fine-tuned model
        messages=[
            {
                "role": "system",
                "content": "You are a classifier for: business, entertainment, politics, sport, tech."
            },
            {
                "role": "user",
                "content": text
            }
        ]
    )
    return completion.choices[0].message.content
```

This function:
- Takes an article and model ID
- Sends it to the model
- Gets back the predicted category

### Step 5️⃣: Generate Predictions
```python
output = [inference_for_eval(text, "ft:gpt-3.5-turbo-0125:bis::9tt19es9") 
          for text in texts]
```

Result:
```python
output = ["business", "tech", "sport", "entertainment", "politics", ...]
```

### Step 6️⃣: Calculate Accuracy
```python
# Count correct predictions
correct_classifications = sum(
    classification == label 
    for classification, label in zip(output, labels)
)

# Calculate percentage
total_classifications = len(labels)
accuracy_percentage = (correct_classifications / total_classifications) * 100

print(f"Accuracy: {accuracy_percentage:.2f}%")
```

## 📈 Understanding Results

### Accuracy Interpretation

| Accuracy | Meaning |
|----------|---------|
| **90%+** | 🟢 Excellent! Model works very well |
| **80-90%** | 🟡 Good, but could improve |
| **70-80%** | 🟡 Acceptable, needs improvement |
| **<70%** | 🔴 Poor, retrain with better data |

### Example Output

```
Sample predictions: ['business', 'tech', 'sport', ...]
Total classifications: 250
Accuracy: 87.60%
```

This means: **87.60% of predictions were correct**

## 🔄 Comparing Models

### Evaluate Multiple Models

```python
# Compare fine-tuned model vs base model vs another version
base_model = "gpt-3.5-turbo"
tuned_model_1 = "ft:gpt-3.5-turbo-0125:bis::9tt19es9"
tuned_model_2 = "ft:gpt-3.5-turbo-0125:bis::abc12345"

for model in [base_model, tuned_model_1, tuned_model_2]:
    output = [inference_for_eval(text, model) for text in texts]
    accuracy = calculate_accuracy(output, labels)
    print(f"{model}: {accuracy:.2f}%")
```

## 🎯 Confusion Matrix

To see which categories are confused:

```python
from sklearn.metrics import confusion_matrix
import numpy as np

cm = confusion_matrix(labels, output, labels=["business", "entertainment", "politics", "sport", "tech"])

# Shows:
# - Rows: Real categories
# - Columns: Predicted categories
# - What the model mistakes for what
```

## ⏱️ Time & Cost

| Aspect | Details |
|--------|---------|
| Evaluation Time | 1-5 minutes |
| Cost | ~$0.01 per 100 API calls |
| API Calls | 1 per test article |

## 💡 What to Check

When evaluating your model:

✅ **High accuracy**: Model is working well
✅ **Balanced accuracy**: Good across all categories
✅ **Stable results**: Same accuracy in notebooks 11.1 and 12.1
❌ **Overfitting**: Great on training data, poor on test data
❌ **Bias**: One category always predicted correctly, others wrong

## 🚨 Common Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| Very low accuracy | Bad training data | Review and clean data |
| One category wrong | Imbalanced training | Add more examples |
| Model timeout | API issue | Retry or split into smaller batches |
| Wrong model ID | Typo in model name | Copy-paste from successful job |

## 📊 Better Evaluation Metrics

Beyond accuracy, consider:

### Precision & Recall
```python
from sklearn.metrics import precision_recall_fscore_support

precision, recall, f1, _ = precision_recall_fscore_support(
    labels, output, average='weighted'
)
print(f"Precision: {precision:.2f}")
print(f"Recall: {recall:.2f}")
print(f"F1-Score: {f1:.2f}")
```

- **Precision**: Of predictions, how many were correct?
- **Recall**: Of real instances, how many did we find?
- **F1-Score**: Balance between precision and recall

## 🔄 Iterative Improvement

1. Run notebook 7.1 (fine-tune)
2. Run notebook 11.1 (evaluate)
3. Check accuracy
4. If low, run 9.1 with better data/hyperparameters
5. Evaluate again with 12.1
6. Compare results

Repeat until satisfied!

## 🎓 Example Analysis

```
Results Summary:
================
Model 1 (7.1):  85.2% accuracy
Model 2 (9.1):  88.7% accuracy ← Best!
Model 3 (10.1): 84.5% accuracy

Decision: Use Model 2 for production
```

## ✅ Success Indicators

Your evaluation is successful when:
- ✅ No API errors
- ✅ Accuracy printed without errors
- ✅ Accuracy > 80%
- ✅ Can compare multiple models
- ✅ Results are reproducible

---

**Next Steps**: 
1. If happy with accuracy → Use in production ✅
2. If not satisfied → Retrain with better data 🔄
3. Save your best model ID for future use 💾

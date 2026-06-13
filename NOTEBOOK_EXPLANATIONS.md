# OpenAI GPT Fine-Tuning Project - Notebook Explanations

This guide provides detailed explanations for each Jupyter notebook in this project. The project focuses on fine-tuning OpenAI's GPT models for text classification on BBC news articles.

---

## 📊 Project Overview

**Objective**: Fine-tune OpenAI's GPT-3.5-turbo model to classify news articles into 5 categories:
- **Business**
- **Entertainment** 
- **Politics**
- **Sport**
- **Tech**

**Dataset**: BBC news articles (bbc.csv)

**Workflow**: Data Exploration → Data Preparation → Fine-tuning → Evaluation

---

## 🔍 EDA (Exploratory Data Analysis) Notebooks

### **1. `4. Fine tune OpenAI GPT model __ 1.2 eda.ipynb`**

**Purpose**: Initial exploration and cleaning of the BBC dataset

**What it does**:
1. **Install Dependencies**: Sets up required libraries
   - `tiktoken`: Tokenizer for OpenAI models
   - `matplotlib`: For visualization
   - `wordcloud`: To create word clouds
   - `nltk`: Natural Language Toolkit
   - `pandas`: Data manipulation

2. **Load & Inspect Data**:
   - Reads the `bbc.csv` file into a pandas DataFrame
   - Displays dataset info and basic statistics
   - Shows the distribution of article categories

3. **Data Cleaning**:
   - Removes duplicate articles
   - Handles missing values (NaN)
   - Prints the number of articles before and after cleaning

4. **Basic Analysis**:
   - Counts articles per category
   - Checks for unique articles
   - Validates data quality

**Key Output**: Cleaned dataset ready for further processing

**Skills Learned**: Data loading, inspection, cleaning, and preprocessing

---

### **2. `4. Fine tune OpenAI GPT model __ 2.2 eda.ipynb`**

**Purpose**: Similar to 1.2 - Exploratory Data Analysis and cleaning

**What it does**:
- Identical to notebook 1.2
- Serves as a second iteration or checkpoint for data exploration
- Ensures dataset quality before fine-tuning

**When to use**: Run this if you want to verify data consistency across iterations

---

### **3. `4. Fine tune OpenAI GPT model __ 3.2 eda.ipynb`**

**Purpose**: Third iteration of EDA and data preparation

**What it does**:
- Repeats the EDA process from notebooks 1.2 and 2.2
- Provides another checkpoint for data validation
- Cleans and prepares the dataset

**Purpose of repetition**: Different iterations may work with different data splits or subsets

---

### **4. `4. Fine tune OpenAI GPT model __ 4.1 eda.ipynb`**

**Purpose**: Fourth iteration of EDA with data analysis

**What it does**:
- Completes the EDA pipeline
- Final data validation before feature engineering
- Ensures all data quality checks are passed

**Key Insight**: Running multiple iterations ensures robustness and catches edge cases

---

### **5. `4. Fine tune OpenAI GPT model __ 6.1 eda.ipynb`**

**Purpose**: Final comprehensive EDA before fine-tuning

**What it does**:
- Complete data exploration and cleaning
- Final validation of the dataset
- Preparation for model fine-tuning

**When to use**: Use this as the final checkpoint before moving to fine-tuning

---

## 🚀 Fine-Tuning Notebooks

### **6. `4. Fine tune OpenAI GPT model __ 7.1 fine-tune.ipynb`**

**Purpose**: Fine-tune OpenAI GPT-3.5-turbo model using prepared training data

**Step-by-step explanation**:

1. **Initialize OpenAI Client**:
   ```python
   from openai import OpenAI
   client = OpenAI()
   ```
   - Creates connection to OpenAI API using your API key

2. **Upload Training Data**:
   ```python
   client.files.create(file=open("test.jsonl","rb"), purpose="fine-tune")
   ```
   - Uploads the training data file (test.jsonl) to OpenAI servers
   - Returns a file ID for reference

3. **Create Fine-Tuning Job**:
   ```python
   client.fine_tuning.jobs.create(
       model="gpt-3.5-turbo",
       training_file="file-lIyi8uMGNxXxxIGWxUD2S02B",
       hyperparameters={"n_epochs": 1},
       validation_file="file-aDNmk2N5cCCiDvpTuIe0el3g"
   )
   ```
   - Initiates the fine-tuning process
   - `n_epochs=1`: Trains on the data 1 time
   - Uses validation file to prevent overfitting

4. **Monitor Fine-Tuning Progress**:
   - List all fine-tuning jobs
   - Retrieve specific job status
   - View job events and logs

5. **Test Fine-Tuned Model**:
   ```python
   completion = client.chat.completions.create(
       model="ft:gpt-3.5-turbo-0125:bis::9tt19es9",
       messages=[{"role": "user", "content": "Classify: A new mobile phone is launched"}]
   )
   ```
   - Sends a test request to your fine-tuned model
   - Gets classification result

**Key Metrics**: Training time, cost, and model performance

**Output**: Fine-tuned model ID (e.g., `ft:gpt-3.5-turbo-0125:bis::9tt19es9`)

---

### **7. `4. Fine tune OpenAI GPT model __ 9.1 fine-tune.ipynb`**

**Purpose**: Repeat fine-tuning with different hyperparameters or data

**What it does**:
- Similar to 7.1 but may use different:
  - Training data
  - Hyperparameters
  - Validation data
  - Model versions

**Use Case**: Experiment with different training strategies to find optimal model

---

### **8. `4. Fine tune OpenAI GPT model __ 10.1 fine-tune.ipynb`**

**Purpose**: Third fine-tuning iteration

**What it does**:
- Another round of fine-tuning
- Tests different configurations
- Compares results with previous iterations

**Why multiple iterations?**: Find the best hyperparameters and data configuration for maximum accuracy

---

## 📈 Evaluation Notebooks

### **9. `4. Fine tune OpenAI GPT model __ 11.1 evaluation.ipynb`**

**Purpose**: Evaluate the performance of the fine-tuned model

**Step-by-step explanation**:

1. **Import Libraries**:
   - OpenAI client for API access
   - Pandas for data handling

2. **Load Test Data**:
   ```python
   df = pd.read_csv('dataset/test.csv', encoding='unicode_escape')
   labels = df.iloc[:,1].tolist()  # Ground truth labels
   texts = df.iloc[:,0].tolist()    # Test articles
   ```
   - Loads test articles and their correct labels
   - Splits data: takes articles from index 450 onwards

3. **Define Inference Function**:
   ```python
   def inference_for_eval(text, m):
       completion = client.chat.completions.create(
           model=m,
           messages=[
               {"role": "system", "content": "You are a classifier..."},
               {"role": "user", "content": text}
           ]
       )
       return completion.choices[0].message.content
   ```
   - Sends test articles to the model
   - Gets classification predictions

4. **Generate Predictions**:
   - Runs all test articles through the fine-tuned model
   - Collects predictions for comparison

5. **Calculate Accuracy**:
   ```python
   accuracy_percentage = (correct_classifications / total_classifications) * 100
   ```
   - Compares predictions with ground truth
   - Calculates percentage accuracy

**Key Metrics**:
- **Accuracy**: Percentage of correctly classified articles
- **Total Classifications**: Number of test samples

**Output Example**: "Accuracy: 92.45%"

---

### **10. `4. Fine tune OpenAI GPT model __ 12.1 evaluation.ipynb`**

**Purpose**: Second evaluation run on the fine-tuned model

**What it does**:
- Similar to 11.1
- May evaluate different model versions
- Tests with different test data subsets
- Compares results across models

**Use Case**: Verify model consistency and compare different fine-tuned versions

---

## 📋 Quick Reference: Notebook Types

| Type | Notebooks | Purpose |
|------|-----------|---------|
| **EDA** | 1.2, 2.2, 3.2, 4.1, 6.1 | Data exploration and cleaning |
| **Fine-Tuning** | 7.1, 9.1, 10.1 | Train models with OpenAI |
| **Evaluation** | 11.1, 12.1 | Test and measure model accuracy |

---

## 🔄 Recommended Workflow

```
1. Run EDA Notebooks (1.2 → 6.1)
   ↓
2. Prepare training & validation data
   ↓
3. Run Fine-Tuning Notebook (7.1)
   ↓
4. Wait for training to complete
   ↓
5. Run Evaluation Notebook (11.1)
   ↓
6. Analyze Results
   ↓
7. Iterate: Run 9.1/10.1 with different settings if needed
```

---

## 🛠️ Key Concepts Explained

### **JSONL Format**
- **JSON Lines**: Each line is a separate JSON object
- Used for fine-tuning data with messages and completions

### **Fine-Tuning Parameters**

| Parameter | Meaning | Example |
|-----------|---------|---------|
| `n_epochs` | How many times to train on the data | 1-3 |
| `training_file` | File ID of training data | file-xyz123 |
| `validation_file` | File used to prevent overfitting | file-abc456 |

### **Hyperparameters**
- `n_epochs`: More epochs = more training (but risk of overfitting)
- `batch_size`: How many samples train at once
- `learning_rate`: How quickly the model learns

---

## 📊 Expected Outputs

### **EDA Notebooks**
- Dataset shape and info
- Category distribution
- Data quality report
- Missing values count

### **Fine-Tuning Notebooks**
- File upload confirmation
- Job ID
- Training progress
- Final model ID

### **Evaluation Notebooks**
- Sample predictions
- Accuracy percentage
- Performance metrics

---

## 💡 Common Issues & Solutions

### **Issue: API Key Not Found**
- Make sure your OpenAI API key is set as environment variable: `OPENAI_API_KEY`

### **Issue: File Not Found**
- Ensure `bbc.csv` and test data files are in the correct directory
- Check file paths in the notebook

### **Issue: Training Takes Too Long**
- This is normal! Fine-tuning can take 5-30 minutes depending on data size
- You can check status with `client.fine_tuning.jobs.retrieve(job_id)`

### **Issue: Low Accuracy**
- Try different hyperparameters
- Use more training data
- Run additional epochs
- Review data quality

---

## 🎯 Learning Path

**Beginner**:
1. Run one EDA notebook (1.2)
2. Understand the data structure
3. Run one fine-tuning notebook (7.1)

**Intermediate**:
1. Run all EDA notebooks
2. Run multiple fine-tuning iterations
3. Run evaluation notebooks
4. Compare results

**Advanced**:
1. Modify hyperparameters
2. Experiment with different data splits
3. Analyze failure cases
4. Implement custom evaluation metrics

---

## 📚 Additional Resources

- [OpenAI Fine-Tuning Docs](https://platform.openai.com/docs/guides/fine-tuning)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
- [Pandas Documentation](https://pandas.pydata.org/)
- [NLTK Documentation](https://www.nltk.org/)

---

## 🎓 What You'll Learn

✅ How to prepare data for fine-tuning  
✅ How to use OpenAI API  
✅ Fine-tuning best practices  
✅ Model evaluation techniques  
✅ Data exploration and cleaning  
✅ Text classification fundamentals  

---

**Happy Learning! 🚀**

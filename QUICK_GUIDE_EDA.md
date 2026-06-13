# Quick Reference: EDA Notebooks (1.2, 2.2, 3.2, 4.1, 6.1)

## 🎯 What are these notebooks?
These notebooks perform **Exploratory Data Analysis (EDA)** on the BBC news dataset. They prepare your data for fine-tuning.

## 📝 What happens in these notebooks?

### Step 1: Install Libraries
```python
pip install tiktoken matplotlib wordcloud nltk pandas
```
- **tiktoken**: Counts tokens (how OpenAI measures input)
- **matplotlib**: Creates charts
- **wordcloud**: Shows most common words visually
- **nltk**: Natural language processing tools
- **pandas**: Data manipulation

### Step 2: Load Data
```python
df = pd.read_csv("bbc.csv")
```
Loads the BBC news articles with categories

### Step 3: Explore Data
- **df.info()**: Shows data types and structure
- **df.describe()**: Statistics (mean, min, max, etc.)
- **df['category'].value_counts()**: How many articles in each category

### Step 4: Clean Data
```python
df.drop_duplicates(inplace=True)  # Remove exact duplicates
df.dropna(inplace=True)            # Remove missing values
```

## 🔢 Expected Output
```
Category counts:
business        510
entertainment   386
politics        417
sport           511
tech            401
```

## 💡 Why run multiple times?
- **Checkpoint**: Verify data consistency
- **Verification**: Ensure cleaning worked correctly
- **Documentation**: Record different versions

## ⏱️ Time to run
⏱️ **2-5 minutes**

## ✅ What you should see
- ✓ No errors loading the CSV
- ✓ Dataset info shows 2-3 columns (text, category, etc.)
- ✓ Clean count of articles per category
- ✓ Zero missing values after cleaning

## 🚨 Common Issues

| Problem | Solution |
|---------|----------|
| `File not found` | Ensure `bbc.csv` is in the same folder |
| `Import error` | Run first cell to install packages |
| `High duplicates` | Data quality - check the source file |

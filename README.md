# Python-Based Code Plagiarism Checker

🔍 What Makes This a Code Plagiarism Checker?

Token-based comparison: 
+ Focuses on code structure, not just text.
+ Handles comments and formatting: Ignores superficial differences.
+ Threshold-based alerts: Flags suspiciously similar files.

This version:
+ Compares all files in two folders.
+ Uses token-based TF-IDF + cosine similarity.
+ Handles empty files and logs high similarity cases.

```python
import os
import re
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

def list_code_files(folder):
    return [os.path.join(folder, f) for f in os.listdir(folder) if f.endswith('.py')]

def read_file(file_path):
    with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
        return f.read()

def preprocess_code(code):
    # Remove comments and normalize whitespace
    code = re.sub(r'#.*', '', code)
    code = re.sub(r'""".*?"""', '', code, flags=re.DOTALL)
    code = re.sub(r'\s+', ' ', code)
    return code.strip()

def calculate_similarity(code1, code2):
    if not code1 or not code2:
        return 0.0
    vectorizer = TfidfVectorizer(token_pattern=r'\b\w+\b')
    try:
        tfidf_matrix = vectorizer.fit_transform([code1, code2])
        return cosine_similarity(tfidf_matrix[0:1], tfidf_matrix[1:2])[0][0]
    except ValueError:
        return 0.0

def check_code_plagiarism(source_folder, sink_folder, threshold=0.8, output_csv='c:/Temp/code_similarity_report.csv'):
    source_files = list_code_files(source_folder)
    sink_files = list_code_files(sink_folder)

    with open(output_csv, 'w', encoding='utf-8') as report:
        report.write("Source File,Sink File,Similarity (%)\n")
        for src in source_files:
            src_code = preprocess_code(read_file(src))
            for sink in sink_files:
                sink_code = preprocess_code(read_file(sink))
                sim = calculate_similarity(src_code, sink_code)
                sim_percent = f"{sim:.2%}"
                print(f"{src} <-> {sink}: {sim_percent}")
                report.write(f"{src},{sink},{sim_percent}\n")
                if sim >= threshold:
                    print(f"⚠️ Possible plagiarism detected: {src} and {sink} ({sim_percent})")

# Example usage:
source_folder = 'C:/Temp/source/'
sink_folder = 'C:/Temp/sink/'
check_code_plagiarism(source_folder, sink_folder)

```

Result:
<img width="1099" height="39" alt="image" src="https://github.com/user-attachments/assets/512a7bcf-a1d5-45be-b506-d53e5487cc3f" />


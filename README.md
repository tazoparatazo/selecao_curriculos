**Olá!  Vou mostrar como desenvolvi um código para ler e processar arquivos PDF, pensando em facilitarp trabalho de recrutadores com a seleção de curriculos.**

Primeiro, importamos as bibliotecas necessárias. Usamos os e re para lidar com arquivos e expressões regulares. pandas é usado para estruturar e manipular os dados, PyPDF2 para ler arquivos PDF e google.colab para carregar arquivos no ambiente do Google Colab.
```
python
Copy code
import os
import re
import pandas as pd
from PyPDF2 import PdfReader
from google.colab import files
```

Depois, defini a função extract_info_from_pdf(pdf_path). Esta função abre um arquivo PDF e extrai todo o texto, página por página.


```
def extract_info_from_pdf(pdf_path):
    with open(pdf_path, "rb") as file:
        pdf = PdfReader(file)
        text = ""
        for page in range(len(pdf.pages)):
            text += pdf.pages[page].extract_text()
        return text
```
Em seguida, criei a função find_contact_info(text). Esta função procura no texto por padrões que se parecem com um e-mail ou um telefone. Atenção: a extração de nomes não está implementada neste exemplo.

```
def find_contact_info(text):
    email = re.findall(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,7}\b', text)
    phone = re.findall(r'\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}', text)
    name = "" 
    return name, email[0] if email else "", phone[0] if phone else ""
```

A função count_keywords(text, keywords) é onde contamos o número de vezes que cada palavra-chave aparece no texto. Converti todo o texto e as palavras-chave em minúsculas para garantir uma comparação correta, independentemente da capitalização.

```
def count_keywords(text, keywords):
    text = text.lower()
    return sum(keyword.lower() in text for keyword in keywords)
```
Agora, chegamos à função process_pdfs_in_folder(folder_path, keywords). Esta função processa todos os arquivos PDF em uma pasta específica. Ela extrai as informações de contato e conta as palavras-chave, armazenando tudo em um DataFrame do pandas.

```
def process_pdfs_in_folder(folder_path, keywords):
    data = {'Name': [], 'Email': [], 'Phone': [], 'Keyword Count': []}
    for filename in os.listdir(folder_path):
        if filename.endswith('.pdf'):
            text = extract_info_from_pdf(os.path.join(folder_path, filename))
            name, email, phone = find_contact_info(text)
            keyword_count = count_keywords(text, keywords)
            data['Name'].append(name)
            data['Email'].append(email)
            data['Phone'].append(phone)
            data['Keyword Count'].append(keyword_count)
    df = pd.DataFrame(data)
    return df.sort_values(by='Keyword Count', ascending=False)
```
Finalmente, nós carregamos os arquivos PDF, definimos as palavras-chave que queremos procurar e processamos os PDFs para criar o DataFrame.

```
uploaded = files.upload()
keywords = ['keyword1', 'keyword2', 'keyword3'] 
df = process_pdfs_in_folder('.', keywords) 
print(df)
```
E é isso! Agora temos um DataFrame pandas com o nome, e-mail, telefone e a contagem de palavras-chave de cada arquivo PDF.

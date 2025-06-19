# python Parser的一些使用方法

这里主要记录一下[这篇调查](rag-parser-diao-cha-jie-guo.md)里parser的简单用法：

## [PyMuPDF](https://zhuanlan.zhihu.com/p/517737462)

安装：

```bash
pip install PyMuPDF
```

使用：

```python
>>> import fitz
>>> print(fitz.__doc__)
PyMuPDF 1.25.3: Python bindings for the MuPDF 1.25.4 library (rebased implementation).
Python 3.10 running on linux (64-bit).
>>> doc = fitz.open('/home/labuser/junyu/GenAIExamples/EdgeCraftRAG/Phoenix TCB HVM English Operation Manual.pdf')
>>> toc = doc.get_toc()
>>> page = doc[441]
>>> page.get_text()
'PHOENIX TCB Operation Manual \nCh 7 – Menu Description \n \n7-162 \nButton Name \nFunction / Description  \nMotion System \nNuMotion Software \nVersion \nDisplay the NuMotion software version. \nFCU Software \nVersion \nDisplay the FCU software version. \nOther Components \nHeater Controller \nDisplay the heater controller software version. \nTP Dispense \nController \nDisplay the TP dispense controller software version. \nJet Dispense \nController \nDisplay the jet dispense controller software version. \nOxygen Analyzer \nDisplay the oxygen analyzer controller software version. \n \n'
```

## [pdfminer.six](https://products.documentprocessing.com/zh/parser/python/pdfminer.six/)

安装：

```bash
pip install pdfminer.six
```

使用：

```python
# Import extract_text function from the pdfminer.six library
from pdfminer.high_level import extract_text

# Specify the PDF file you want to extract text from
pdf_file = 'documentprocessing.pdf'

# Extract text from the PDF
text = extract_text(pdf_file)

# Removing any empty lines in the document
# Split the text into lines and filter out empty lines
lines = [line.strip() for line in text.splitlines() if line.strip()]

# Join the non-empty lines back together with newline characters
cleaned_text = '\n'.join(lines)

# Print the cleaned text
print(cleaned_text)
```

## [pdfplumber](https://www.cnblogs.com/mayi0312/p/16638325.html)

安装：

```bash
pip install pdfplumber
```

使用：

```python
>>> import pdfplumber
>>> pdf_name = '/home/labuser/junyu/GenAIExamples/EdgeCraftRAG/Phoenix TCB HVM English Operation Manual.pdf'
>>> pdf = pdfplumber.open(pdf_name)
>>> page_441 = pdf.pages[441]
>>> page_441.extract_text()
'PHOENIX TCB Operation Manual\nCh 7 – Menu Description\nButton Name Function / Description\nMotion System\nNuMotion Software Display the NuMotion software version.\nVersion\nFCU Software Display the FCU software version.\nVersion\nOther Components\nHeater Controller Display the heater controller software version.\nTP Dispense Display the TP dispense controller software version.\nController\nJet Dispense Display the jet dispense controller software version.\nController\nOxygen Analyzer Display the oxygen analyzer controller software version.\n7-162'
```

## [tabula](https://www.cnblogs.com/taosiyu/p/14116099.html)

[安装](https://blog.csdn.net/weixin_40566713/article/details/140291561)：

```bash
pip install jpype1

sudo apt update
sudo apt install openjdk-11-jre

pip install tabula-py
```

使用：

```python
import tabula

path = 'test.pdf'

df = tabula.read_pdf(path, encoding='gbk', pages='all')
for indexs in df.index:
    print(df.loc[indexs].values)
```

## [PyPDF2](https://blog.csdn.net/PolarisRisingWar/article/details/125030542)

安装：

```bash
pip install PyPDF2
```

使用：

```python
from PyPDF2 import PdfReader

reader = PdfReader(pdf_path)
number_of_pages = len(reader.pages)

print(number_of_pages)

page = reader.pages[0]
text = page.extract_text()
```

## [pypdfium2](https://pypi.org/project/pypdfium2/)

安装：

```bash
pip install pypdfium2
```

使用：

```python
import pypdfium2 as pdfium

pdf = pdfium.PdfDocument("./path/to/document.pdf")
version = pdf.get_version()  # get the PDF standard version
n_pages = len(pdf)  # get the number of pages in the document
page = pdf[0]  # load a page

# Load a text page helper
textpage = page.get_textpage()

# Extract text from the whole page
text_all = textpage.get_text_range()
# Extract text from a specific rectangular area
text_part = textpage.get_text_bounded(left=50, bottom=100, right=width-50, top=height-100)

# Locate text on the page
searcher = textpage.search("something", match_case=False, match_whole_word=False)
# This returns the next occurrence as (char_index, char_count), or None if not found
first_occurrence = searcher.get_next()
```

## [camelot](https://cloud.tencent.com/developer/article/1489031)

安装：

```bash
pip install camelot-py
```

使用：

```python
import camelot
 
# 从PDF文件中提取表格
tables = camelot.read_pdf('E://eg.pdf', pages='1', flavor='stream')
 
# 表格信息
print(tables)
print(tables[0])
# 表格数据
print(tables[0].data)
```

##

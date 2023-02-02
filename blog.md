# Working with Oral Testimonies as Data

In this blog post, we will explore some of the challenges of working with oral testimonies as data. We will also cover some solutions and methodological steps you can make in working with this type of data. For our case study, we will use a collection of over 1,000 English oral testimonies from the United States Holocaust Memorial Museum.

# Background

For those who do not know, I work in the field of multilingual natural language processing, specifically applying NLP solutions to historical documents and museum archives. I am attached to the Smithsonian Institution's Data Science Lab as the digital humanities postdoctoral fellow. My appoint is split, however, between the American Women's History Initiative at the Smithsonian and the USHMM. At the latter, I have worked with the oral testimonies for nearly two years. In addition this, I have also worked with oral testimonies from South Africa's Truth and Reconciliation Commission (TRC) for the Bitter Aloe Project.

Throughout this blog post, I will be drawing primarily from my personal experience in working with oral testimonies from both projects. I will be detailing some of the methodological challenges that transcend a single collection of oral testimonies. It is my hope that by approaching the problem in this way, the information I provide will be specifically related to USHMM oral testimonies, but broadly applicable to oral testimonies in other collections.

# Challenges of USHMM Oral Testimonies

The USHMM oral testimony collection comprises nearly 2,000 transcribed testimonies of Holocaust survivors. Of these 2,000 testimonies, approximately 1,300 are in English. In this blog post, we will be working with a subset of approximately 1,100 that follow a similar schema, or way in which they are structured.

The testimonies themselves present certain challenges within the field of NLP. First, the individuals who gave oral testimony in English were not native speakers. In some cases, this results in the speaker having to express certain concepts in their first language.

This also leads to certain issues in the transcription. The transcriber of the original audio files was not necessarily present at the time of the oral interview and, in some instances, spells words phonetically. These are frequently concepts, places, or things in eastern Europe. In other cases, the speaker will have a thick accent which prevents the transcriber from accurately transcribing the audio. This has resulted in noted gaps in certain testimonies.

The testimonies are available as PDFs from the museum, but these PDFs present certain challenges. First, they are encrypted which means the current OCR is difficult to extract programmatically. Second, the OCR has mistakes consistent with OCR of the early 2000s. Third, the raw text of the OCR is not structured which results in headers and footers frequently appearing alongside the main text. It also means that the data within a testimony is not tagged, so we cannot identify individual speakers or separate questions from answers. In other words, the PDFs in their original state is simply raw, unstructured text with frequent OCR errors.

Each of these challenges make working with the oral testimonies en masse difficult. The issues I have noted above are not unique to the USHMM oral testimonies. These are identical issues that surface when working with oral testimonies from other collections, such as the TRC in South Africa. There are other collections of Holocaust oral testimonies, such as those available from the USC Shoah Foundation. While these testimonies are structured with cleaned XML markup, the data is not open-source and requires permission to obtain.

# Download the Original PDFs from USHMM

In order to begin working with the USHMM oral testimonies, it was vital to go back to the original PDFs and re-OCR them. In order to do that, however, we first must download the files. We can do this in Python with `requests` which allows users to call up a server and download request.

Attached to this blog is a collection of notebooks that can be used to recreate the workflow laid out in this blog, including the downloading of the original PDFs. The urls for each English oral testimony can be found in the `data` subfolder of this repository.

The CSV looks like the table below, where the page is the url for a given testimony. On that page sits a PDF transcript which can be downloaded.

|    | identifiers                                                             | date                        | page                                                   |
|---:|:------------------------------------------------------------------------|:----------------------------|:-------------------------------------------------------|
|  0 | Oral History , Accession Number: 2001.28 , RG Number: RG-50.030.0410    | interview: 2001 February 20 | https://collections.ushmm.org/search/catalog/irn512935 |
|  1 | Oral History , Accession Number: 2015.382.1 , RG Number: RG-50.106.0245 | interview: 2014 November 30 | https://collections.ushmm.org/search/catalog/irn185375 |
|  2 | Oral History , Accession Number: 2001.213.1 , RG Number: RG-50.969.0001 | Unknown                     | https://collections.ushmm.org/search/catalog/irn558424 |
|  3 | Oral History , RG Number: RG-50.999.0419                                | interview: 2013 April 25    | https://collections.ushmm.org/search/catalog/irn598532 |
|  4 | Oral History , RG Number: RG-50.030.0274                                | interview: 1995 August 18   | https://collections.ushmm.org/search/catalog/irn504758 |

To download a PDF on a particular page, you can make a request to the server and download the PDF directly via the following Python function:

```python
def download_pdf(url, DIR):
    s =  requests.get(url)
    soup = BeautifulSoup(s.content)
    pdf = ""
    for a in soup.find_all("a"):
        if "pdf" in a["href"]:
            pdf = a["href"]
    if pdf != "":
        s = requests.get(pdf)
        filename = pdf.split("/")[-1]
        with open(f"{DIR}/{filename}", "wb") as f:
            f.write(s.content)
    else:
        print(f"No available pdf for {url}")
```


# OCR with Tesseract

Once all the oral testimonies have been downloaded, we can then re-OCR them via Python and Tesseract (an OCR engine from Google) via `pytesseract`. To convert the PDFs to a series of images, we can use the `convert_from_path` class in the `pdf2image` library.


```python
import glob
from pdf2image import convert_from_path
import pytesseract
```

We can then use `glob` to grab all the PDFs that contain the substring `trs_en.pdf`, which indicates that the PDF is an English transcript.

```python
files = glob.glob("pdfs/*trs_en.pdf")
len(files)
```

Once we have all the files, we can then iterate over each one and convert it 

```python
for filename in files:
    pages = convert_from_path(filename, 500)
    text = ""
    for page in pages:
        res = pytesseract.image_to_string(page, config="--psm 6")
        text = text+res+"\n"
    ocr_filename = filename.replace("pdfs", "tesseract_ocr").replace(".pdf", ".txt")
    with open (ocr_filename, "w", encoding="utf-8") as f:
        f.write(text)
```




# Post-Processing OCR


# Structuring Oral Testimonies


---
layout: post
title: Using QR Codes to Open Documents
subtitle: Improving compliance with QR Codes
cover-img: /assets/img/QR-Code-Workplace/qr_code_scan_cover.jpg
thumbnail-img: /assets/img/QR-Code-Workplace/qr_code_thumbnail_color.png
share-img: /assets/img/QR-Code-Workplace/qr_code_thumbnail_color.png
tags: [Selenium, BeautifulSoup, QR, pyzbar, opencv, Pharmaceutical]
readtime: True
---


# Problem Statement
In the Pharmaceutical Sector there are many different Standard Operating Procedures (SOP) and Forms for different activities in the production plant. Approved SOPs / Forms are usually parked in a central repository on a Content Management Platform (e.g. Veeva, Open Text).

In order to ensure that the latest version is always available at the production area, many companies incorporate a department to physically print a latest copy whenever a new version is available to serve as a reference. These documents are usually stored in a file that contains multiple SOPs together which brings about the issue of:
- Searching for the correct SOP
- Searching for the relevant step in the associated SOP
- Difficult to keep track of multiple SOP numbers or titles in order to reference them when required.
- Human Error tied to delayed updating of physical copies
- Human Error tied to incorrect physical version provided

So how can we solve some of these problems?

# Idea
To incorporate QR code tied to each equipment which contains the SOP number which can be scanned and retrieve the effective SOP version for reference on the tablet or laptop in the production area. This QR code can be tagged onto the associated equipment to quickly bring up the relevant document when required, eliminating the need for physical copies on the production floor and the effective version will always be displayed to the production team.

With the Covid-19 pandemic, I'm sure many Singaporeans are already familiar with scanning QR codes for SafeEntry/TraceTogether!

# The Code
## Packages Used / Installation
```python
pip install Pillow
pip install opencv-python
pip install pyzbar # For Windows
brew install zbar # For MacOs
pip install webdriver-manager
```
## Imports
```python
import cv2
from pyzbar import pyzbar
from bs4 import BeautifulSoup
import subprocess
```
## Reading/Decoding the QR Code
```python
def read_qr_code(image_frame):
    qr_codes = pyzbar.decode(image_frame)
    scanned_qr_text = ''
    for qr_code in qr_codes:
        qr_info = qr_codes.data.decode('utf-8')
        scanned_qr_text = qr_info
    return image_frame, scanned_qr_text
```

## Main Function
```python
def main():
    # Change value of VideoCapture to 1 if you intend to use another webcam / camera
    cam = cv2.VideoCapture(0)
    ret, image_frame = cam.read()
    while ret:
        ret, image_frame = cam.read()
        image_frame, qr_text = read_qr_code(image_frame)

        cv2.imshow('QR Code Scanner', image_frame)
        if len(qr_text)>0:
            # ---
            # Run code here to open document
            show_sop()
            # ---
            pass
        elif cv2.waitKey(1) & 0xFF == 27:
            break
    cam.release()
    cv2.destroyAllWindows()

if __name__ == '__main__':
    main()
```

## Opening the document with Selenium
```python
def show_sop():
    # Set proxies here if needed for a corporate environment
    os.environ["HTTP_PROXY"] = "address:port"
    os.environ["HTTPS_PROXY"] = "address:port"
    global driver # Set driver to global to prevent closure of Edge/Chrome

    # For Chrome
    driver = webdriver.Chrome(ChromeDriverManager(cache_valid_range=30).install())

    # For Microsoft Edge
    driver = webdriver.Edge(EdgeChromiumDriverManager(cache_valid_range=30).install())

    driver.get('insert address here')

    # ------
    # Input rest of code here to open the document with Selenium
    ...
    # ------

    # ------    
    # if there are additional versions e.g. In Training / Effective, use BeautifulSoup to check
    soup = BeautifulSoup(driver.page_source, features='html.parser')
    # ------

    # Kill the webdriver
    subprocess.call('../kill_driver.bat') # Code below
```

#### **`kill_driver.bat`**
```shell
@echo off
rem Kills all webdriver instances for Chrome and Edge

taskkill /im msedgedriver.exe /f
taskkill /im chromedriver.exe /f
```

# Final Thoughts
Although this mini-project may not have a significant in terms of savings (dollars wise), it hopes to help prevent unnecessary deviations or compliance issues which can take up valuable time and resources in a company and spur interest in digitalization of workflows / processes.

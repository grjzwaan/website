---
title:  "Flask Pdf Download"
date: 2019-04-24
description: "Getting generating PDF downloads in Flask to actually work. Not as easy as it sounds..."
tags: ['Python']
layout: post
---
This is a short note on getting PDF downloads working on Flask.

The goal is a button with 'Download as PDF':

* Generate an HTML page
* Convert his page to a PDF
* Download the PDF while staying on the same page


This seemed easy: just use pdfkit, get the correct dependencies and go. It turns out there were some hiccups for which I needed to combine several answers.

* wkhtml did not work headless, I needed to patch it
* PDFkit examples and the API ask for a filename, I did not want to store the PDF
* PDFkit returns bytes and not a fileobject that Flask expects
* For a few minutes I forgot about caching headers. I always want to generate a PDF on the current version


## Getting PDFkit to run on a Docker image
I used the following code to get it working. It uses `apt-get` to get wkhtmltopdf and its dependencies. Then, I patched wkhtmltopdf to make it work _headless_.

```docker
RUN apt-get update
RUN apt-get install -y wkhtmltopdf
RUN wget -nv https://downloads.wkhtmltopdf.org/0.12/0.12.5/wkhtmltox_0.12.5-1.stretch_amd64.deb
RUN apt install -y ./wkhtmltox_0.12.5-1.stretch_amd64.deb
RUN pip install pdfkit
```

For some reason (there are more people talking about this issue) the headless part does not work out-of-the-box due to some upstream problem of a dependency.

## Generate the PDF

```python
import pdfkit
# ...Import other Flask stuff

# Render the HTML
print_html = render_template('print.html')

# Convert the HTML to PDF, as target filename give 'False'
pdf = pdfkit.from_string(print_html, False)

# Convert the bytes to a file(like)
file = io.BytesIO(pdf)

# Optional: add a timestamp to the generated file.
created_on = current_time_local().strftime('%Y-%m-%d %H:%M:%S')
filename = f"Filename ({created_on})"

# Output to the browser
return send_file(file,
                 attachment_filename=filename,
                 mimetype='application/pdf',
                 # Give this argument to let the user stay on the current page
                 as_attachment=True,
                 # Set a low cache timeout
                 cache_timeout=-1)
```

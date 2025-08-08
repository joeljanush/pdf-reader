sudo apt update
sudo apt install python3-pip
pip3 install flask PyMuPDF

pip3 install --break-system-packages flask PyMuPDF

cd ~                      # Go to your home folder (or wherever you want)
mkdir pdf_reader_site     # Make a new folder for your project
cd pdf_reader_site        # Go into that folder

mkdir templates

nano app.py

from flask import Flask, render_template, request
import fitz  # PyMuPDF
import os

app = Flask(__name__)
UPLOAD_FOLDER = 'uploads'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

# Ensure upload folder exists
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@app.route('/', methods=['GET', 'POST'])
def index():
    pdf_text = ''
    if request.method == 'POST':
        pdf_file = request.files['pdf']
        if pdf_file and pdf_file.filename.endswith('.pdf'):
            filepath = os.path.join(app.config['UPLOAD_FOLDER'], pdf_file.filename)
            pdf_file.save(filepath)

            # Read PDF using PyMuPDF
            doc = fitz.open(filepath)
            for page in doc:
                pdf_text += page.get_text()

    return render_template('index.html', pdf_text=pdf_text)

if __name__ == '__main__':
    app.run(debug=True)

------------------------------------------------------------------------------

nano templates/index.html

<!DOCTYPE html>
<html>
<head>
    <title>PDF Reader</title>
</head>
<body>
    <h1>Upload and Read a PDF</h1>
    <form method="POST" enctype="multipart/form-data">
        <input type="file" name="pdf" accept=".pdf" required>
        <input type="submit" value="Upload">
    </form>

    {% if pdf_text %}
        <h2>PDF Content:</h2>
        <pre>{{ pdf_text }}</pre>
    {% endif %}
</body>
</html>

------------------------------------------------------------------------------

tree

pdf_reader_site/
├── app.py
└── templates/
    └── index.html

python3 app.py

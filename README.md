# Resume-Generater
简历生成器(高质量)
from flask import Flask, render_template, request, send_file
import pdfkit
import openai
import os
from dotenv import load_dotenv

load_dotenv()
app = Flask(__name__)

# DeepSeek配置
openai.api_key = os.getenv("DEEPSEEK_API_KEY")

# PDF工具路径（必须修改！）
pdf_config = pdfkit.configuration(
    wkhtmltopdf=r'C:\Program Files\wkhtmltopdf\bin\wkhtmltopdf.exe'
)

@app.route('/')
def index():
    return render_template('form.html')

@app.route('/generate', methods=['POST'])
def generate():
    try:
        data = request.form
        response = openai.ChatCompletion.create(
            model="deepseek-chat",
            messages=[{
                "role": "user",
                "content": f"请优化简历内容：{data['experience']}"
            }]
        )
        advice = response.choices[0].message.content
        html = render_template('template.html', 
                             name=data['name'],
                             experience=data['experience'],
                             advice=advice)
        pdf = pdfkit.from_string(html, False, configuration=pdf_config)
        return send_file(pdf, download_name="resume.pdf")
    except Exception as e:
        return f"错误：{str(e)}"

if __name__ == '__main__':
    app.run(debug=True)
    

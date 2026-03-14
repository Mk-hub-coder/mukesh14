# mukesh14
14 march heackthon
from flask import Flask, request, render_template_string
import PyPDF2

app = Flask(__name__)

# Skill Database
skills_db = [
"python","java","c++","machine learning","data science","react",
"node","sql","aws","docker","kubernetes","deep learning","nlp",
"pandas","tensorflow","html","css","javascript","git"
]

# HTML Page
html = """
<!DOCTYPE html>
<html>
<head>
<title>AI Resume Analyzer</title>
<style>
body{
font-family:Arial;
background:#f0f2f5;
padding:40px;
}
.container{
background:white;
padding:30px;
border-radius:10px;
max-width:700px;
margin:auto;
}
textarea{
width:100%;
height:120px;
}
button{
padding:10px 20px;
background:#007BFF;
color:white;
border:none;
border-radius:5px;
}
.result{
margin-top:20px;
background:#eef;
padding:20px;
border-radius:8px;
}
</style>
</head>
<body>

<div class="container">

<h2>AI Resume Analyzer</h2>

<form method="POST" enctype="multipart/form-data">

Upload Resume (PDF)
<br><br>

<input type="file" name="resume" required>

<br><br>

Paste Job Description
<br>

<textarea name="jd"></textarea>

<br><br>

<button type="submit">Analyze Resume</button>

</form>

{% if score %}

<div class="result">

<h3>Resume Score: {{score}} / 10</h3>

<h3>Extracted Skills</h3>
<p>{{skills}}</p>

<h3>Missing Skills (Based on JD)</h3>
<p>{{missing}}</p>

<h3>Suggestions</h3>
<ul>
{% for s in suggestions %}
<li>{{s}}</li>
{% endfor %}
</ul>

<h3>Professional Summary</h3>
<p>{{summary}}</p>

</div>

{% endif %}

</div>

</body>
</html>
"""

# Extract text from PDF
def extract_text(file):

    reader = PyPDF2.PdfReader(file)

    text = ""

    for page in reader.pages:
        text += page.extract_text()

    return text.lower()


# Extract skills
def extract_skills(text):

    found = []

    for skill in skills_db:

        if skill in text:
            found.append(skill)

    return found


# Resume Score
def score_resume(text, skills):

    score = 0

    if len(skills) >= 5:
        score += 3

    if "project" in text:
        score += 2

    if "experience" in text:
        score += 2

    if "education" in text:
        score += 1

    if len(text.split()) > 300:
        score += 2

    return min(score,10)


# Suggestions
def suggestions(score, skills):

    tips = []

    if score < 8:
        tips.append("Improve project descriptions using action verbs.")

    if len(skills) < 5:
        tips.append("Add more technical skills relevant to the job.")

    tips.append("Include quantified achievements.")

    tips.append("Keep resume clear and ATS friendly.")

    return tips


# JD Skill Comparison
def compare_jd(resume_skills, jd):

    jd = jd.lower()

    missing = []

    for skill in skills_db:

        if skill in jd and skill not in resume_skills:
            missing.append(skill)

    return missing


# Professional Summary
def generate_summary(skills):

    if len(skills) == 0:

        return "Motivated professional seeking opportunities to apply technical knowledge and grow in a challenging environment."

    return f"Results-driven developer skilled in {', '.join(skills[:3])}. Passionate about building scalable and impactful technology solutions."


@app.route("/", methods=["GET","POST"])
def index():

    if request.method == "POST":

        file = request.files["resume"]

        jd = request.form["jd"]

        text = extract_text(file)

        skills = extract_skills(text)

        score = score_resume(text, skills)

        tips = suggestions(score, skills)

        missing = compare_jd(skills, jd)

        summary = generate_summary(skills)

        return render_template_string(
            html,
            score=score,
            skills=skills,
            missing=missing,
            suggestions=tips,
            summary=summary
        )

    return render_template_string(html)


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=False)

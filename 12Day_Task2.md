# 🎓 Aditya University — AI Boot Camp 2026
# Task 2: AI Faculty Assistant using Streamlit + Groq

---

## 📌 Problem Statement
Develop a simple AI-powered Faculty Assistant using Streamlit and Groq that helps faculty generate teaching content.

---

## 🛠️ Tools Used
- **Streamlit** — Web UI framework
- **Groq API** — LLaMA 3.3 70B model
- **Python** — Backend logic

---

## 📦 Step 1: Install Dependencies

Run in Command Prompt:
```bash
pip install streamlit groq
```

### requirements.txt
```
streamlit
groq
```

---

## 🔑 Step 2: Get Groq API Key
1. Go to https://console.groq.com/keys
2. Click "Create API Key"
3. Copy the key (starts with `gsk_...`)

---

## 💻 Step 3: Complete Source Code (app.py)

```python
import streamlit as st
from groq import Groq
import json

# Configure Groq API
client = Groq(api_key="gsk_YOUR_GROQ_API_KEY")

# Page Configuration
st.set_page_config(
    page_title="AI Faculty Assistant",
    page_icon="🎓",
    layout="wide"
)

# Title
st.title("🎓 AI Faculty Assistant")
st.subheader("Generate Teaching Content Instantly")
st.divider()

# Topic Input
topic = st.text_input(
    "📚 Enter Topic:",
    placeholder="e.g. Artificial Intelligence in Education"
)

# Generate Button
if st.button("⚡ Generate Content", type="primary"):
    if not topic:
        st.warning("⚠️ Please enter a topic first!")
    else:
        with st.spinner("🤖 Generating content..."):
            try:
                # Call Groq API
                response = client.chat.completions.create(
                    model="llama-3.3-70b-versatile",
                    messages=[
                        {
                            "role": "system",
                            "content": "You are an educational content generator. Always respond with valid JSON only. No extra text, no markdown, no code blocks."
                        },
                        {
                            "role": "user",
                            "content": f"""
                            Generate educational content for the topic: {topic}

                            Return ONLY this JSON structure:
                            {{
                                "learning_objectives": [
                                    "objective 1",
                                    "objective 2",
                                    "objective 3"
                                ],
                                "lecture_outline": [
                                    "1. Introduction",
                                    "2. Main Concept",
                                    "3. Applications",
                                    "4. Examples",
                                    "5. Conclusion"
                                ],
                                "mcqs": [
                                    {{
                                        "question": "Question 1?",
                                        "options": ["A. opt1", "B. opt2", "C. opt3", "D. opt4"],
                                        "answer": "A"
                                    }},
                                    {{
                                        "question": "Question 2?",
                                        "options": ["A. opt1", "B. opt2", "C. opt3", "D. opt4"],
                                        "answer": "B"
                                    }},
                                    {{
                                        "question": "Question 3?",
                                        "options": ["A. opt1", "B. opt2", "C. opt3", "D. opt4"],
                                        "answer": "C"
                                    }},
                                    {{
                                        "question": "Question 4?",
                                        "options": ["A. opt1", "B. opt2", "C. opt3", "D. opt4"],
                                        "answer": "D"
                                    }},
                                    {{
                                        "question": "Question 5?",
                                        "options": ["A. opt1", "B. opt2", "C. opt3", "D. opt4"],
                                        "answer": "A"
                                    }}
                                ],
                                "summary": "A short 3-4 sentence summary of the topic."
                            }}
                            """
                        }
                    ],
                    temperature=0.7
                )

                # Parse Response
                raw = response.choices[0].message.content
                cleaned = raw.replace("```json", "").replace("```", "").strip()
                data = json.loads(cleaned)

                # Display Results
                st.success("✅ Content Generated Successfully!")
                st.divider()

                # Learning Objectives
                st.subheader("🎯 Learning Objectives")
                for i, obj in enumerate(data["learning_objectives"], 1):
                    st.write(f"**{i}.** {obj}")

                st.divider()

                # Lecture Outline
                st.subheader("📋 Lecture Outline")
                for item in data["lecture_outline"]:
                    st.write(f"✅ {item}")

                st.divider()

                # MCQs
                st.subheader("❓ Multiple Choice Questions")
                for i, mcq in enumerate(data["mcqs"], 1):
                    st.write(f"**Q{i}: {mcq['question']}**")
                    for opt in mcq["options"]:
                        st.write(f"　　{opt}")
                    st.success(f"✔️ Answer: {mcq['answer']}")
                    st.write("")

                st.divider()

                # Summary
                st.subheader("📝 Topic Summary")
                st.info(data["summary"])

            except Exception as e:
                st.error(f"❌ Error: {str(e)}")

st.divider()
st.caption("Powered by Groq LLaMA AI | Aditya University AI BootCamp 2026")
```

---

## ▶️ Step 4: Run the App

```bash
streamlit run app.py
```

Then open browser at:
```
http://localhost:8501
```

---

## 🖥️ Expected User Interface

| Element | Description |
|---|---|
| 📚 Topic Input | Text box to enter any topic |
| ⚡ Generate Button | Click to generate content |
| 🎯 Learning Objectives | 3 objectives displayed |
| 📋 Lecture Outline | 5 outline points displayed |
| ❓ MCQs | 5 questions with options and answers |
| 📝 Summary | Short topic summary |

---

## 🧪 Sample Input & Output

### Input
```
Artificial Intelligence in Education
```

### Output
```json
{
  "learning_objectives": [
    "Understand the role of AI in modern education",
    "Identify key AI tools used in teaching and learning",
    "Evaluate the impact of AI on student outcomes"
  ],
  "lecture_outline": [
    "1. Introduction to AI in Education",
    "2. AI-Powered Learning Tools",
    "3. Personalized Learning with AI",
    "4. Real-world Examples and Case Studies",
    "5. Future of AI in Education"
  ],
  "mcqs": [
    {
      "question": "What is the primary benefit of AI in education?",
      "options": [
        "A. Replacing teachers",
        "B. Personalized learning",
        "C. Reducing school hours",
        "D. Eliminating exams"
      ],
      "answer": "B"
    }
  ],
  "summary": "AI in education refers to the use of artificial intelligence technologies to enhance teaching and learning experiences. It enables personalized learning paths, automates administrative tasks, and provides intelligent tutoring systems. AI tools help educators create better content and give students instant feedback."
}
```

---

## 📝 Brief Explanation
This Streamlit application serves as an AI-powered Faculty Assistant that generates structured teaching content using Groq LLaMA AI. Faculty members can enter any topic and instantly receive learning objectives, a lecture outline, five MCQs with answers, and a topic summary. The app uses the Groq API to process natural language prompts and return structured JSON responses. It provides a clean, interactive interface making it easy for educators to prepare teaching materials quickly.

---

## 🔑 API Key
Get your free Groq API key from: https://console.groq.com/keys

> ⚠️ Never share or commit your API key to GitHub!

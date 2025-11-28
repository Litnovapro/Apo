# backend.py
import base64
from flask import Flask, request, jsonify
import google.generativeai as genai

# ----------------------------------------
# CONFIGURE API KEY
# ----------------------------------------
API_KEY = "YOUR_GEMINI_API_KEY"   # <-- Replace with your key
genai.configure(api_key=API_KEY)

# Models
TEXT_MODEL = genai.GenerativeModel("gemini-pro")
VISION_MODEL = genai.GenerativeModel("gemini-pro-vision")

app = Flask(__name__)

# ----------------------------------------
# TEXT GENERATION ENDPOINT
# ----------------------------------------
@app.route("/text", methods=["POST"])
def text_generation():
    data = request.json
    prompt = data.get("prompt", "")

    if not prompt:
        return jsonify({"error": "No prompt provided"}), 400

    response = TEXT_MODEL.generate_content(prompt)
    return jsonify({"response": response.text})


# ----------------------------------------
# IMAGE + TEXT ENDPOINT (Vision)
# ----------------------------------------
@app.route("/vision", methods=["POST"])
def vision():
    prompt = request.form.get("prompt", "")

    if "image" not in request.files:
        return jsonify({"error": "Image file missing"}), 400

    img_file = request.files["image"].read()
    img_data = base64.b64encode(img_file).decode("utf-8")

    response = VISION_MODEL.generate_content(
        [
            prompt,
            {"mime_type": "image/jpeg", "data": img_data}
        ]
    )

    return jsonify({"response": response.text})


# ----------------------------------------
# HOME ROUTE
# ----------------------------------------
@app.route("/", methods=["GET"])
def home():
    return "Gemini Backend Running Successfully"

# ----------------------------------------
# RUN SERVER
# ----------------------------------------
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

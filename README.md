# minor-project-
plant disease detection system
from flask import Flask, render_template, request, redirect, url_for
import tensorflow as tf      
from tensorflow.keras.preprocessing import image
import numpy as np
import os
 
app = Flask(_name_)
model = tf.keras.models.load_model("leaf_disease_model.h5")
UPLOAD_FOLDER = "static/uploaded"
app.config["UPLOAD_FOLDER"] = UPLOAD_FOLDER
        
# Disease classes
class_names = [
    "Asian_Soybean_Rust",
    "Sclerotinia_Stem_Rot",
    "Rhizoctonia_Root_Rot",
    "Fusarium_Wilt",
    "Septoria_Brown_Spot",   
    "Downy_Mildew"
]
        
# Disease details (PDF se mapped)
disease_info = {
    "Asian_Soybean_Rust": {
        "Occurrence": "Occurs during warm, humid weather.",
        "Prevention": "Plant resistant varieties, rotate crops.",
        "Medicine": "Use fungicides like Azoxystrobin."
    },
    "Sclerotinia_Stem_Rot": {
        "Occurrence": "Seen in mid to late crop stages with moist soil.",
        "Prevention": "Improve air circulation and avoid overhead irrigation.”,
        "Medicine": "Apply Boscalid or other fungicides."
    },
    "Rhizoctonia_Root_Rot": {
        "Occurrence": "Common in poorly drained soils.",
        "Prevention": "Use well-drained seedbeds, crop rotation.",
        "Medicine": "Use fungicide seed treatments."
    },
    "Fusarium_Wilt": {
        "Occurrence": "Fungal infection in early plant stages.",
        "Prevention": "Sterilize soil, use disease-free seeds.",
        "Medicine": "Apply Carbendazim or Thiophanate-methyl."   
    },
    "Septoria_Brown_Spot": {
        "Occurrence": "Caused by Septoria glycines in moist weather.",
        "Prevention": "Plant clean seeds, use wide row spacing.",

        "Medicine": "Use Mancozeb or Chlorothalonil.”
 },
    "Downy_Mildew": {
        "Occurrence": "Thrives in cool, moist conditions.",
        "Prevention": "Avoid overcrowding, ensure sunlight.",
        "Medicine": "Spray with Metalaxyl or Fosetyl-Al."
    }
}
    
@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        file = request.files["image"]
        if file:
            filepath = os.path.join(app.config["UPLOAD_FOLDER"], file.filename)
            file.save(filepath)

            try:
        if not file.filename.lower().endswith(('.jpg', '.jpeg', '.png')):
                    print("❌ Unsupported file format")
                    return redirect(url_for("unknown", img=file.filename))
        
                img = image.load_img(filepath, target_size=(224, 224))
                img_array = image.img_to_array(img)
                img_array = np.expand_dims(img_array, axis=0) / 255.0
    
                prediction = model.predict(img_array)[0]
                confidence = np.max(prediction)
                predicted_class = class_names[np.argmax(prediction)]
        
                print("🧠 Predicted Class:", predicted_class)
                print("🎯 Confidence Score:", confidence)
            
                if confidence < 0.8 or predicted_class not in class_names:
          print("⚠ Low confidence or invalid class → unknown")
                    return redirect(url_for("unknown", img=file.filename))
                else:
                    return redirect(url_for("result", disease=predicted_class, img=file.filename))
                
            except Exception as e:
                print("💥 Image processing error:", e)
                return redirect(url_for("unknown", img=file.filename))
                
    return render_template("index.html")
                
@app.route("/result")
def result():
    disease = request.args.get("disease")
    img = request.args.get("img")

    info = disease_info[disease]
 return render_template("result.html", disease=disease, info=info, img=img)
                    
@app.route("/unknown")
def unknown():
    img = request.args.get("img")
    return render_template("unknown.html", img=img)
                
if _name_ == "_main_":
    app.run(debug=True)

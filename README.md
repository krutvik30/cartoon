# cartoon
from flask import Flask, request, jsonify, send_file
import cv2
import os
import numpy as np
from moviepy.editor import VideoFileClip

app = Flask(__name__)
UPLOAD_FOLDER = "uploads"
OUTPUT_FOLDER = "outputs"
os.makedirs(UPLOAD_FOLDER, exist_ok=True)
os.makedirs(OUTPUT_FOLDER, exist_ok=True)

def cartoonify_frame(frame):
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    blurred = cv2.medianBlur(gray, 5)
    edges = cv2.adaptiveThreshold(blurred, 255, cv2.ADAPTIVE_THRESH_MEAN_C, cv2.THRESH_BINARY, 9, 9)
    color = cv2.bilateralFilter(frame, 9, 300, 300)
    cartoon = cv2.bitwise_and(color, color, mask=edges)
    return cartoon

def process_video(input_path, output_path):
    clip = VideoFileClip(input_path)
    def process_frame(frame):
        return cartoonify_frame(frame)
    new_clip = clip.fl_image(process_frame)
    new_clip.write_videofile(output_path, codec='libx264')

@app.route("/upload", methods=["POST"])
def upload_video():
    if "video" not in request.files:
        return jsonify({"error": "No video file uploaded"}), 400
    
    file = request.files["video"]
    filename = os.path.join(UPLOAD_FOLDER, file.filename)
    output_filename = os.path.join(OUTPUT_FOLDER, "cartoon_" + file.filename)
    
    file.save(filename)
    process_video(filename, output_filename)
    
    return send_file(output_filename, as_attachment=True)

if __name__ == "__main__":
    app.run(debug=True)

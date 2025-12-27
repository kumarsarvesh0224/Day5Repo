pip install google-generativeai pillow opencv-python numpy


import cv2
import numpy as np
from PIL import Image
import google.generativeai as genai

genai.configure(api_key="YOUR_API_KEY")

class CPUTextToVideo:
    def __init__(self):
        self.model = genai.GenerativeModel("models/gemini-flash-latest")

    def generate_script(self, text):
        response = self.model.generate_content(
            f"Write a cinematic visual description for: {text}"
        )
        return response.text

    def generate_image_prompt(self, script):
        response = self.model.generate_content(
            f"Convert this into an image prompt:\n{script}"
        )
        return response.text

    def generate_image(self, prompt):
        # Gemini image generation
        model = genai.GenerativeModel(
            "models/gemini-2.5-flash-image-preview"
        )
        response = model.generate_content(prompt)

        image_bytes = response.candidates[0].content.parts[0].inline_data.data
        image = Image.open(
            bytes_to_image(image_bytes)
        )
        image.save("frame.png")
        return image

    def animate_image(self, image, out_path="output.mp4"):
        img = np.array(image)
        h, w, _ = img.shape

        fourcc = cv2.VideoWriter_fourcc(*"mp4v")
        video = cv2.VideoWriter(out_path, fourcc, 8, (w, h))

        for i in range(60):
            scale = 1 + i * 0.002
            resized = cv2.resize(
                img,
                None,
                fx=scale,
                fy=scale,
                interpolation=cv2.INTER_LINEAR
            )
            y = (resized.shape[0] - h) // 2
            x = (resized.shape[1] - w) // 2
            frame = resized[y:y+h, x:x+w]
            video.write(frame)

        video.release()

    def run(self, text):
        script = self.generate_script(text)
        prompt = self.generate_image_prompt(script)
        image = self.generate_image(prompt)
        self.animate_image(image)
        print("🎉 Video generated: output.mp4")

def bytes_to_image(data):
    from io import BytesIO
    return BytesIO(data)

if __name__ == "__main__":
    pipeline = CPUTextToVideo()
    pipeline.run("A red Ferrari driving on a coastal highway at sunset")

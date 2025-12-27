pip install torch torchvision diffusers transformers accelerate safetensors pillow opencv-python



import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
from diffusers import StableDiffusionPipeline
from PIL import Image
import numpy as np
import cv2
import os

# Config
DEVICE = "cpu"
IMAGE_SIZE = 256
NUM_FRAMES = 30
FPS = 8

os.makedirs("outputs", exist_ok=True)

# -------------------------------
# Step 1: Text Generation (Local GPT2)
# -------------------------------
tokenizer = AutoTokenizer.from_pretrained("gpt2")
model = AutoModelForCausalLM.from_pretrained("gpt2").to(DEVICE)

def generate_script(prompt):
    inputs = tokenizer(prompt, return_tensors="pt").to(DEVICE)
    outputs = model.generate(inputs["input_ids"], max_length=60)
    return tokenizer.decode(outputs[0], skip_special_tokens=True)

prompt = "A red Ferrari driving on a coastal highway at sunset."
script = generate_script(prompt)
print("Generated script:", script)

# -------------------------------
# Step 2: Text → Image (Stable Diffusion)
# -------------------------------
pipe = StableDiffusionPipeline.from_pretrained(
    "runwayml/stable-diffusion-v1-5", torch_dtype=torch.float32
).to(DEVICE)

image = pipe(script, height=IMAGE_SIZE, width=IMAGE_SIZE, num_inference_steps=25).images[0]
image.save("outputs/keyframe.png")

# -------------------------------
# Step 3: Image → Video (PIL + OpenCV)
# -------------------------------
img = np.array(image)
h, w, _ = img.shape
video = cv2.VideoWriter("outputs/output.mp4", cv2.VideoWriter_fourcc(*"mp4v"), FPS, (w,h))

for i in range(NUM_FRAMES):
    scale = 1 + i*0.002  # simple zoom
    resized = cv2.resize(img, None, fx=scale, fy=scale, interpolation=cv2.INTER_LINEAR)
    y = (resized.shape[0]-h)//2
    x = (resized.shape[1]-w)//2
    frame = resized[y:y+h, x:x+w]
    video.write(frame)

video.release()
print("🎉 Video saved at outputs/output.mp4")

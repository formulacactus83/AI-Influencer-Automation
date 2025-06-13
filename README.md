AI Influencer Bot

An automated AI-generated Instagram influencer pipeline with face swapping, image generation, and posting automation.

---------------------------
Features
---------------------------

- Image Generation: Create realistic images using Fooocus on Google Colab (free).
- Face Swapping: Ensure a consistent character with Roop’s open-source face-swapping tool.
- Automated Posting: Schedule daily Instagram posts with instagrapi, including dynamic captions.
- Stock Monitoring: Alerts when image stock is low, prompting regeneration.
- Monetization: Promote content on Fanvue’s free tier (manual uploads).
- No Videos: Focuses solely on image-based posts for simplicity.

---------------------------
Prerequisites
---------------------------

- Windows, Mac, or Linux with 8GB RAM (CPU works; NVIDIA GPU speeds up Roop).
- Google Account: For Google Colab and Drive (15GB free storage).
- Instagram Account: Free account with bio indicating “AI Influencer” or “Digital Creator”.
- Fanvue Account: Free account at fanvue.com, marked “AI Created”.
- Python 3.8 or higher.
- Git: For version control and cloning repositories.
- VS Code: Recommended code editor.
- Internet: Stable connection for Colab and Instagram.

---------------------------
Installation
---------------------------

1. Set Up VS Code and Python

- Download and install VS Code.
- Install Python 3.8+:
  - Windows: check “Add Python to PATH” during installation.
  - Mac/Linux: Follow default prompts.

2. Create Project Directory

- Create a folder (e.g., AIInfluencer).
- Inside it, create subfolders:
  - images
  - swapped
  - scripts

3. Install Git

- Windows: Download from git-scm.com.
- Mac: brew install git
- Linux: sudo apt install git (Ubuntu)

4. Clone Roop

cd AIInfluencer
git clone https://github.com/s0md3v/roop
cd roop
pip install -r requirements.txt

- Download inswapper_128.onnx from Roop Releases and place it in AIInfluencer/roop/models/.

5. Install Python Libraries

pip install instagrapi pillow

---------------------------
Setup
---------------------------

1. Generate or Collect Images

Option 1: Use Existing Images

- Obtain ~200 AI-generated images with creator’s consent.
- Select one high-quality face image as base_face.png.
- Save all images to AIInfluencer/images/
- Upload to Google Drive: /MyDrive/ai_girl/images/

Option 2: Generate Images with Fooocus

- Open Google Colab, create a new notebook.
- Mount Google Drive:

from google.colab import drive
drive.mount('/content/drive')

- Install Fooocus:

!pip install pygit2==1.12.2
!git clone https://github.com/lllyasviel/Fooocus.git
%cd Fooocus
!python entry_with_update.py --share

- Run, click the gradio.app URL.
- Generate prompts:

import random
import os

outfits = ["light blue dress", "casual jeans and t-shirt", "red bikini", "black workout gear", "white summer dress", "floral skirt", "leather jacket", "pink sweater"]
locations = ["sunny beach", "city park", "cozy cafe", "gym", "urban rooftop", "mountain trail", "night market", "flower garden"]
poses = ["standing confidently", "sitting casually", "walking dynamically", "posing playfully", "smiling warmly"]

output_dir = "/content/drive/MyDrive/ai_girl/images"
os.makedirs(output_dir, exist_ok=True)

existing_images = [f for f in os.listdir(output_dir) if f.endswith('.png') and f != 'base_face.png']
prompts = []
for i in range(200 - len(existing_images)):
    outfit = random.choice(outfits)
    location = random.choice(locations)
    pose = random.choice(poses)
    prompt = f"A 22-year-old woman, blue eyes, long brown hair, wearing {outfit}, {pose} in a {location}, sunny day, realistic, ultra-detailed, full-body shot"
    prompts.append(prompt)
    print(f"Prompt {i+1}: {prompt}")

- In Fooocus, copy each prompt, set Style=Photograph, Quality=High, Upscale=2x.
- Save images to Google Drive folder.
- Download images to AIInfluencer/images/

2. Face Swap with Roop

- Create face_swap.py in AIInfluencer/scripts/:

import os
import subprocess

input_dir = "../images/"
output_dir = "../swapped/"
base_face = "../images/base_face.png"
os.makedirs(output_dir, exist_ok=True)

for img in os.listdir(input_dir):
    if img.endswith(".png") and img != "base_face.png":
        input_path = os.path.join(input_dir, img)
        output_path = os.path.join(output_dir, f"swapped_{img}")
        subprocess.run([
            "python", "../roop/run.py",
            "--target", input_path,
            "--source", base_face,
            "--output", output_path
        ])
        print(f"Swapped face for {img}")

- Run:

cd scripts
python face_swap.py

3. Configure Instagram Posting

- Create post_to_instagram.py in AIInfluencer/scripts/:

import os
import random
from instagrapi import Client
import datetime

USERNAME = "your_instagram_username"
PASSWORD = "your_instagram_password"

IMAGE_DIR = "../swapped/"
CAPTIONS = [
    "Living the digital dream 🌟 #AIInfluencer #DigitalCreator",
    "Virtual vibes only! 💖 #AIGirl #Fanvue",
    "Join my Fanvue for more! Link in bio ✨ #AIModel",
    "Spreading joy in the digital world 😊 #AIInfluencer",
    "Chasing virtual sunsets 🌅 #AIGirl #DigitalLife",
]

def login_instagram():
    cl = Client()
    try:
        cl.login(USERNAME, PASSWORD)
    except Exception as e:
        print(f"Login failed: {e}")
        exit(1)
    return cl

def check_stock():
    images = [f for f in os.listdir(IMAGE_DIR) if f.endswith('.png')]
    if len(images) < 10:
        print(f"WARNING: Only {len(images)} images left! Regenerate more in Fooocus.")
    return len(images) > 0

def select_content():
    images = [os.path.join(IMAGE_DIR, f) for f in os.listdir(IMAGE_DIR) if f.endswith('.png')]
    if images:
        content_path = random.choice(images)
        os.remove(content_path)
        return content_path
    return None

def generate_caption():
    return random.choice(CAPTIONS)

def post_to_instagram():
    if not check_stock():
        print("ERROR: No images left. Regenerate in Fooocus.")
        exit(1)
    cl = login_instagram()
    content_path = select_content()
    if not content_path:
        print("ERROR: No content available.")
        exit(1)
    caption = generate_caption()
    try:
        cl.photo_upload(content_path, caption=caption)
        print(f"Posted image: {content_path} at {datetime.datetime.now()}")
    except Exception as e:
        print(f"Post failed: {e}")

if __name__ == "__main__":
    post_to_instagram()

- Replace your_instagram_username and your_instagram_password.

- Run:

cd scripts
python post_to_instagram.py

4. Schedule Automation

Linux/Mac: use crontab:

0 12 * * * python3 /path/to/AIInfluencer/scripts/post_to_instagram.py

Windows: use Task Scheduler.

5. Set Up Fanvue

- Sign up at fanvue.com, select “AI Created”.
- Upload 5-10 images from AIInfluencer/swapped/ weekly.
- Add description: “Exclusive AI content! Subscribe for more!”.
- Link Fanvue in Instagram bio.

---------------------------
Usage
---------------------------

- Generate or collect ~200 images and run face_swap.py (one-time).
- Configure post_to_instagram.py with Instagram credentials.
- Schedule post_to_instagram.py to run daily.
- Upload content to Fanvue weekly.
- Monitor Instagram weekly for engagement.

---------------------------
Maintenance
---------------------------

- Image Stock: Regenerate when <10 images (script warns).
- Captions: Update monthly with trending hashtags.
- Instagram Safety: Post every 1-2 days to avoid bans.
- Fanvue: Upload weekly (~10 min).

---------------------------
Troubleshooting
---------------------------

- Colab Disconnects: Free tier limits runtime.
- Roop Slow: CPU takes longer; run overnight.
- Instagram Login Fails: Verify credentials, enable 2FA.
- Low Engagement: Adjust prompts or hashtags.

---------------------------
Notes
---------------------------

- Transparency: Add “AI-generated” to Instagram/Fanvue bios.
- Legal: Use only images with permission and original prompts.
- Storage: Use Google Drive (15GB free) or local storage.

---------------------------
Contributing
---------------------------

- Fork the repository.
- Create a feature branch: git checkout -b feature/your-feature.
- Commit changes: git commit -m "Add your feature".
- Push: git push origin feature/your-feature.
- Open a pull request.

---------------------------
License
---------------------------

MIT License. See LICENSE for details.

---------------------------
Resources
---------------------------

- VS Code
- Fooocus
- Roop
- Instagrapi
- Fanvue Help

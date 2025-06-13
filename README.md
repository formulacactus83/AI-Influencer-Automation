# AI Influencer Bot

An automated pipeline for creating an AI-generated Instagram influencer, featuring image generation, face swapping, and scheduled posting.

![AI Influencer Example](https://via.placeholder.com/800x200.png?text=AI+Influencer+Bot) <!-- Placeholder image -->

## Table of Contents
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Setup](#setup)
- [Usage](#usage)
- [Maintenance](#maintenance)
- [Troubleshooting](#troubleshooting)
- [Notes](#notes)
- [Contributing](#contributing)
- [License](#license)
- [Resources](#resources)

## Features

- **Image Generation**: Generate realistic images using [Fooocus](https://github.com/lllyasviel/Fooocus) on Google Colab (free).
- **Face Swapping**: Maintain a consistent character appearance with [Roop](https://github.com/s0md3v/roop)’s open-source face-swapping tool.
- **Automated Posting**: Schedule daily Instagram posts with [instagrapi](https://instagrapi.readthedocs.io), including dynamic captions.
- **Stock Monitoring**: Receive alerts when image stock is low, prompting regeneration.
- **Monetization**: Promote content on [Fanvue](https://fanvue.com)’s free tier (manual uploads).
- **No Videos**: Focuses solely on image-based posts for simplicity.

## Prerequisites

- **Computer**: Windows, Mac, or Linux with 8GB RAM (CPU works; NVIDIA GPU accelerates Roop).
- **Google Account**: For Google Colab and Drive (15GB free storage).
- **Instagram Account**: Free account with bio indicating “AI Influencer” or “Digital Creator”.
- **Fanvue Account**: Free account at [fanvue.com](https://fanvue.com), marked “AI Created”.
- **Python**: Version 3.8 or higher ([python.org](https://www.python.org/downloads)).
- **Git**: For version control ([git-scm.com](https://git-scm.com)).
- **VS Code**: Recommended code editor ([code.visualstudio.com](https://code.visualstudio.com)).
- **Internet**: Stable connection for Colab and Instagram.

## Installation

1. **Set Up VS Code and Python**
   - Download and install [VS Code](https://code.visualstudio.com).
   - Install [Python 3.8+](https://www.python.org/downloads):
     - **Windows**: Check “Add Python to PATH” during installation.
     - **Mac/Linux**: Follow default prompts.
   - Open VS Code, install the Python extension (search “Python” by Microsoft in Extensions).
   - Verify Python: In VS Code’s terminal (`Terminal > New Terminal`), run:
     ```bash
     python --version
     ```

2. **Create Project Directory**
   - Create a folder (e.g., `AIInfluencer`).
   - Open in VS Code: `File > Open Folder`.
   - Create subfolders:
     ```
     images/
     swapped/
     scripts/
     ```

3. **Install Git**
   - **Windows**: Download from [git-scm.com](https://git-scm.com).
   - **Mac**: Run:
     ```bash
     brew install git
     ```
     Install [Homebrew](https://brew.sh) if needed.
   - **Linux**: Run:
     ```bash
     sudo apt install git
     ```

4. **Clone Roop**
   ```bash
   cd AIInfluencer
   git clone https://github.com/s0md3v/roop
   cd roop
   pip install -r requirements.txt
   ```
   - Download `inswapper_128.onnx` from [Roop Releases](https://github.com/s0md3v/roop/releases) and place in `AIInfluencer/roop/models/`.

5. **Install Python Libraries**
   ```bash
   pip install instagrapi pillow
   ```

## Setup

### 1. Generate or Collect Images
You need ~200 images for ~6 months of daily posts.

#### Option 1: Use Existing Images
- Obtain ~200 AI-generated images with creator’s written consent (save proof, e.g., email).
- Select one high-quality face image as `base_face.png`.
- Save all images to `AIInfluencer/images/`.
- Upload to Google Drive: `/MyDrive/ai_girl/images/`.

#### Option 2: Generate Images with Fooocus
1. Open [Google Colab](https://colab.google), create a new notebook.
2. Mount Google Drive:
   ```python
   from google.colab import drive
   drive.mount('/content/drive')
   ```
3. Install Fooocus:
   ```python
   !pip install pygit2==1.12.2
   !git clone https://github.com/lllyasviel/Fooocus.git
   %cd Fooocus
   !python entry_with_update.py --share
   ```
   Run (~5-10 minutes), click the `*.gradio.app` URL.
4. Generate prompts:
   ```python
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
   ```
5. In Fooocus’s web interface:
   - Copy each prompt, set Style=Photograph, Quality=High, Upscale=2x.
   - Save images to `/MyDrive/ai_girl/images/` (~2-3 hours).
6. Download to `AIInfluencer/images/`.

### 2. Face Swap with Roop
1. Create `AIInfluencer/scripts/face_swap.py`:
   ```python
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
   ```
2. Run:
   ```bash
   cd scripts
   python face_swap.py
   ```
   Takes ~2-3 hours for 200 images.

### 3. Configure Instagram Posting
1. Create `AIInfluencer/scripts/post_to_instagram.py`:
   ```python
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
   ```
2. Replace `your_instagram_username` and `your_instagram_password`.
3. Test:
   ```bash
   cd scripts
   python post_to_instagram.py
   ```

### 4. Schedule Automation
#### Linux/Mac
1. Open terminal:
   ```bash
   crontab -e
   ```
2. Add:
   ```bash
   0 12 * * * python3 /path/to/AIInfluencer/scripts/post_to_instagram.py
   ```
   Replace `/path/to/` with your folder path.

#### Windows
1. Open Task Scheduler (search in Windows).
2. Create Basic Task:
   - Name: “AIInfluencerPost”.
   - Trigger: Daily, 12 PM.
   - Action: Start a Program.
   - Program: `python`.
   - Arguments: `C:\AIInfluencer\scripts\post_to_instagram.py`.
   - Start in: `C:\AIInfluencer\scripts`.

### 5. Set Up Fanvue
1. Sign up at [fanvue.com](https://fanvue.com), select “AI Created”.
2. Upload 5-10 images from `AIInfluencer/swapped/` weekly.
3. Add description: “Exclusive AI content! Subscribe for more!”
4. Link Fanvue in Instagram bio.
5. Schedule releases in Fanvue’s dashboard.

## Usage
1. Generate/collect ~200 images and run `face_swap.py` (one-time).
2. Configure `post_to_instagram.py` with Instagram credentials.
3. Schedule `post_to_instagram.py` daily.
4. Upload content to Fanvue weekly.
5. Monitor Instagram weekly for engagement.

## Maintenance
- **Image Stock**: Regenerate when <10 images (script warns; ~2-3 hours every ~6 months).
- **Captions**: Update `CAPTIONS` monthly with trending hashtags ([hashtagify.me](https://hashtagify.me)).
- **Instagram Safety**: Post every 1-2 days to avoid bans. Appeal bans via Instagram.
- **Fanvue**: Upload weekly (~10 minutes).

## Troubleshooting
- **Colab Disconnects**: Free tier limits runtime. Use local PC for scripts.
- **Roop Slow**: CPU takes longer; run overnight.
- **Instagram Login Fails**: Verify credentials, enable 2FA if needed.
- **Low Engagement**: Adjust prompts or hashtags.

## Notes
- **Transparency**: Add “AI-generated” to Instagram/Fanvue bios.
- **Legal**: Use only images with permission and original prompts.
- **Storage**: Google Drive (15GB free) or local storage.

## Contributing
Contributions are welcome! Please:
1. Fork the repository.
2. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature
   ```
3. Commit changes:
   ```bash
   git commit -m "Add your feature"
   ```
4. Push:
   ```bash
   git push origin feature/your-feature
   ```
5. Open a pull request.

## License
MIT License. See [LICENSE](LICENSE) for details.

## Resources
- [VS Code](https://code.visualstudio.com)
- [Fooocus](https://github.com/lllyasviel/Fooocus)
- [Roop](https://github.com/s0md3v/roop)
- [Instagrapi](https://instagrapi.readthedocs.io)
- [Fanvue Help](https://help.fanvue.com)
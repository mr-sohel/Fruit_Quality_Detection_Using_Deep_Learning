We built a real-time AI fruit inspector using a $5 camera — here's the full story.

Most people think cutting-edge AI requires expensive hardware and enterprise infrastructure. We proved otherwise.

Our team just completed an end-to-end IoT Fruit Quality Detection System — a full pipeline that goes from a budget ESP32-CAM on the edge, all the way to a deployed AI model responding through a Telegram bot. Here's what we learned building it from scratch:

---

We started with failure (and it was worth it)

Our first model was MobileNetV2 — a solid classifier, but completely lost the moment multiple fruits appeared in one frame. It couldn't localize. It couldn't separate. It tried to classify the whole image as one object.

That failure pushed us to YOLOv8 Nano — a lightning-fast object detection model built for exactly this: detecting and labeling multiple objects simultaneously with bounding boxes, even on resource-constrained hardware.

---

Three challenges that nearly broke us

• Dirty data: Multi-fruit clusters in training images confused the model badly. We manually cleaned the dataset — and the accuracy jump was immediate and dramatic.
• Cloud latency: Deploying raw PyTorch .pt weights on Hugging Face's free tier caused brutal delays and timeouts. Converting to ONNX eliminated the PyTorch runtime entirely — response time dropped to 3–4 seconds.
• Hardware realities: The ESP32-CAM's OV2640 sensor has poor white-balance and unpredictable lighting. We simulated this during training with heavy HSV jitter augmentation — and it paid off in real-world accuracy.

---

The results speak for themselves

mAP@50: 98.97% | Precision: 98.17% | Recall: 97.79%
Nearly 99% mAP on an 8-class detection problem, running at the edge on a $5 camera. We were honestly shocked.

### Data & Labels
![Labels](model/fruit_quality/labels.jpg)
![Training Batch 0](model/fruit_quality/train_batch0.jpg)

### Evaluation Metrics
![Training History](train_history.png)
![Confusion Matrix](model/fruit_quality/confusion_matrix_normalized.png)
![Validation Predictions](model/fruit_quality/val_batch2_pred.jpg)


---

The full stack

• Edge device: ESP32-CAM + OV2640 2MP sensor
• Detection model: YOLOv8 Nano — 50 epochs, 320×320, batch 64
• Training: Intel Arc A750 GPU (with custom XPU patches for Ultralytics)
• Deployment: Hugging Face Spaces (free tier) via ONNX Runtime
• Interface: Telegram Bot — manual + automatic alert modes

---

The real-world potential

This isn't just a student project. The same architecture scales directly to conveyor-belt sorting systems, cold-chain logistics monitoring, and automated quality grading for food suppliers — all at a fraction of traditional system costs.

---

None of this happens without the team

Deepest gratitude to my teammates: Md Sohel Rana, Faisal Islam Fahad, Naushin Sultana Mim, and Jerin — for every late debugging session, every brilliant insight, and every moment of refusing to settle. I'm incredibly proud of what we built together.

To run this project on a different machine, you need to clone the project and set up a Python environment with the required dependencies. I've created a `requirements.txt` file in the project folder to make this easy!

Here is the step-by-step guide you can follow (or share with your teammates) to get the AI model running on **any Windows, Mac, or Linux computer**.

### Download dataset from [Mendeley](https://data.mendeley.com/datasets/xkbjx8959c/2)

### Step 1: Install Python
Ensure Python is installed on the new machine. 
1. Download Python 3.10 or newer from [python.org](https://www.python.org/downloads/).
2. **Important for Windows:** During installation, make sure to check the box that says 
    **"Add python.exe to PATH"** before clicking Install.

### Step 2: Open the Terminal and create a Virtual Environment
Navigate to the copied project folder in your terminal (Command Prompt/PowerShell on Windows, or Terminal on Mac/Linux). Run the following commands to create and activate an isolated environment:

**On Windows:**
```bash
python -m venv venv
venv\Scripts\activate
```

**On Mac/Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
```
*You will know it worked if you see `(venv)` at the start of your terminal prompt.*

### Step 3: Install Dependencies
With your virtual environment activated, install all the required AI packages using the `requirements.txt` file:
```bash
pip install -r requirements.txt
```
*This will automatically install `ultralytics`, `opencv-python`, and `onnxruntime` (which is necessary to run your fast `.onnx` model).*

### Step 4: Test the Model!
Once the installation is complete, you can test if the model works on the new machine by running the prediction script on a sample image:

```bash
python predict.py "1.jpg" --show
```
- Replace `"1.jpg"` with the path to any test image you brought over.
- The `--show` flag will pop up a window displaying the YOLOv8 bounding boxes on the fruit.
- If you use `--save` instead of `--show`, it will save the heavily annotated image into a newly created `runs/` folder.

***

### (Optional) What about the Telegram Bot & ESP32-CAM?
If you are moving the *entire* hardware pipeline to a new network:
1. **ESP32-CAM:** You will need to plug the ESP32-CAM into your computer, open Arduino IDE, flash your esp-32 cam with ESP-32_Flash_File.cpp, and update the Wi-Fi credentials (SSID and Password) to the new machine's local Wi-Fi router, and re-flash the board.

2. **Telegram Bot Server:** 
    Step 1: Create Telegram Bot
    Open Telegram
    Search "BotFather"
    Send:
    /start
    /newbot
    Give:
    Name: Rotten Fruit Detector
    Username: something_bot

   You will get:

    BOT TOKEN = 123456:ABC-XYZ...
    Step 2: Install Required Libraries
    pip install python-telegram-bot inference pillow

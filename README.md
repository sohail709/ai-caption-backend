# ai-caption-backend
from fastapi import FastAPI, UploadFile, File
import openai
import tempfile
import os

openai.api_key = "PASTE_YOUR_OPENAI_KEY"

app = FastAPI()

@app.get("/")
def home():
    return {"status": "AI Caption Server Running"}

@app.post("/generate/")
async def generate(video: UploadFile = File(...)):
    with tempfile.NamedTemporaryFile(delete=False) as temp:
        temp.write(await video.read())
        video_path = temp.name

    audio_path = video_path + ".wav"
    os.system(f"ffmpeg -i {video_path} {audio_path}")

    with open(audio_path, "rb") as audio:
        transcript = openai.Audio.transcribe(
            model="whisper-1",
            file=audio
        )

    response = openai.ChatCompletion.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "user", "content": f"Make viral captions with emojis:\n{transcript['text']}"}
        ]
    )

    return {"captions": response.choices[0].message.content}

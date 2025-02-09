import os
import openai
from dotenv import load_dotenv
from pyrogram import Client, filters
from pyrogram.types import Message
import yt_dlp

# Load environment variables
load_dotenv()
BOT_TOKEN = os.getenv("BOT_TOKEN")
API_ID = int(os.getenv("API_ID"))
API_HASH = os.getenv("API_HASH")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")

# Initialize bot
bot = Client("hinata_bot", api_id=API_ID, api_hash=API_HASH, bot_token=BOT_TOKEN)
openai.api_key = OPENAI_API_KEY

# Telugu AI Brain Function
def generate_response(user_message):
    prompt = f"హాయ్ హినాటా, మీరు ఒక తెలివైన AI. ఈ సందేశానికి తెలుగులో సమాధానం ఇవ్వండి: {user_message}"
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "system", "content": "మీరు హినాటా హ్యూగా, తెలివైన AI."},
                  {"role": "user", "content": prompt}]
    )
    return response["choices"][0]["message"]["content"]

# AI Chat Feature
@bot.on_message(filters.text & ~filters.command("play"))
def ai_chat(client, message: Message):
    reply = generate_response(message.text)
    message.reply_text(reply)

# Group Admin Features
@bot.on_message(filters.command("ban") & filters.group)
def ban_user(client, message: Message):
    if message.reply_to_message:
        client.kick_chat_member(message.chat.id, message.reply_to_message.from_user.id)
        message.reply_text("వాడిని బ్లాక్ చేశాను!")

@bot.on_message(filters.command("mute") & filters.group)
def mute_user(client, message: Message):
    if message.reply_to_message:
        client.restrict_chat_member(message.chat.id, message.reply_to_message.from_user.id, can_send_messages=False)
        message.reply_text("వాడిని మ్యూట్ చేశాను!")

@bot.on_message(filters.command("unmute") & filters.group)
def unmute_user(client, message: Message):
    if message.reply_to_message:
        client.restrict_chat_member(message.chat.id, message.reply_to_message.from_user.id, can_send_messages=True)
        message.reply_text("వాడిని అన్‌మ్యూట్ చేశాను!")

# YouTube Music Streaming
@bot.on_message(filters.command("play"))
def play_music(client, message: Message):
    if len(message.command) < 2:
        message.reply_text("దయచేసి పాట పేరు లేదా లింక్ ఇవ్వండి.")
        return
    query = " ".join(message.command[1:])
    message.reply_text(f"సాంగ్ `{query}` ను ప్లే చేస్తున్నాను...")

    # Download audio using yt-dlp
    ydl_opts = {
        "format": "bestaudio",
        "postprocessors": [{"key": "FFmpegExtractAudio", "preferredcodec": "mp3"}],
    }
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(f"ytsearch:{query}", download=True)
        filename = ydl.prepare_filename(info)
        audio_file = filename.replace(".webm", ".mp3").replace(".m4a", ".mp3")
        message.reply_audio(audio_file, caption="ఇదిగో నీ పాట 🎶")

# Start bot
print("Hinata Bot is running...")
bot.run()
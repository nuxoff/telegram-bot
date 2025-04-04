import os
import logging
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, CallbackContext
from gtts import gTTS
import openai
import tempfile

# Включаем логирование (чтобы видеть ошибки, если они будут)
logging.basicConfig(format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO)

# Получаем токены из переменных окружения (их нужно добавить в Render)
TELEGRAM_TOKEN = os.getenv("TELEGRAM_TOKEN")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")

# Если есть ключ OpenAI — подключаем
if OPENAI_API_KEY:
    openai.api_key = OPENAI_API_KEY

# Функция обработки команд
async def start(update: Update, context: CallbackContext) -> None:
    await update.message.reply_text("Привет! Я AI-бот. Напиши мне что-то!")

# Функция обработки текстовых сообщений
async def handle_message(update: Update, context: CallbackContext) -> None:
    user_message = update.message.text

    if OPENAI_API_KEY:
        # Отправляем сообщение в ChatGPT
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[{"role": "user", "content": user_message}]
        )
        reply = response["choices"][0]["message"]["content"]
    else:
        reply = f"Ты написал: {user_message}"

    await update.message.reply_text(reply)

# Функция преобразования текста в голос
async def voice_message(update: Update, context: CallbackContext) -> None:
    text = update.message.text
    tts = gTTS(text, lang="ru")

    with tempfile.NamedTemporaryFile(suffix=".mp3", delete=False) as temp_file:
        tts.save(temp_file.name)
        with open(temp_file.name, "rb") as voice:
            await update.message.reply_voice(voice)

# Главная функция
def main():
    app = Application.builder().token(TELEGRAM_TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, voice_message))

    print("Бот запущен...")
    app.run_polling()

if __name__ == "__main__":
    main()

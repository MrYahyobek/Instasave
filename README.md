import os
import yt_dlp
import datetime
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import (
    ApplicationBuilder, CommandHandler, MessageHandler,
    ContextTypes, filters, JobQueue
)

TOKEN = "7876511816:AAGuZ6kzRqhI3gdYqwRoGINxGvVyvZ7326w"
CHANNEL_USERNAME = "@Mr_Yahyobe"
ADMIN_CONTACT = "@Mr_Yahyobek"
foydalanuvchilar = set()


async def tekshir_obuna(user_id, context):
    try:
        member = await context.bot.get_chat_member(CHANNEL_USERNAME, user_id)
        return member.status in ['member', 'administrator', 'creator']
    except:
        return False


async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id

    if not await tekshir_obuna(user_id, context):
        button = InlineKeyboardMarkup([
            [InlineKeyboardButton("📢 Obuna bo'lish", url=f"https://t.me/{CHANNEL_USERNAME.strip('@')}")]
        ])
        await update.message.reply_text("❌ Botdan foydalanish uchun kanalga obuna bo‘ling.", reply_markup=button)
        return

    foydalanuvchilar.add(user_id)
    await update.message.reply_text("🎬 Instagram videosining linkini yuboring...")


async def yukla(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    text = update.message.text

    if not await tekshir_obuna(user_id, context):
        button = InlineKeyboardMarkup([
            [InlineKeyboardButton("📢 Obuna bo'lish", url=f"https://t.me/{CHANNEL_USERNAME.strip('@')}")]
        ])
        await update.message.reply_text("❌ Obuna bo‘lishingiz kerak.", reply_markup=button)
        return

    foydalanuvchilar.add(user_id)

    if "instagram.com" not in text:
        await update.message.reply_text("❗ Instagram video link yuboring.")
        return

    await update.message.reply_text("🔄 Yuklanmoqda...")

    try:
        ydl_opts = {
            'outtmpl': 'video.%(ext)s',
            'format': 'best[ext=mp4]',
            'quiet': True
        }

        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(text, download=True)
            file_path = ydl.prepare_filename(info)

        await update.message.reply_video(video=open(file_path, 'rb'))
        os.remove(file_path)

    except Exception as e:
        await update.message.reply_text("❌ Yuklab bo‘lmadi. Video maxfiy bo‘lishi mumkin.")
        print("Xatolik:", e)


async def reklama(context: ContextTypes.DEFAULT_TYPE):
    for user_id in foydalanuvchilar:
        try:
            await context.bot.send_message(chat_id=user_id, text=f"📢 Reklama uchun {ADMIN_CONTACT} ga murojaat qiling")
        except:
            pass


async def run_bot():
    app = ApplicationBuilder().token(TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, yukla))

    app.job_queue.run_daily(reklama, time=datetime.time(hour=10, minute=0))
    print("🚀 Bot ishga tushdi")
    await app.run_polling()


if name == '__main__':
    import asyncio
    asyncio.run(run_bot())

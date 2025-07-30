import json
import os
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

BOT_TOKEN = os.getenv("BOT_TOKEN")
ALLOWED_GROUP_ID = int(os.getenv("GROUP_ID"))

with open("phone_data.json", "r") as f:
    phone_data = json.load(f)["data"]

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_chat.id != ALLOWED_GROUP_ID:
        return
    await update.message.reply_text("📱 Send a mobile number to get details.")

async def search_mobile(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_chat.id != ALLOWED_GROUP_ID:
        return
    number = update.message.text.strip().replace(" ", "").replace("+91", "").lstrip("0")
    result = None
    for entry in phone_data:
        mob = entry["📱 Mobile"].replace(" ", "").replace("+91", "").lstrip("0")
        alt = entry["📱 Alt Number"].replace(" ", "").replace("+91", "").lstrip("0")
        if mob == number or alt == number:
            result = entry
            break
    if result:
        msg = f"""📱 Mobile: {result['📱 Mobile']}
👤 Name: {result['👤 Name']}
👨‍👦 Father: {result['👨‍👦 Father Name']}
🏠 Address: {result['🏠 Full Address']}
📱 Alt: {result['📱 Alt Number']}
📞 Sim/State: {result['📞 Sim/State']}
🆔 Aadhar: {result['🆔 Aadhar Card']}
📧 Email: {result['📧 Email']}"""
        await update.message.reply_text(msg)
    else:
        await update.message.reply_text("❌ Number not found.")

def main():
    app = ApplicationBuilder().token(BOT_TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, search_mobile))
    app.run_polling()

if __name__ == "__main__":
    main()

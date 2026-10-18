import asyncio
import os
import sqlite3
from aiogram import Bot, Dispatcher, F
from aiogram.filters import Command
from aiogram.types import (
    Message,
    CallbackQuery,
    ReplyKeyboardMarkup,
    KeyboardButton,
    InlineKeyboardMarkup,
    InlineKeyboardButton
)

# Читаем настройки из переменных окружения (для хостинга)
TOKEN = os.getenv("BOT_TOKEN", "8881775192:AAG2llRNMJhlozg8rkOl7HYZSQ9VVEuzOWc")
admin_ids_env = os.getenv("ADMIN_IDS", "8881775192")
ADMIN_IDS = [int(i.strip()) for i in admin_ids_env.split(",")]

bot = Bot(token=TOKEN)
dp = Dispatcher()

def init_db():
    conn = sqlite3.connect("gym_baza.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS supporters (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER UNIQUE,
            full_name TEXT,
            username TEXT UNIQUE,
            phone TEXT,
            training_type TEXT,
            rehab TEXT,
            training_time TEXT
        )
    """)
    conn.commit()
    conn.close()

init_db()

def save_user_data(user_id, **kwargs):
    conn = sqlite3.connect("gym_baza.db")
    cursor = conn.cursor()
    cursor.execute("INSERT OR IGNORE INTO supporters (user_id) VALUES (?)", (user_id,))
    
    for key, value in kwargs.items():
        cursor.execute(f"UPDATE supporters SET {key} = ? WHERE user_id = ?", (value, user_id))
        
    conn.commit()
    conn.close()

@dp.message(Command("start"))
async def cmd_start(message: Message):
    keyboard = ReplyKeyboardMarkup(
        keyboard=[
            [KeyboardButton(text="📱 Підтримати і хочу займатися!", request_contact=True)]
        ],
        resize_keyboard=True,
        one_time_keyboard=True
    )
    
    text = (
        "Простір «База» на Патріотичній — чи буде відкриття?\n\n"
        "Цей зал ми створюємо для тих, хто любить спорт, тренується для себе і живе поруч. "
        "Наша мета — працювати з вами в команді та відштовхуватися від ваших потреб, а не завозити те, що не треба.\n\n"
        "🔴 <b>Ви самі голосуєте в боті за те, яким хочете бачити зал, а ми просто робимо ваші бажання!</b>\n\n"
        "⭐ Все залежить от количества людей, которые добавятся! Если мы соберем нужное количество, проекта не будет.\n\n"
        "👉 Натисніть кнопку нижче, щоб долучитися до списку перших учасників і вплинути на те, яким буде зал!"
    )
    
    await message.answer(text, reply_markup=keyboard, parse_mode="HTML")

@dp.message(F.contact)
async def contact_handler(message: Message):
    user = message.from_user
    phone = message.contact.phone_number
    full_name = f"{user.first_name or ''} {user.last_name or ''}".strip()
    username = f"@{user.username}" if user.username else full_name
    
    try:
        save_user_data(user.id, full_name=full_name, username=username, phone=phone)
    except sqlite3.IntegrityError:
        pass
    
    keyboard = InlineKeyboardMarkup(
        inline_keyboard=[
            [InlineKeyboardButton(text="💪 СИЛОВІ ТРЕНУВАННЯ (залізо)", callback_data="train_strength")],
            [InlineKeyboardButton(text="🔥 ФІТНЕС ТА РОЗТЯЖКА", callback_data="train_fitness")],
            [InlineKeyboardButton(text="⚡ БІОМЕХАНІКА / САМООБОРОНА", callback_data="train_defense")],
            [InlineKeyboardButton(text="🚀 УСЕ РАЗОМ", callback_data="train_all")]
        ]
    )
    
    await message.answer(
        "Дякуємо! Питання 1 з 3:\n\nЯкий напрямок тренувань вас цікавить найбільше?",
        reply_markup=keyboard
    )

@dp.callback_query(F.data.startswith("train_"))
async def process_training_choice(callback: CallbackQuery):
    choice_map = {
        "train_strength": "Силові тренування (залізо)",
        "train_fitness": "Фітнес та розтяжка",
        "train_defense": "Біомеханіка / Самооборона",
        "train_all": "Усе разом"
    }
    selected_text = choice_map.get(callback.data, "Інше")
    
    save_user_data(callback.from_user.id, training_type=selected_text)
    
    keyboard = InlineKeyboardMarkup(
        inline_keyboard=[
            [InlineKeyboardButton(text="✅ Так, цікавить профілактика/реабілітація", callback_data="rehab_yes")],
            [InlineKeyboardButton(text="❌ Ні, тільки тренування", callback_data="rehab_no")]
        ]
    )
    
    await callback.message.edit_text(
        "Питання 2 з 3:\n\nЧи цікавить вас профілактика болю в спині / суглобах або фізіотерапія?",
        reply_markup=keyboard
    )
    await callback.answer()

@dp.callback_query(F.data.startswith("rehab_"))
async def process_rehab_choice(callback: CallbackQuery):
    rehab_choice = "Так" if callback.data == "rehab_yes" else "Ні"
    
    save_user_data(callback.from_user.id, rehab=rehab_choice)
    
    keyboard = InlineKeyboardMarkup(
        inline_keyboard=[
            [InlineKeyboardButton(text="🌅 Ранок (до 12:00)", callback_data="time_morning")],
            [InlineKeyboardButton(text="☀️ День (12:00 - 17:00)", callback_data="time_day")],
            [InlineKeyboardButton(text="🌙 Вечір (після 17:00)", callback_data="time_evening")],
            [InlineKeyboardButton(text="🔄 Гнучний графік", callback_data="time_any")]
        ]
    )
    
    await callback.message.edit_text(
        "Питання 3 z 3:\n\nУ який час вам було б найзручніше тренуватися?",
        reply_markup=keyboard
    )
    await callback.answer()

@dp.callback_query(F.data.startswith("time_"))
async def process_time_choice(callback: CallbackQuery):
    time_map = {
        "time_morning": "Ранок (до 12:00)",
        "time_day": "День (12:00 - 17:00)",
        "time_evening": "Вечір (після 17:00)",
        "time_any": "Гнучний графік"
    }
    selected_time = time_map.get(callback.data, "Інший час")
    
    user_id = callback.from_user.id
    save_user_data(user_id, training_time=selected_time)
    
    conn = sqlite3.connect("gym_baza.db")
    cursor = conn.cursor()
    cursor.execute("SELECT id FROM supporters WHERE user_id = ?", (user_id,))
    row = cursor.fetchone()
    
    if row:
        internal_id = row[0]
        cursor.execute("SELECT COUNT(*) FROM supporters WHERE id <= ?", (internal_id,))
        count_row = cursor.fetchone()
        user_number = count_row[0] if count_row else 1
    else:
        user_number = 1
        
    conn.close()
    
    await callback.message.edit_reply_markup(reply_markup=None)
    
    await callback.message.answer(
        f"<b>Дякуємо!</b> Ваші відповіді успішно збережено.\n\n"
        f"🎯 Ви — <b>{user_number}-й</b> учасник, який долучився до проєкту «База»!\n\n"
        "📜 Щоб переглянути загальний список учасників і статистику, натисніть команду /list\n\n"
        "Очікуйте на нові сповіщення та інформацію щодо старту занять!",
        parse_mode="HTML"
    )
    await callback.answer()

@dp.message(Command("list"))
async def cmd_list(message: Message):
    conn = sqlite3.connect("gym_baza.db")
    cursor = conn.cursor()
    cursor.execute("SELECT username, id, training_type, rehab, training_time FROM supporters ORDER BY id ASC")
    rows = cursor.fetchall()
    conn.close()
    
    if not rows:
        await message.answer("Поки що немає зареєстрованих учасників.")
        return
        
    text = "📋 <b>Список учасників проєкту «База»:</b>\n\n"
    total_users = len(rows)
    
    training_type_counts = {}
    rehab_counts = {}
    training_time_counts = {}
    
    for index, row in enumerate(rows, start=1):
        uname = row[0] if row[0] else "Анонім"
        t_type = row[2]
        rehab = row[3]
        t_time = row[4]
        
        text += f"{index}. {uname}\n"
        
        if t_type:
            training_type_counts[t_type] = training_type_counts.get(t_type, 0) + 1
        if rehab:
            rehab_counts[rehab] = rehab_counts.get(rehab, 0) + 1
        if t_time:
            training_time_counts[t_time] = training_time_counts.get(t_time, 0) + 1
        
    text += "\n📊 <b>Статистика відповідей:</b>\n\n<b>💪 Напрямок тренувань:</b>\n"
    for t_type, count in training_type_counts.items():
        percentage = (count / total_users) * 100
        text += f"• {t_type}: {percentage:.0f}%\n"
        
    text += "\n<b>🩺 Потреба в профілактиці/реабілітації:</b>\n"
    for rehab, count in rehab_counts.items():
        percentage = (count / total_users) * 100
        text += f"• {rehab}: {percentage:.0f}%\n"
        
    text += "\n<b>⏰ Зручний час:</b>\n"
    for t_time, count in training_time_counts.items():
        percentage = (count / total_users) * 100
        text += f"• {t_time}: {percentage:.0f}%\n"
        
    if len(text) > 4000:
        text = text[:4000] + "\n...\n(список занадто довгий)"
        
    await message.answer(text, parse_mode="HTML")

@dp.message(Command("stats"))
async def admin_stats_cmd(message: Message):
    if message.from_user.id not in ADMIN_IDS:
        return
        
    conn = sqlite3.connect("gym_baza.db")
    cursor = conn.cursor()
    cursor.execute("SELECT COUNT(*) FROM supporters")
    count = cursor.fetchone()[0]
    conn.close()
    
    await message.answer(f"📊 <b>Статистика проекту:</b>\nВсього зареєстровано учасників: <b>{count}</b>", parse_mode="HTML")

if __name__ == "__main__":
    asyncio.run(dp.start_polling(bot))

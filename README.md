import telebot
from telebot import types
import json
import os

TOKEN = '7609253462:AAG5m-pbuIf8xN2xqknxIj9yEFm6slccEv4'
bot = telebot.TeleBot(TOKEN)

# --- Хранилище очков ---------------------------------------------------------
SCORE_FILE = 'scores.json'

def load_scores():
    if os.path.exists(SCORE_FILE):
        try:
            with open(SCORE_FILE, 'r') as f:
                return json.load(f)
        except json.JSONDecodeError:
            return {}
    return {}

def save_scores(scores):
    with open(SCORE_FILE, 'w') as f:
        json.dump(scores, f)

def get_score(user_id):
    scores = load_scores()
    return scores.get(str(user_id), 0)

def add_score(user_id, delta):
    if delta <= 0:
        return
    scores = load_scores()
    key = str(user_id)
    scores[key] = scores.get(key, 0) + delta
    save_scores(scores)

# --- Клавиатура ---------------------------------------------------------------
mm = types.ReplyKeyboardMarkup(row_width=3, resize_keyboard=True)

button1 = types.KeyboardButton("/кубик")
button2 = types.KeyboardButton("/казино")
button3 = types.KeyboardButton("/боулинг")
button4 = types.KeyboardButton("/баскетбол")
button5 = types.KeyboardButton("/футбол")
button6 = types.KeyboardButton("/дартс")
button7 = types.KeyboardButton("/баллы")  # новая кнопка баллы

mm.add(button1, button2, button3, button4, button5, button6, button7)

# --- Обработчик стартовой команды ------------------------------------------------
@bot.message_handler(commands=['command1'])
def send_welcome(message):
    bot.reply_to(message, "Привет!Давай поиграем", reply_markup=mm)

# --- Универсальная обработка игр ---------------------------------------------------
def process_game(message, emoji):
    dice_msg = bot.send_dice(message.chat.id, emoji=emoji)
    # Попробуем получить результат броска
    if dice_msg and getattr(dice_msg, 'dice', None) is not None:
        value = dice_msg.dice.value  # 1..6
        # Простейшая система очков: 4->+1, 5-6 -> +2
        delta = 2 if value >= 5 else 1 if value >= 4 else 0

        user_id = message.from_user.id
        if delta > 0:
            add_score(user_id, delta)
            new_score = get_score(user_id)
            bot.reply_to(dice_msg, f"Вы заработали +{delta} балла(ов)")
        else:
            new_score = get_score(user_id)
            bot.reply_to(dice_msg, f"Баллы не начислены.")

# --- Обработчики команд для игр --------------------------------------------------
@bot.message_handler(commands=['кубик'])
def handle_kubik(message):
    process_game(message, '🎲')

@bot.message_handler(commands=['казино'])
def handle_kazino(message):
    process_game(message, '🎰')

@bot.message_handler(commands=['боулинг'])
def handle_bouling(message):
    process_game(message, '🎳')

@bot.message_handler(commands=['баскетбол'])
def handle_basketball(message):
    process_game(message, '🏀')

@bot.message_handler(commands=['футбол'])
def handle_football(message):
    process_game(message, '⚽')

@bot.message_handler(commands=['дартс'])
def handle_darts(message):
    process_game(message, '🎯')

# --- Баллы: показываем текущий счёт --------------------------------------------
@bot.message_handler(commands=['баллы'])
def handle_score_request(message):
    user_id = message.from_user.id
    score = get_score(user_id)
    bot.reply_to(message, f"Ваш текущий счёт: {score} баллов.")

# --- Стартовая активизация бота (можно заменить на '/start') --------------------
bot.polling()

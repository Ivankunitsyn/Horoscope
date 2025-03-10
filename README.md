import telebot
import random
import openai
import os
from datetime import datetime

class HoroscopeBot:
    def __init__(self):
        self.TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
        self.OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
        
        self.zodiac_signs = [
            "Овен", "Телец", "Близнецы", "Рак", "Лев", "Дева",
            "Весы", "Скорпион", "Стрелец", "Козерог", "Водолей", "Рыбы"
        ]
        
        if not self.TOKEN:
            print("TELEGRAM_BOT_TOKEN not found in environment variables!")
            return
            
        self.bot = telebot.TeleBot(self.TOKEN)
        
        if self.OPENAI_API_KEY:
            openai.api_key = self.OPENAI_API_KEY
        else:
            print("OPENAI_API_KEY not found in environment variables!")

        self.register_handlers()
        
    def register_handlers(self):
        @self.bot.message_handler(commands=['start'])
        def send_welcome(message):
            self.bot.send_message(message.chat.id, "Добро пожаловать! Выберите опцию:", reply_markup=self.main_menu())

        @self.bot.message_handler(commands=['help'])
        def send_help(message):
            self.bot.send_message(message.chat.id, "Напиши свой знак зодиака, и я пришлю тебе гороскоп!")
            self.send_zodiac_keyboard(message.chat.id)

        @self.bot.message_handler(func=lambda message: message.text in self.zodiac_signs)
        def send_horoscope(message):
            sign = message.text
            horoscope = self.generate_ai_horoscope(sign)
            self.bot.send_message(message.chat.id, f"Гороскоп для {sign}:\n{horoscope}")

    def main_menu(self):
        keyboard = telebot.types.ReplyKeyboardMarkup(resize_keyboard=True)
        btn1 = telebot.types.KeyboardButton("🔮 Обычный гороскоп")
        btn2 = telebot.types.KeyboardButton("🌙 Лунный календарь")
        btn3 = telebot.types.KeyboardButton("💞 Совместимость знаков")
        keyboard.add(btn1, btn2, btn3)
        return keyboard

    def generate_ai_horoscope(self, sign):
        if not self.OPENAI_API_KEY:
            return random.choice(["Сегодня удачный день!", "Будьте осторожны с деньгами.", "Отличное время для новых начинаний."])
        
        prompt = f"Создай гороскоп для знака {sign} на сегодня."
        try:
            response = openai.ChatCompletion.create(
                model="gpt-4",
                messages=[{"role": "system", "content": prompt}]
            )
            return response["choices"][0]["message"]["content"]
        except Exception as e:
            print(f"Ошибка OpenAI: {e}")
            return "Не удалось получить гороскоп, попробуйте позже."

    def run(self):
        if hasattr(self, 'bot'):
            print("Запуск Telegram бота...")
            try:
                self.bot.polling(none_stop=True)
            except Exception as e:
                print(f"Ошибка запуска бота: {e}")
        else:
            print("Бот не инициализирован.")  
            from bot import HoroscopeBot

if __name__ == "__main__":
    bot = HoroscopeBot()
    bot.run()
    pyTelegramBotAPI
openai

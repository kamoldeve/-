import telebot
import datetime

# Вставьте ваш токен
API_TOKEN = '7691991139:AAFVN2hwseN6n-AK91xTJdQsPvdrfVJLOqQ'
bot = telebot.TeleBot(API_TOKEN)

# Хранилище для информации о пользователях
user_data = {}


# Команда /start
@bot.message_handler(commands=['start'])
def send_welcome(message):
    # Инициализация данных пользователя
    if message.chat.id not in user_data:
        user_data[message.chat.id] = {'paid': False, 'subscription_end': None}

    bot.send_message(
        message.chat.id,
        "Привет! Я бот для оформления годовой подписки. Нажми /pay для активации подписки. После активации ты получишь полный доступ!"
    )


# Команда для начала оплаты
@bot.message_handler(commands=['pay'])
def process_payment(message):
    user_id = message.chat.id
    if user_id not in user_data:  # Инициализация данных, если пользователь впервые
        user_data[user_id] = {'paid': False, 'subscription_end': None}

    if user_data[user_id]['paid']:
        bot.send_message(message.chat.id, "Ты уже оплатил подписку на год! 😎")
    else:
        # Имитация успешного платежа
        user_data[user_id]['paid'] = True
        user_data[user_id]['subscription_end'] = datetime.datetime.now() + datetime.timedelta(days=365)
        bot.send_message(message.chat.id, "Спасибо за оплату! Твоя подписка активирована на 1 год! 🎉")



@bot.message_handler(commands=['status'])
def check_status(message):
    user_id = message.chat.id
    if user_id not in user_data:
        user_data[user_id] = {'paid': False, 'subscription_end': None}

    if user_data[user_id]['paid']:
        subscription_end = user_data[user_id]['subscription_end']
        if subscription_end > datetime.datetime.now():
            time_left = subscription_end - datetime.datetime.now()
            bot.send_message(message.chat.id,
                             f"Ваша подписка активна. До окончания осталось: {str(time_left).split('.')[0]}")
        else:
            bot.send_message(message.chat.id, "Ваша подписка истекла. Пожалуйста, оплатите снова.")
    else:
        bot.send_message(message.chat.id,
                         "Ваша подписка не активирована. Пожалуйста, нажмите /pay для активации подписки.")


@bot.message_handler(commands=['help'])
def help_message(message):
    bot.send_message(
        message.chat.id,
        "/pay - оплатить подписку.\n"
        "/status - проверка текущего статуса подписки.\n"
        "Ваша подписка даст доступ ко всем премиум-функциям на год!"
    )


@bot.message_handler(commands=['reset'])
def reset_subscription(message):
    user_id = message.chat.id
    if user_id not in user_data:
        user_data[user_id] = {'paid': False, 'subscription_end': None}

    user_data[user_id] = {'paid': False, 'subscription_end': None}
    bot.send_message(message.chat.id, "Подписка была сброшена. Пожалуйста, снова нажмите /pay для активации.")


bot.polling()


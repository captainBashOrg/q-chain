
from aiogram import Bot, Dispatcher, types
from aiogram.contrib.fsm_storage.memory import MemoryStorage
#import asyncio
from aiogram.dispatcher.filters.state import State, StatesGroup
#from aiogram.dispatcher import FSMContext
from aiogram.utils import executor

api = "ХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХХ"
bot = Bot(token=api)
dp = Dispatcher(bot, storage=MemoryStorage())


@dp.message_handler(commands=['start'])
async def start_message(message):
    await message.answer('Привет! Я бот помогающий Вашему здоровью.')
    await message.answer('Для начала расчета введите команду "/Calories"')

class UserState(StatesGroup):
    age = State()
    growth = State()
    weight =  State()
    sex_ =  State()



@dp.message_handler(commands=['Calories'])
async def set_age(message):
    await message.answer('Введите Ваш возраст:')
    await UserState.age.set()


@dp.message_handler(state=UserState.age)
async def set_growth(message, state):
    if not message.text.isdigit(): # Возраст не цифирь
        await message.answer('Пожалуйста, вводите Ваш возраст числом')
        await state.finish()
        return
    await state.update_data(age=message.text)
    await message.answer('Введите Ваш рост (см):')
    await UserState.growth.set()



@dp.message_handler(state=UserState.growth)
async def set_weight(message, state):

    if not message.text.isdigit(): # рост не цифирь
        await message.answer('Пожалуйста, вводите Ваш рост числом')
        await state.finish()
        return

    await state.update_data(growth=message.text)
    await message.answer('Введите Ваш вес (кг):')
    await UserState.weight.set()


@dp.message_handler(state=UserState.weight)
async def set_sex_(message, state):
    if not message.text.isdigit(): # вес не цифирь
        await message.answer('Пожалуйста, вводите Ваш вес числом')
        await state.finish()
        return

    await state.update_data(weight=message.text)
    await message.answer('Сообщите Ваш пол (М\Ж): ')
    await UserState.sex_.set()



@dp.message_handler(state=UserState.sex_)
async def send_calories(message, state):
    await state.update_data(sex_=message.text)
    data = await state.get_data()

    try:
        age = float(data['age'])
        weight = float(data['weight'])
        growth = float(data['growth'])
    except:
        await message.answer(f'Пожалуйста, вводите рост, вес и возраст цифрами.')
        #await message.answer(data )
        await state.finish()
        return

    # Упрощенный вариант формулы Миффлина-Сан Жеора:
    # для мужчин: 10 х вес (кг) + 6,25 x рост (см) – 5 х возраст (г) + 5
    calories_man = 10 * weight + 6.25 * growth - 5 * age + 5
    #для женщин: 10 x вес (кг) + 6,25 x рост (см) – 5 x возраст (г) – 161
    calories_wom = 10 * weight + 6.25 * growth - 5 * age - 161

    sex_ =  str(data['sex_'])
    if sex_.upper() in ('M', 'М'):
        await message.answer(f'Норма (муж.): {calories_man} ккал')
    elif sex_.upper() in ('Ж', 'W', 'F'):
        await message.answer(f'Норма (жен.): {calories_wom} ккал')
    else:
        await message.answer('Пожалуйста, вводите пол только М или Ж.')

    await message.answer('До новых встреч!')
    await message.answer('Для начала расчета введите команду "/Calories"')
    await state.finish()




#обработка нетипичноего сообщения
@dp.message_handler()
async def all_message(message):
    #print(f"Иное сообщение, {message}")
    #print('Введите команду /start, чтобы начать общение.')
    #await message.answer ('Введите команду /start, чтобы начать общение.')
    #print(f"Иное сообщение, {message['chat']['first_name']} {message['chat']['last_name']}" )
    await message.answer(f"Здравствуйте, {message['chat']['first_name']} {message['chat']['last_name']}!")
    await message.answer('Для начала расчета введите команду "/Calories"')


if __name__ == '__main__':
    executor.start_polling(dp, skip_updates=True)

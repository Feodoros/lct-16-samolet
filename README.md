# lct-16-samolet
Сабмит на хакатон ЛЦТ 2024, задача 16 от ГК Самолет

# О нас
![изображение](https://github.com/Feodoros/lct-16-samolet/assets/23313519/951d9b16-4025-45de-8d57-8cecf93aa18d)


Мы — мультифункциональная команда компьютерных инженеров, дизайнеров и стратегов, которые разделяют веру в технологии как мощный инструмент преобразования бизнеса и общества, в котором мы живем. 

Мы занимаемся разработкой SaaS-продукта для онлайн-коммуникаций, который поменяет рабочие процессы и станет ежедневным инструментом профессионалов в этой области. Наша ближайшая цель — взять на себя все рутинные задачи, связанные с онлайн-встречами: запись встреч, расшифровка, построение саммари, выделение итогов и задач с дедлайнами и ответственными.

## Преимущества сервиса [mymeet.ai](https://mymeet.ai/ru)
- Скорость обработки часового звонка -- 3-4 минуты
- Точность транскрибации на русском языке -- 97%
- Строим саммари, выделяем договоренности, составляем задачи с дедлайнами и ответственными
- Автозапись Google Meet, Zoom, Яндекс.Телемост, SberJazz, Trueconf
- Интеграция с Telegram, Google Calendar, amoCRM
- Полноценное B2B on-premise решение на наших собственных локальных моделях

# Решение задачи от ГК Самолет
## Шаг 1. Подсчет вхождений слова скидка.
T = 'да лисное по призрак лишнее ну почему иду пять двух полностью последнего да да да и там есть данные надо чтобы стандартная двух что была пятьдесят пятьдесят десять миллионов три а первоначальный вздох сколько вас а еще какие то фиджи приедем там сейчас менеджера да вам я такая там проценты а если не столько да хорошо конечно и это заработал хорошо а инвестое отправьте пожалуйстаздравствуйте меня зовут на менеджере дело продаж группы самолет вы рассматриваете какой то определенный комплекс для себя что хотели просмотреть сколько комнат двух комнатная квартира так давайте посмотрим я вижу вы интересовались ранее да этим знаете где он находится о там есть да уже готовы какая площадь вас интересует секунду так ну вот готовая двухкомнатная пятьдесят квадратных метров с отделкой десять миллионов триста да минимум пятнадцать процентов миллион шестьсот продаж дополнительно могу вам отправить скидку два процента она действует в течение двух дней сегодня и завтра удобно сегодня к нам подъехать так ну там договор ээ уже введенный дом поэтому там нужно будет в офисе уточнять а так на этапе строительства там от трех девяти и до четырех восьми но здесь нужно смотреть потому что я тоже готовый дом но там тоже семейная ипотека есть если не семейная то от пяти девяти до шести восьми процентов да вам скидку направлять сегодня ждем вас до девяти офис работает просто этот телефон назовете администратор увидит что вы записаны на встречу да ждем вас тогда угу да конечно намегасы'

На вход программе у нас поступает текст T. Нам его необходимо обработать. В первом шаге в этом тексте мы еще все упоминания слова “скидка” и его аналогов. Для каждого вхождения берем локальный контекст справа и слева по 25 слов и берем их для дальнейшего анализа. Также запоминаем доп инфу, по типу начала локального контекста и его конец. Эта информация нам пригодиться позже.

пример обработки:

temp_ans1 = shag1(T)

temp_ans1 = [{'context': 'секунду так ну вот готовая двухкомнатная пятьдесят квадратных метров с отделкой десять миллионов триста да минимум пятнадцать процентов миллион шестьсот продаж дополнительно могу вам отправить скидку два процента она действует в течение двух дней сегодня и завтра удобно сегодня к нам подъехать так ну там договор ээ уже введенный дом поэтому', 'start': 113, 'end': 164}, {'context': 'потому что я тоже готовый дом но там тоже семейная ипотека есть если не семейная то от пяти девяти до шести восьми процентов да вам скидку направлять сегодня ждем вас до девяти офис работает просто этот телефон назовете администратор увидит что вы записаны на встречу да ждем вас тогда угу да', 'start': 187, 'end': 238}]

## Шаг 2. Классификация текстов.
В temp_ans1 у нас хранятся все схождения локальные контексты где упоминалась скидка. Далее нам нужно ответить на вопрос: В каждом локальном контексте скидка предлагалась или просто упоминалась или предлагалась без явного контекста и так далее? Если скидка упоминалась, то тогда будем дальше анализировать локальный контекст, если нет, то не  будем его анализировать.

Для этих целей была натренированна маленькая модель классификации, основанная на [cointegrated/rubert-tiny](https://huggingface.co/cointegrated/rubert-tiny). Ее цель, говорить 1, если скидка именно предлагалась. И 0, если не предлагалась. 

## Шаг 3. Кандидаты в B_discounts.
Если в чанке (локальном контексте) скидка предлагалась (те модель ответила 1). То мы вспоминаем в каких местах мы встречали слово скидка и его аналоги и записываем их к нашим кандидатам на B_discounts

```
if answer_model == 1:
  B_discounts_candidates = find_discount_positions(chunk[‘context’], start_pos =chunk[‘start’])

B_discounts_candidates = [138] – токен где упоминается скидка
```

Тонкий момент: Берем каждое упоминание скидка в качества кандадата, потому что чуть позже будем чистить не нужные упоминания ее. В функцию find_discount_positions передаем еще start_pos начало локального контекста, чтобы восстановить исходный токен слова.

## Шаг 4. Поиск размерности.
Найдя все позиции слова скидка и аналогов, можно переходить к поиску следующих токенов. Для это в локальной области сначала ищется максимально близкая размерность скидки, относительно текущего кандидата. 

Это значит что мы сначала в чанке ищем упоминание размерности скидки, те слова потипу процент, рублей, евро и так далее. А затем выбираем из них тот токен, который ближе находиться в к нашему кандидату

```
for b_candidate in B_discounts_candidates:
	token_razmernost = serach_razmernost(b_candidate, chunk[‘context’])

token_razmernost = [140] – сотвествует слову процент
```

Тонкий момент: поиск идет путем простого поиска слов без мл. Если таких токенов найдено не будет, то закачиваем анализ и просто в ответе выдаем B_discounts_candidates

## Шаг 5. Поиск числового значения.
После поиска размерности, если мы ее нашли, то начинаем поиск цифр свзяанных с этой размерностью. Для того, чтобы это сделать мы строим морфологическое дерево по нашему локальноному контексту, с помощью библиотеки [natasha](https://github.com/natasha/natasha). C помощью нее мы определяем ноду для размерности скидки и смотрим на всех детей этой ноды и ищем там числа. Все числа (если они есть) будем запоминать. 
numbers = serach_numbers(token, chunk[‘context’])

numbers = [139] – сотвествует слову три

Тонкий момент: Если цифр не нашли, то заканчиваем алогритм и отдаем только позицию слова сидки и ее размерность. 
Если в тексте встречается два одинаковый слова числа, то береться ближайшее к токену размерности.
Бывает такое, что числа могут быть рандомно раскиданы по тексту и все они являются предками размерности скидки. Тогда из всего этого многообразия береться ближайшая группа чисел. Группа чисел – объединение чисел, если их расстояния в тексте не больше чем 2. Это делаетс чтобы лоавить комбинации числе, например пятдесят два.

## Шаг 6. Формирование ответа.
Как только мы обработали слово сккидка, ее размерность и число, то считаем что обработали таким образом один чанк. 

В качестве b_discount выдаем все токены скидки, которые мы запонили в чанке (если их было несколько и они ссылались на одну и туже размерность, то оставляем только одно из этих значений, то которое ближе к токену размерности). 

В качестве b_value беру самый маленький токен из numbers и token_razmernost. 

В качестве i_value беруз все оставльные токены из numbers и token_razmernost

Далее подобном образом, обрабатываем все чанки, объединяем, убираем пересечения, соритируем и выдаем ответ.

# Преимущества решения.
- Мы использовали только открытые open-source модели
- Мы натренировали на синтетической разметке маленькую быструю BERT-модель для определения наличия скидки по локальному контексту 
- Метрика F1 -- **0.81**
- Скорость работы всего решения на ЦПУ (Intel(R) Core(TM) i7-10870H CPU @ 2.20GHz) -- **10 телефонных разговоров в секунду**

# Пример работы решения.
**Входной текст:**
```
добрый день NAME меня зовут игнать артема агенты недвижим по ней города хотим завтра с клиентом к вам приехать с утречком в городе парк так с секунда секундан так дикту девятьсот двадцать пять ноль тридцать четыре пятьдесят пять семь два да селен да и еще такой такси а дом один да совершенно связь ну вот как раз здесь если по идее мы должны успеть во сколько там ждать если что тяжело будет наверно получше ждать ну внеси вести есть место ел ввести да ну такси добрый день меня зовут NAME конечно назовите пожалуйста номер клиента NAME северно горький пар такси могу сейчас вызвать а можно будет завтра позвонить тоже по горячей линии мало ли вы разберетесь до метро именно клиент доберется до метро а оттуда хочется на такси доехать вас правильно понимаю все смотрите так я вам назначу встречу завтра а во сколько будет удобно смотрите у нас по времени там определенное время поэтому если придете раньше позже придется подождать в живую очереди но недолго готового времени я не знаю какого как менеджер освободиться в ближайшее то есть ну максимум минут минус финан в десяти утра да записала подскажите поедете наличные автомобиле на общем метро выше я вышел вам адрес клиенту точнее на номер телефона вот а тогда как подъездить к метро набрать на горячую линию и скажите то что вы назначены у вас назначена встреча вот и так смотрите так как вы записались сегодня подойдете завтра за ближайшее посещение офису у вас лично от меня будет скидка у клиента точнее скидка два процента как за
```
**Результат:**
```
{'B-discount': [251], 'I-value': [253], 'B-value': [252]}

'B-discount' — скидка 
'B-value' — два 
'I-value' — процента
```
**Время на ЦПУ:**
0.09 сек

# Запуск решения
1) Прототип доступен по ссылке: https://colab.research.google.com/drive/1ugUQO-WWCQlMISFd0swq7cF6-UB8XSEA#scrollTo=p2Jv3168BEjF
2) Ничего скачивать дополнительно не надо. Достаточно последовательно запустить все клетки.
3) Генерация ответа происходит в последней [клетке](https://colab.research.google.com/drive/1ugUQO-WWCQlMISFd0swq7cF6-UB8XSEA#scrollTo=AVYfsCUrQ03d) секции [Пример генерации отчета](https://colab.research.google.com/drive/1ugUQO-WWCQlMISFd0swq7cF6-UB8XSEA#scrollTo=GuCwwsRh7DRk) ![изображение](https://github.com/Feodoros/lct-16-samolet/assets/23313519/498d68c9-f319-4011-a77c-803051780e08)

# Дополнительное решение на MyMeetLLM
Также мы дообучили нашу собственную LLM модель на решение данной задачи. Она работает локально на ГПУ (от A40) и выдает результат в виде конкретного ответа (Да\Нет) и значения скидки. Проверить второе решение можно в самой последней [клетке](https://colab.research.google.com/drive/1ugUQO-WWCQlMISFd0swq7cF6-UB8XSEA#scrollTo=OcrEfWyxmbcH&line=1&uniqifier=1).
![изображение](https://github.com/Feodoros/lct-16-samolet/assets/23313519/280b0698-f006-477f-8d92-712d70342e2a)

**Время на ГПУ:**
0.05 сек

**Метрика F1:** 
0.96 на train датасете. 

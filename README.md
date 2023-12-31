На Python + FastAPI реализовать REST эндпоинты для работы со списком конференций:

GET /conferences - получение списка конференций;
GET /conferences/{conference_id}- получение полной информации о конференции;
POST /conferences - добавление новой заявки на участие в конференции;
PUT /conferences/{conference_id} - редактирование ранее созданной конференции.
В качестве базы данных (хранилища) перечня конференций используется таблица Google.Sheets, с которой необходимо работать через Google Sheets API. Структура таблицы (столбцы):

-идентификатор (число) конференции (можно опустить, если он совпадает с номером строки в таблице);
-идентификатор (строка) отдельной гугл-таблицы, в которой будет храниться информация об участниках конференции (см. Бэкенд. Работа с заявками #1 );
-полное название конференции на русском (напр., "76-я международная студенческая научная конференция ГУАП");
-короткое название конференции на русском (напр., "МСНК 2023");
-полное название конференции на английском (не обязательное поле);
-короткое название конференции на английском (не обязательное поле);
-название организации, проводящей конференцию;
-дата начала приема заявок на участие в конференции;
-дата окончания приема заявок на участие в конференции;
-дата начала приема статей/докладов для публикации;
-дата окончания приема статей/докладов для публикации;
-дата начала проведения конференции;
-дата завершения конференции;
-URL сайта конференции (не обязательное поле);
-электронная почта для связи с организаторами.

1.
GET /conferences
Входные данные (параметры запроса)
Параметр filter, используется для фильтрации (поиска) конференций в гугл-таблице. Может принимать следующие значения:

all - в выдачу должны включаться все имеющиеся в гугл-таблице конференции;
active - в выдачу должны включаться все конференции, по которым в данный момент идет прием заявок на участие (значение по умолчанию);
past - в выдачу должны включаться все конференции, срок приема заявок по которым уже прошел;
future - в выдачу должны включаться все конференции, срок начала приема заявок по которым еще не наступил.
Выходные данные
JSON-список, содержащий короткую информацию о найденных конференциях. Если найденных записей нет, возвращается пустой список [].
Формат возвращаемого json:

[
  {
    "id": 1,
    "name_rus_short": "Короткое название конференции на русском",
    "name_eng_short": "Короткое название конференции на английском",
    "conf_start_date": "Дата начала проведения конференции",
    "conf_end_date": "Дата завершения конференции"
  }
]
Примеры обращения к эндпоинту
/conferences
/conferences?filter="past"
/conferences/?filter="future"

2.
GET /conferences/{conference_id}
Входные данные (URL)
Параметр {conference_id} (число) - идентификатор конференции.

Логика работы
В гугл-таблице осуществляется поиск конференции с указанным идентификатором. Если найдена, возвращается вся информация о ней. Если не найдена, возвращается ошибка 404.

Выходные данные
JSON, содержащий полную информацию о найденной конференции, включая ее id. Пример:

{
  "id": 2, // conference_id
  "name_rus": "76-я международная студенческая научная конференция ГУАП",
  "name_rus_short": "МСНК-2023",
  "name_eng": "",
  "name_eng_short": "",
  "registration_start_date": "20.02.2023", // дата начала приема заявок
  "registration_end_date": "14.04.2023", // дата окончания приема заявок
  "submission_start_date": "01.05.2023", // дата начала приема докладов для публикации
  "submission_end_date": "30.05.2023", // дата окончания приема докладов
  "conf_start_date": "17.04.2023", // дата начала конференции
  "conf_end_date": "21.04.2023", // дата окончания конференции
  "organized_by": "ГУАП", // название организации, проводящей мероприятие
  "url": "https://guap.ru/msnk/",
  "email": "example@example.com"
}
Если в запросе имеется заголовок "Authorization: Bearer {token}" и указанный в нем токен проходит проверку, в выходной JSON включается дополнительное поле "google_spreadsheet", содержащее строковый идентификатор гугл-таблицы с данными об участниках конференции. Например, : "google_spreadsheet": "15yVKYIUea6gj2wTWR1ndPzeGENDyBIxo1eHZDkRljUI". Если токен не проходит проверку, возвращается ошибка 403.

3.
POST /conferences
Входные данные (тело запроса)
JSON, аналогичный тому, что приведен в описании выходных данных эндпоинта GET /conferences/{conference_id}, без поля "id", но с полем "google_spreadsheet".

Логика работы
Проверяется наличие заголовка "Authorization: Bearer {token}" в запросе. Если токен валидный, запрос обрабатывается и в таблицу со списком конференций добавляется новая запись. Если токен не валидный или отсутствует заголовок "Authorization" - возвращается ошибка 403.

Выходные данные
В случае успешной записи в таблицу эндпоинт возвращает JSON, аналогичный поданному на вход, но с дополнительным полем "id", в котором содержится порядковый номер добавленной в таблицу строки (т.н. {conference_id}).

4.
PUT /conferences/{conference_id}
Входные данные (тело запроса)
JSON, аналогичный тому, что приведен в описании выходных данных эндпоинта GET /conferences/{conference_id}, без поля "id". Все поля являются необязательными.

Логика работы
Проверяется наличие заголовка "Authorization: Bearer {token}" в запросе. Если токен валидный, запрос обрабатывается и в таблице со списком конференций вносятся изменения в запись с указанным {conference_id}. Если токен не валидный или отсутствует заголовок "Authorization" - возвращается ошибка 403. Если запись с указанным id конференции в таблице отсутствует, возвращается ошибка 404. Если запись найдена и авторизация пройдена, осуществляется замена значений в таблице на значения в теле запроса. Если какие-то поля с описанием информации о конференции в теле запроса отсутствуют, они в таблице не меняются.

Выходные данные
В случае отсутствия ошибки: JSON, аналогичный тому, что приведен в описании выходных данных эндпоинта GET /conferences/{conference_id}, включая поля "id", "google_spreadsheet" и все остальные, в т.ч. и те, которые не были изменены данным запросом.
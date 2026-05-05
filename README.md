# JCarTools Web API

Документация API для HTML-экранов JCarTools.

- HTML-версия документации: [`index.html`](./index.html)
- Разделы: [чёрный экран](#чёрный-экран), [приборная панель](#приборная-панель)

## Чёрный экран

Чёрный экран работает внутри Android WebView. Со стороны JS доступен объект:

```js
window.androidApi
```

Почти все методы принимают первым аргументом токен:

```js
const TOKEN = "SECURE_TOKEN_2025";
```

После загрузки страницы нужно сообщить Android, что JS готов принимать события:

```js
window.androidApi.onJsReady(TOKEN);
```

## JS -> Android

| Метод | Возврат | Назначение |
| --- | --- | --- |
| `onJsReady(token)` | `void` | Сообщить Android, что страница готова принимать события. |
| `onClose(token)` | `void` | Закрыть чёрный экран. |
| `onSettings(token)` | `void` | Открыть настройки текущего веб-экрана. |
| `runEnum(token, enumName)` | JSON string | Запустить встроенную команду по имени. |
| `getRunEnum(token)` | JSON string | Получить список доступных команд `runEnum`. |
| `getRunEnumPic(token, enumName)` | string | Получить иконку команды в base64. |
| `runApp(token, packageName)` | JSON string | Запустить приложение по package name. |
| `getUserApps(token)` | JSON string | Получить список пользовательских приложений. |
| `getAppInfo(token, packageName)` | JSON string | Получить название и иконку приложения. |
| `setvol(token, value)` | `void` | Установить громкость. |
| `getvol(token)` | number | Получить текущую media-громкость в процентах. |
| `getCarData(token, name)` | JSON string | Получить данные автомобиля по имени запроса. |
| `getFileList(token)` | JSON string | Получить список файлов из Downloads. |
| `getFile(token, name)` | string | Получить файл из Downloads в base64. |
| `setLogFilter(token, q1, q2, q3)` | `void` | Задать до трёх фильтров для чтения строк из автомобильных логов. |
| `pingEvent(token)` | `void` | Проверить обратную отправку события в JS. |
| `setBright(token, value)` | `void` | Передать значение яркости/прозрачности `0..100`. |
| `runJson(token, json)` | JSON string | Резервный метод для сложных JSON-команд. |

## Получение данных

### `getvol(token)`

Возвращает число `0..100`. При ошибке возвращает `-1`.

```js
const volume = window.androidApi.getvol(TOKEN);
```

Пример ответа:

```json
85
```

### `getCarData(token, "frontSeats")`

Возвращает JSON-строку с состоянием передних сидений.

Доступные имена запроса:

- `frontSeats`
- `front_seats`

```js
const seats = JSON.parse(
  window.androidApi.getCarData(TOKEN, "frontSeats")
);
```

Пример ответа:

```json
{
  "name": "frontSeats",
  "frontLeft": {
    "seat": 1,
    "heat": 2,
    "vent": 0
  },
  "frontRight": {
    "seat": 2,
    "heat": 0,
    "vent": 1
  }
}
```

Где:

| Поле | Значение |
| --- | --- |
| `seat: 1` | левое / водительское переднее сиденье |
| `seat: 2` | правое / пассажирское переднее сиденье |
| `heat` | текущий уровень подогрева |
| `vent` | текущий уровень вентиляции |

### `getUserApps(token)`

Возвращает JSON-массив пользовательских приложений.

```js
const apps = JSON.parse(window.androidApi.getUserApps(TOKEN));
```

Пример ответа:

```json
[
  {
    "name": "Навигатор",
    "package": "ru.yandex.yandexnavi",
    "icon": "iVBORw0KGgo..."
  },
  {
    "name": "Музыка",
    "package": "com.example.music",
    "icon": "iVBORw0KGgo..."
  }
]
```

### `getAppInfo(token, packageName)`

Возвращает JSON-строку с названием и иконкой приложения.

```js
const appInfo = JSON.parse(
  window.androidApi.getAppInfo(TOKEN, "ru.yandex.yandexnavi")
);
```

Пример ответа:

```json
{
  "name": "Навигатор",
  "icon": "iVBORw0KGgo..."
}
```

### `getRunEnum(token)`

Возвращает JSON-массив доступных команд.

```js
const commands = JSON.parse(window.androidApi.getRunEnum(TOKEN));
```

Пример ответа:

```json
[
  {
    "RunEnum": "MEDIA_PLAY",
    "RunEnumText": "Медиа плей"
  },
  {
    "RunEnum": "MEDIA_PAUSE",
    "RunEnumText": "Медиа пауза"
  }
]
```

### `getRunEnumPic(token, enumName)`

Возвращает base64-строку иконки команды.

```js
const iconBase64 = window.androidApi.getRunEnumPic(TOKEN, "MEDIA_PLAY");
image.src = "data:image/png;base64," + iconBase64;
```

Пример ответа:

```text
iVBORw0KGgoAAAANSUhEUg...
```

### `getFileList(token)`

Возвращает JSON-массив файлов из Downloads.

```js
const files = JSON.parse(window.androidApi.getFileList(TOKEN));
```

Пример ответа:

```json
[
  {
    "name": "theme.zip",
    "size": 88340,
    "date": 1770000000000
  }
]
```

### `getFile(token, name)`

Возвращает base64-содержимое файла. Если файл не найден или токен неверный, возвращается пустая строка.

```js
const fileBase64 = window.androidApi.getFile(TOKEN, "theme.zip");
```

Пример ответа:

```text
UEsDBBQAAAAIA...
```

### Ошибки

Методы, которые возвращают JSON-строку, обычно возвращают объект с полем `error`.

```json
{
  "error": "unauthorized"
}
```

Возможные ошибки для `getCarData`:

| Ошибка | Значение |
| --- | --- |
| `unauthorized` | неверный токен |
| `empty_name` | не передано имя запроса |
| `unknown_data` | неизвестное имя запроса |
| `car_cmd_proxy_null` | данные автомобиля недоступны |

## Android -> JS

Android отправляет события в функцию:

```js
window.onAndroidEvent = function(type, data) {
  if (typeof data === "string") {
    data = JSON.parse(data);
  }
};
```

| Событие | Поля `data` | Назначение |
| --- | --- | --- |
| `musicInfo` | `PlayStat`, `SongName`, `SongArtist`, `SongAlbum`, `SongAlbumPicture`, `Trpos`, `Trdur` | Данные текущего трека. |
| `gps` | `lat`, `lon`, `speed`, `heading` | Координаты, GPS-скорость и курс. |
| `speed` | `value` | Скорость автомобиля. |
| `rpm` | `value` | Обороты двигателя. |
| `hud` | `hudSenderType`, `aradarOn`, `naviOn`, `turnType`, `turnDist`, `remainDist`, `nextRoad`, `speedLimit` | Навигационные и антирадарные данные. |
| `weather` | `temp`, `icon` | Погода. |
| `theme` | `mode` | Тема: `dark` или `light`. |
| `LogData` | `value` | Строка из автомобильного лога, найденная по фильтрам `setLogFilter(...)`. |
| `GPSSignalQuality` | `updateSignalQuality` | Качество GPS-сигнала. |
| `ping` | `ok` | Ответ на `pingEvent(token)`. |

### Получение данных из логов авто

Сначала задаются фильтры:

```js
window.androidApi.setLogFilter(
  TOKEN,
  "VehicleSpeed",
  "SeatHeat",
  "HVAC"
);
```

Затем подходящие строки приходят событием `LogData`:

```js
window.onAndroidEvent = function(type, data) {
  if (typeof data === "string") {
    data = JSON.parse(data);
  }

  if (type === "LogData") {
    console.log(data.value);
  }
};
```

Пример события:

```json
{
  "value": "[HVAC] SeatHeat left level=2"
}
```

## Команды `runEnum`

Команда запускается по имени:

```js
window.androidApi.runEnum(TOKEN, "MEDIA_PLAY");
```

| Группа | Имена | Описание |
| --- | --- | --- |
| Навигация интерфейса | `GO_TO_PP`, `GO_TO_GU`, `TOGGLE_GU_PP`, `TOGGLE_GU_PP_CPP`, `GO_CPP_TO_PP`, `TOGGLE_CPP_PP` | Переключение между головным устройством, приборкой, картами и штатными экранами. |
| Системные действия | `RUN_BLACK`, `OPEN_SHTORKA`, `CLOSE_SHTORKA`, `GLOBAL_BACK`, `GLOBAL_HOME`, `RUN_START_APP_MENU`, `RUN_FUN_CAR`, `SPLIT_RUN` | Чёрный экран, шторка, глобальные кнопки, меню приложений, быстрый доступ к функциям авто и разделение экрана. |
| Медиа | `MEDIA_PLAY`, `MEDIA_PAUSE`, `MEDIA_NEXT`, `MEDIA_BLACK` | Плей, пауза, следующий и предыдущий трек. |
| Передние сиденья | `heat_seat_l_0..3`, `heat_seat_r_0..3`, `vent_seat_l_0..3`, `vent_seat_r_0..3` | Подогрев и вентиляция водительского и пассажирского передних сидений. |
| Обогревы | `heat_wheel_on`, `heat_wheel_off`, `heat_windshield_on`, `heat_windshield_off`, `heat_rearwindow_on`, `heat_rearwindow_off` | Руль, лобовое стекло и заднее стекло. |
| Задние сиденья | `heat_zad_seat_l_0..3`, `heat_zad_seat_r_off`, `heat_zad_seat_r_1..3` | Подогрев заднего левого и заднего правого сиденья. |
| Профили и прочее | `VIBOR_VODITEL`, `voditel_seat_1`, `voditel_seat_2`, `voditel_seat_3`, `VIEW_ALL_MESSAGE`, `b_fiksik_on`, `b_fiksik_off`, `RUN_APP_MORE`, `RUN_SPICH_FOCUS`, `Recirculation_On`, `Recirculation_Off` | Выбор водителя, сообщения, голосовой помощник, группа задач и рециркуляция. |

## Примеры

### Запуск приложения из сохранённого слота

```js
const app = JSON.parse(localStorage.getItem("app_slot_1"));

if (app) {
  window.androidApi.runApp(TOKEN, app.package);
}
```

### Обновление плеера по событию

```js
window.onAndroidEvent = function(type, data) {
  if (typeof data === "string") {
    data = JSON.parse(data);
  }

  if (type === "musicInfo") {
    document.querySelector(".title").textContent = data.SongName || "";
    document.querySelector(".artist").textContent = data.SongArtist || "";
  }
};
```

## Приборная панель

Раздел для API экранов приборной панели. Здесь будут описаны события скорости, RPM, HUD и специфичные методы приборной панели.

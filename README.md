<!DOCTYPE html>
<html lang="ru">
<body>

<h1>lpac 2.3.0 Backport for OpenWrt & eSIM Management Toolchain</h1>

<p>
    Этот репозиторий содержит бэкпорт современного стека <b>lpac 2.3.0</b> и сопутствующих утилит (libqmi, qmi-utils) для архитектуры <b>OpenWrt aarch64 (Cortex-A53)</b>, 
    а также интеллектуальный управляющий скрипт для работы с картами <b>eUICC (9esim)</b> в связке с 5G/4G модемами.
</p>

<hr>

<h2>Что проделано</h2>
<ul>
    <li><b>Бэкпорт lpac 2.3.0:</b> Версия адаптирована для работы под musl libc в среде OpenWrt 21.x – 25.x.</li>
    <li><b>Обновление стека QMI:</b> Собраны <code>libqmi 1.36.0</code> и <code>qmi-utils</code> для полноценной поддержки расширенных команд сброса.</li>
    <li><b>QMI/AT Wrapper:</b> Создан бинарный враппер <code>/usr/bin/lpac</code> для прозрачного управления переменными окружения.</li>
    <li><b>lpac-esim-qmi.sh:</b> Разработан интерактивный BusyBox-совместимый скрипт (v2.6.1) с логикой «умного» сброса модема.</li>
</ul>

<h2>Тестовое окружение</h2>
<ul>
    <li><b>Роутер:</b> RAX3000M (OpenWrt 23/24).</li>
    <li><b>Модем:</b> <b>Foxconn T99W175 (Snapdragon X55)</b> в режиме QMI.</li>
    <li><b>eUICC:</b> Съемная карта <b>9esim (ST-чип)</b>.</li>
</ul>

<hr>

<h2>Возможности скрипта lpac-esim-qmi.sh</h2>
<p>Скрипт решает главную проблему модемов Qualcomm X55 — <i>жесткое кэширование IMSI</i>, из-за которого профиль на карте меняется, а модем продолжает «видеть» старого оператора.</p>

<h3>Основные функции:</h3>
<ul>
    <li><b>Интеллектуальный Switch:</b> Автоматическое включение нового профиля с последующей деактивацией старого и принудительным ребутом модема.</li>
    <li><b>Live Modem Monitor:</b> Экран реального времени для наблюдения за процессом регистрации в сети (disabled -> enabling -> connecting -> connected).</li>
    <li><b>Автоматизация APN:</b> Полная совместимость со скриптами hotplug, которые пересоздают интерфейсы при горячем переподключении.</li>
</ul>

<h3>Структура меню:</h3>
<pre>
1) Profile Management
   ├── List profiles (Список с маскировкой ICCID)
   ├── Switch active profile (Смена профиля + Hard Reboot)
   ├── Enable/Disable profile (Ручное управление)
   └── Back / Exit

2) Status & Diagnostics
   ├── eUICC Chip Info (EID, версия прошивки чипа)
   ├── Live Modem Monitor (Статус ModemManager в реальном времени)
   ├── Full Status Dump (Сводный отчет по всему стеку)
   └── Back / Exit

3) Maintenance & Resets
   ├── Hard Reboot Modem (Программный сброс DMS Reset)
   ├── Nuclear USB Port Reset (Сброс на уровне USB-шины через sysfs)
   ├── Clear Notifications (Очистка очереди уведомлений eUICC)
   ├── Select Devices (Выбор портов QMI и AT)
   └── Doctor (Проверка зависимостей и окружения)
</pre>

<hr>

<h2>Переменные окружения lpac</h2>
<p>Для корректной работы lpac в OpenWrt используются следующие переменные (настраиваются автоматически скриптом или враппером):</p>

<table border="1" cellpadding="5">
    <tr>
        <th>Переменная</th>
        <th>Значение по умолчанию</th>
        <th>Описание</th>
    </tr>
    <tr>
        <td><code>LPAC_APDU</code></td>
        <td><code>qmi</code></td>
        <td>Тип бекенда (qmi или at)</td>
    </tr>
    <tr>
        <td><code>LPAC_HTTP</code></td>
        <td><code>curl</code></td>
        <td>Метод загрузки данных через libcurl</td>
    </tr>
    <tr>
        <td><code>LPAC_APDU_QMI_DEVICE</code></td>
        <td><code>/dev/cdc-wdm0</code></td>
        <td>Путь к управляющему порту QMI</td>
    </tr>
    <tr>
        <td><code>LPAC_APDU_QMI_UIM_SLOT</code></td>
        <td><code>1</code></td>
        <td>Номер слота SIM (обычно 1 для M.2 модемов)</td>
    </tr>
</table>

<hr>

<h2>Известные проблемы и решения</h2>
<ul>
    <li>
        <b>Ошибка "Invalid transition":</b> Возникает, если ModemManager пытается управлять модемом, который находится в процессе ребута. 
        <i>Решение:</i> Использовать режим <b>Hard Reboot</b> в скрипте, который инициирует горячее переподключение (Hotplug).
    </li>
    <li>
        <b>Кэширование профиля на X55:</b> Модем успешно меняет профиль, но не видит сеть или видит старого оператора.
        <i>Решение:</i> Использовать пункт <b>Nuclear USB Port Reset</b> (Level 3) в меню Maintenance.
    </li>
    <li>
        <b>Проблема со временем (NTP):</b> Если на роутере не синхронизировано время, lpac не сможет скачать профили из-за ошибок SSL.
        <i>Решение:</i> Дождитесь поднятия интернета или установите время вручную командой <code>date -s</code>.
    </li>
</ul>

<p align="right"><i>Developed for OpenWrt Community. 2026.</i></p>

</body>
</html>

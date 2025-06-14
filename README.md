Документация проекта: Сайт "Подборки мест для отдыха"
1. Обзор проекта
Цель проекта:Создать веб-приложение, которое позволяет пользователям просматривать подборки мест для отдыха по сезонам (лето, зима, весна, осень), регистрироваться, управлять профилем, добавлять места в избранное и предоставляет администратору (первому зарегистрированному пользователю) возможность управлять пользователями.
Ключевые функции:

Регистрация и авторизация с шифрованием паролей.
Назначение роли администратора первому пользователю.
Личный кабинет: редактирование профиля (имя, email) и удаление аккаунта.
Избранное: сохранение мест в базе для авторизованных пользователей, в браузере для гостей.
Админ-панель: просмотр и удаление пользователей.
Баннер cookies для согласия на использование локального хранилища.

Технологии:

PHP: Серверная логика (регистрация, авторизация, работа с базой).
MySQL: Хранение данных (пользователи, избранное).
JavaScript: Управление баннером cookies и избранным для гостей.
MAMP: Локальный сервер (Apache для PHP, MySQL для базы).
HTML и CSS используются для структуры и стилей, но не будут подробно разбираться.

Особенности:

Первый зарегистрированный пользователь автоматически становится администратором.
Безопасность: шифрование паролей, защита от SQL-инъекций через подготовленные выражения.
Нет сторонних библиотек (например, jQuery, Bootstrap), весь код написан вручную.


2. Структура проекта
Проект размещён в папке /Applications/MAMP/htdocs, где MAMP запускает локальный сервер. Вот структура файлов, связанных с ключевой логикой:
htdocs/
├── js/
│   └── cookie.js         # Скрипт для баннера cookies
├── index.php            # Главная страница
├── summer.php           # Подборка мест для лета
├── winter.php           # Подборка мест для зимы
├── spring.php           # Подборка мест для весны
├── autumn.php           # Подборка мест для осени
├── favorites.php        # Страница избранного
├── register.php         # Регистрация
├── login.php            # Авторизация
├── profile.php          # Личный кабинет
├── admin.php            # Админ-панель
├── logout.php           # Выход
├── config.php           # Подключение к базе
├── delete_user.php      # Удаление пользователя админом
├── toggle_favorite.php  # Добавление/удаление в избранное
├── remove_favorite.php  # Удаление из избранного
└── database.sql         # SQL-скрипт для базы

Описание файлов:

PHP-файлы: Реализуют серверную логику и взаимодействие с базой.
cookie.js: Управляет баннером cookies (сохраняет согласие в localStorage).
database.sql: Создаёт базу travel_db с таблицами users и favorites.
Папка css/ (с styles.css) и images/ не рассматриваются, так как связаны с дизайном.


3. Роль базы данных
База: travel_dbСистема: MySQL (в MAMP)Таблицы:

users:
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role ENUM('user', 'admin') DEFAULT 'user',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


id: Уникальный идентификатор пользователя.
username: Имя пользователя.
email: Уникальный email.
password: Зашифрованный пароль (с помощью password_hash).
role: admin для первого пользователя, user для остальных.
created_at: Дата регистрации.


favorites:
CREATE TABLE favorites (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    item_id VARCHAR(50) NOT NULL,
    title VARCHAR(100) NOT NULL,
    img VARCHAR(255) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);


id: Уникальный идентификатор записи.
user_id: ID пользователя.
item_id: ID места (например, "autumn1").
title: Название места.
img: Путь к изображению.
ON DELETE CASCADE: Удаление пользователя удаляет его избранное.



Зачем база?

Хранит пользователей (их данные, роли, пароли).
Сохраняет избранное для авторизованных пользователей.
Обеспечивает работу админ-панели (список пользователей).

Взаимодействие:

PHP подключается к базе через config.php:session_start();
$db_host = 'localhost';
$db_user = 'root';
$db_pass = 'root';
$db_name = 'travel_db';
$db_port = 8889;
$conn = mysqli_connect($db_host, $db_user, $db_pass, $db_name, $db_port);
if (!$conn) {
    die("Ошибка подключения: " . mysqli_connect_error());
}


Используются подготовленные выражения для SQL-запросов, чтобы предотвратить инъекции.


4. Ключевые элементы логики
4.1. Регистрация (register.php)
Цель: Создание нового пользователя, назначение роли admin первому зарегистрированному.
Логика:

Пользователь отправляет форму (POST-запрос) с полями: username, email, password, confirm_password.
Проверки:
Поля не пустые.
Email корректный (filter_var).
Пароль ≥ 6 символов.
Пароли совпадают.
Email не занят:$stmt = mysqli_prepare($conn, "SELECT id FROM users WHERE email = ?");
mysqli_stmt_bind_param($stmt, 's', $email);
mysqli_stmt_execute($stmt);
mysqli_stmt_store_result($stmt);
if (mysqli_stmt_num_rows($stmt) > 0) {
    $errors[] = "Этот email уже зарегистрирован.";
}




Назначение роли:
Проверяется количество пользователей:$check_users = mysqli_query($conn, "SELECT COUNT(*) as total FROM users");
$row = mysqli_fetch_assoc($check_users);
$role = ($row['total'] == 0) ? 'admin' : 'user';


Первый пользователь получает role = 'admin', остальные — role = 'user'.


Сохранение:
Пароль шифруется:$hashed_password = password_hash($password, PASSWORD_DEFAULT);


Данные записываются в users:$stmt = mysqli_prepare($conn, "INSERT INTO users (username, email, password, role) VALUES (?, ?, ?, ?)");
mysqli_stmt_bind_param($stmt, 'ssss', $username, $email, $hashed_password, $role);
mysqli_stmt_execute($stmt);




Создаётся сессия:$_SESSION['user_id'] = mysqli_insert_id($conn);
$_SESSION['username'] = $username;
$_SESSION['role'] = $role;
header("Location: profile.php");



Безопасность:

Подготовленные выражения защищают от SQL-инъекций.
Пароль хранится в зашифрованном виде (password_hash).


4.2. Авторизация (login.php)
Цель: Вход в аккаунт с проверкой пароля.
Логика:

Пользователь отправляет форму с email и password.
Проверяется существование email:$stmt = mysqli_prepare($conn, "SELECT id, username, password, role FROM users WHERE email = ?");
mysqli_stmt_bind_param($stmt, 's', $email);
mysqli_stmt_execute($stmt);
$result = mysqli_stmt_get_result($stmt);
$user = mysqli_fetch_assoc($result);


Проверяется пароль:if (password_verify($password, $user['password'])) {
    $_SESSION['user_id'] = $user['id'];
    $_SESSION['username'] = $user['username'];
    $_SESSION['role'] = $user['role'];
    header("Location: profile.php");
} else {
    $errors[] = "Неверный email или пароль.";
}


При успехе создаётся сессия, пользователь перенаправляется в личный кабинет.

Безопасность:

Пароль проверяется через password_verify, сравнивая с зашифрованным.
Подготовленные выражения для SQL.


4.3. Личный кабинет (profile.php)
Цель: Редактирование профиля и удаление аккаунта.
Логика:

Редактирование:
Показываются текущие данные:$stmt = mysqli_prepare($conn, "SELECT username, email FROM users WHERE id = ?");
mysqli_stmt_bind_param($stmt, 'i', $user_id);
mysqli_stmt_execute($stmt);
$result = mysqli_stmt_get_result($stmt);
$user = mysqli_fetch_assoc($result);


Пользователь отправляет форму с новым username и email.
Проверки:
Поля не пустые.
Email корректный.
Email не занят другим пользователем:$stmt = mysqli_prepare($conn, "SELECT id FROM users WHERE email = ? AND id != ?");
mysqli_stmt_bind_param($stmt, 'si', $new_email, $user_id);




Обновление:$stmt = mysqli_prepare($conn, "UPDATE users SET username = ?, email = ? WHERE id = ?");
mysqli_stmt_bind_param($stmt, 'ssi', $new_username, $new_email, $user_id);
mysqli_stmt_execute($stmt);
$_SESSION['username'] = $new_username;




Удаление аккаунта:
Пользователь подтверждает действие через JavaScript (confirm).
Удаление из users:$stmt = mysqli_prepare($conn, "DELETE FROM users WHERE id = ?");
mysqli_stmt_bind_param($stmt, 'i', $user_id);
mysqli_stmt_execute($stmt);


Избранное удаляется автоматически (ON DELETE CASCADE).
Сессия уничтожается:session_destroy();
header("Location: index.php");





Безопасность:

Проверка авторизации:if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
}


Подготовленные выражения.


4.4. Админ-панель (admin.php)
Цель: Управление пользователями (просмотр, удаление).
Логика:

Проверяется роль:if (!isset($_SESSION['user_id']) || $_SESSION['role'] !== 'admin') {
    header("Location: login.php");
}


Получается список пользователей:$query = "SELECT id, username, email, role, created_at FROM users";
$result = mysqli_query($conn, $query);
$users = mysqli_fetch_all($result, MYSQLI_ASSOC);


Удаление через delete_user.php:$stmt = mysqli_prepare($conn, "DELETE FROM users WHERE id = ?");
mysqli_stmt_bind_param($stmt, 'i', $user_id);
mysqli_stmt_execute($stmt);



Безопасность:

Ограничение доступа только для админа.
Подготовленные выражения в delete_user.php.


4.5. Избранное (favorites.php, toggle_favorite.php, remove_favorite.php)
Цель: Сохранение и отображение избранных мест.
Логика:

Для авторизованных:
Добавление/удаление в toggle_favorite.php:$stmt = mysqli_prepare($conn, "SELECT id FROM favorites WHERE user_id = ? AND item_id = ?");
if (mysqli_stmt_num_rows($stmt) > 0) {
    $stmt = mysqli_prepare($conn, "DELETE FROM favorites WHERE user_id = ? AND item_id = ?");
} else {
    $stmt = mysqli_prepare($conn, "INSERT INTO favorites (user_id, item_id, title, img) VALUES (?, ?, ?, ?)");
}


Отображение в favorites.php:$stmt = mysqli_prepare($conn, "SELECT item_id, title, img FROM favorites WHERE user_id = ?");


Удаление в remove_favorite.php:$stmt = mysqli_prepare($conn, "DELETE FROM favorites WHERE user_id = ? AND item_id = ?");




Для гостей:
Используется localStorage через JavaScript:function toggleFavorite(itemId, title, img) {
    let favorites = JSON.parse(localStorage.getItem('favorites') || '[]');
    if (favorites.some(item => item.itemId === itemId)) {
        favorites = favorites.filter(item => item.itemId !== itemId);
    } else {
        favorites.push({ itemId, title, img });
    }
    localStorage.setItem('favorites', JSON.stringify(favorites));
}






4.6. Баннер Cookies (cookie.js)
Цель: Запрос согласия на использование localStorage.
Логика:

Проверяется, принято ли согласие:if (!localStorage.getItem('cookiesAccepted')) {
    document.getElementById('cookieConsent').style.display = 'flex';
}


При нажатии "Принять" сохраняется:function acceptCookies() {
    localStorage.setItem('cookiesAccepted', 'true');
    document.getElementById('cookieConsent').style.display = 'none';
}




5. Технологии и их роль

PHP: Обрабатывает запросы, управляет сессиями, взаимодействует с базой.
MySQL: Хранит данные в таблицах users и favorites.
JavaScript: Управляет баннером cookies и избранным для гостей.
MAMP: Предоставляет сервер (Apache для PHP, MySQL для базы).
HTML и CSS не рассматриваются, так как связаны с дизайном.


6. Инструкции для запуска

Установите MAMP:
Скачайте и установите MAMP в /Applications/MAMP.


Разместите файлы:
Скопируйте проект в /Applications/MAMP/htdocs.


Создайте базу:
Откройте http://localhost:8888/phpMyAdmin (логин: root, пароль: root).
Создайте базу travel_db, импортируйте database.sql.


Запустите MAMP:
Откройте MAMP, нажмите Start Servers.


Откройте сайт:
Перейдите на http://localhost:8888/index.php.


Тестируйте:
Зарегистрируйтесь (первый пользователь — админ).
Войдите, отредактируйте профиль, добавьте место в избранное.
Под админом откройте admin.php.




7. Заключение
Проект демонстрирует создание динамического веб-приложения с авторизацией, управлением пользователями и избранным. Ключевые элементы:

Назначение админа первому пользователю.
Шифрование паролей (password_hash, password_verify).
Безопасные SQL-запросы.
Сессии для авторизации.
Логика избранного в базе и localStorage.

Для преподавателя:

Проект показывает понимание серверной логики, работы с базами, сессиями и безопасностью.
Код структурирован, использует современные практики.
MAMP имитирует реальный сервер для локальной разработки.

Если нужны уточнения, я готов ответить на вопросы преподавателя!

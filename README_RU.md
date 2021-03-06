Vkontakte Api for PHP
======================

Работа с API Вконтакте для StandAlone приложений на языке php.

Для работы с api вам необходимо выполнить несколько действий:

1. Создать **"Standalone-приложение"** https://vk.com/editapp?act=create
2. Получить access_token (об этом ниже)
3. Классу нужно передать **client_id** приложения и секретный ключ который вам даётся при создании приложения

## Получение access_token

Выполним метод get_code_token для получения ссылки которая вернёт нам code

```php
	include_once 'vk.php';

	$v = new Vk(array(
		'client_id' => 12345, // (обязательно) номер приложения
		'secret_key' => 'XXXXXX', // (обязательно) получить тут https://vk.com/editapp?id=12345&section=options где 12345 - client_id
		'user_id' => 12345, // ваш номер пользователя в вк
		'scope' => 'wall', // права доступа
		'v' => '5.35' // не обязательно
	));

	$url = $v->get_code_token();

	echo $url;
```

Переменная **$url** будет содержать ссылку при переходе на которую вас попросят авторизоваться и предоставить права приложению, после чего вас перекинут на пустую страницу и в URL будет access_token=**нужный код**.

## Выполнение Api

Для выполнения определённых Api вам необходимы на это права, для этого при создании токена нужно указать нужные scope.

Для того чтобы получить необходимые права во время авторизации, при открытии окна авторизации нужно передать параметр scope, содержащий названия необходимых ему прав, разделённых пробелом или запятой.

https://vk.com/dev/permissions

```php
	$config['secret_key'] = 'ваш секретный ключ приложения';
	$config['client_id'] = 12345; // номер приложения
	$config['user_id'] = 12345; // id текущего пользователя (не обязательно)
	$config['access_token'] = 'ваш токен доступа';
	$config['scope'] = 'wall,photos,video'; // права доступа к методам (для генерации токена)

	$v = new Vk($config);

	// пример публикации сообщения на стене пользователя
	// значения массива соответствуют значениям в Api https://vk.com/dev/wall.post

	$response = $v->api('wall.post', array(
	    'message' => 'I testing API form https://github.com/fdcore/vk.api'
	));

	// или

	$response = $v->wall->post(array(
	    'message' => 'I testing API form https://github.com/fdcore/vk.api'
	));
```

## Execute

Универсальный метод, который позволяет запускать последовательность других методов, сохраняя и фильтруя промежуточные результаты.

Подробнее https://vk.com/dev/execute


```
	$v->execute('return API.wall.post({message: "Проверка Execute ^_^"});'));
```

## Заливка файлов

Для заливки файлов в данный момент есть 3 метода:

- Загрузка видеозаписей **$v->upload_video()**
- Загрузка фотографий на стену пользователя **$v->upload_photo()**
- Загрузка документов **$v->upload_doc()**

### Пример загрузки фото

Для загрузки фотографии, существует метод **upload_photo()**.

Параметры:

- **$gid** - (стандартно 0) идентификатор сообщества, на стену которого нужно загрузить фото (без знака «минус»). (*целое число*)
- **$files** - массив путей к файлам (например array('4b67bhWrc4g.jpg', 'n52W2BdXdYE.jpg'))
- **$return_ids** - (стандартно false) возвращать id файлов или готовые строки для прикрепления (например photo12345_6789)

```php
    // загрузка фото на сервер
    $attachments = $v->upload_photo(0, array('4b67bhWrc4g.jpg', 'n52W2BdXdYE.jpg'));

    // публикация на стене
    $response = $v->wall->post(array(
        'message'=>'я публикую фотографии',
        'attachments' => implode(',', $attachments)
      )
    );

```

### Пример загрузки видео

```php

   // встраивание видео с YouTube без заливки

   $attach_video = $v->upload_video(array(
       'link'=>'https://youtu.be/exAmqVtYbis',
       'title' => 'Hatsune Miku Project Diva 2nd Opening Full HD',
       'description' => "First Song: "Kocchi Muite baby" by ryo and kz",
       'wallpost' => 1
   ));

   // заливка видео на VK.com

   $attach_name = $v->upload_video(
       array('name' => 'Test video',
           'description' => 'My description',
           'wallpost' => 1,
           'group_id' => 0
       ), 'video.mp4');

```

### Пример загрузки документа

```php
    $attach_doc_file = $v->upload_doc(0, 'iZKE4JdP4Q0mT.jpg');

    if ( is_string($attach_doc_file) ) echo $attach_doc_file;

```

# API
#### Нахождение лиц и их эмбеддингов
##### Формат запроса
```
GET: url
```

##### Пример запроса
```
GET /recognize 

{
url=https://pp.userapi.com/c845018/v845018291/3c404/HEx7GhxeJUk.jpg
}
```

##### Параметры запроса
|Название   |  Тип | Описание  |  Обязательный |
|---|---|---|---|
|  url |  string | url картинки, на которой небходимо найти лица  | да  |


##### Пример ответа
```
{
    "boxes": [
        [
            340,
            214,
            399,
            287
        ]
    ],
    "confidence": [
        0.9702990651130676
    ],
    "embeddings": [
        [
            -0.13520943,
            -0.06863844,
            -0.12447117,
            ..........
	    ..........
	    ..........
            -0.10029904,
            0.19816388,
            0.18933909
        ]
    ],
    "faces_num": 1
}
```

##### Параметры ответа

| Название  |  Тип | Описание  |
|---|---|---|
| boxes  | list of lists | Возвращает координаты найденных лиц, каждый элемент boxes это list [x1, y1, x2, y2 ], где x1 < x2, y1 < y2 |
| confidence  | list  | Уверенность нейронки в том, что найденная область, действительно, является лицом  |
|  embeddings | list of lists  | Encoding лиц, каждый эмбеддинг имеет размерность 128|
|  faces_num |  int | Число, найденных лиц |

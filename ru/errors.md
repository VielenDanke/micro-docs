Цикл статей по микро - ошибки

Ошибки - пакет еррорс используется для передачи ошибок между сервисами для консистентной передачи ошибок. Особенно важно это при использовании грпц клиента и сервера, так как в противном случае будет утерян оригинальный код ошибки.
Внутри самого сервиса можно использовать любые виды ошибок.
Ошибка содержит код, описание и идентификатор сервиса, который выдал ошибку.

Туду - еррор эмиттер, клиент, брокер или сервер, или даже клиентский сервис при создании создают инстанс еррор эмиттера и используют для генерации ошибок.

Таким образом все ошибки имеют консистентной вид. В качестве айди нельзя передавать ууид или другой идентификатор, так как это все присутствует в метаданных в контексте.

Пакет еррорс используется только внутри экосистемы микро, наружу пользователям выдается только внешняя ошибка, вместе с идентификатором запроса, для последующего обращения в службу поддержки при необходимости.

Вопрос, нужно ли логировать ошибки внутри фреймворка или только отдавать наружу


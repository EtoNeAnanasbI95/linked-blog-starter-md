JWT -- JSON Web token
* Удобный декодер: ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABwAAAAcCAMAAABF0y+mAAAAvVBMVEVHcEwcHBwZGRkZGRkZGRkaGhoFBQUAAAAVFhSBgYG5ubgWBQkYPDb///85HicGFxUnX1NI6cs/v6iys7PPQzf+Tjx0Jh9J79E0kYOeNyr/UEFaHxU8uKEXFQcXIB9E1rw0zbLjPCzsSjwAFgQvP5YoMHQeGjF6sqe8fHhNIUSAK3KaL4Q+VeBAWukzTcEZcGGura2sOknWPLr+RN//RuctX3qNMFkdKFA5UdHqP8uvMpUdQJQcOYHePL1iI1L51Fu6AAAABnRSTlMAFrD4/7B/5ke/AAABhElEQVR4AX2TBXLDMBBF7UaJtzGu2UqDhjAz3v9YlapoGv6D0pOWV1HUrxJ5odKXyhh5K1Up35wqmla5OZYVaROAVL6r1e8bWlKIkG4YoFVNs6qBZdlE6ArBQVfXPNP0NNsPQltCwSKMYgETSmnNvv2p/0SIBoc1xvzbn/V6vdFEbHGfbUo7xE5TCetZXugOInZNsxfQoJb2B8NU/hyNx5M8xmg6m4U0mA8Wy9USBKznY67ROnZII0w2jCyXq4H9B2Gb7TjdFTohdrFfMu0PfRBmoQ7b425yKgiTNTzvDxdIGRMwhcthfx5a/KI4TXbHLdSvEPoHYapgbvRCuMi2wqw9+ItguUnCBnHi9WjMldfFT2BkMZgHNPRm0wjjfMJil0VIhwMrZWULeqbZRURHL/KsLiCjqU06lLZ5+VqIzQar503hbZ/3ghfeQIx+9NuWNXgvEtGymHXPAXL7Mwx8W0DdRcEkZNSy5JgYhk6E/geMyAEDkAP2cTQ/D7Will+vQ1lVfgHa0zEXT+rLEwAAAABJRU5ErkJggg==) https://jwt.io 

## Схема токена
- Токен состоит из трёх частей:
    1. Header (там лежит алгоритм шифрования и тип токена <короче это просто статичная история, которая никак не меняется и всегда будет чёто типа “alg: HS256, type: jwt”>)
    2. Payload (одна из двух самый важных частей, в ней пишется время истечения токена и какая-то инфа о пользователе, просто чтобы сервер мог ауинтифицировать кто в него стучится, обычно это просто айди пользователя из бд)
    3. Signature (вторая самая важная часть, она является шифром header и payload. Шифр происходит посредством использования алгоритма шифрования и секрета, который знает ток сервер. То есть сервер делает этот токен, и никто не может его изменить, а чтобы провалидировать этот токен серверу не надо лезть в бд)

## Почему это круто
- Весь сок работы jwt токенов в том, что нам не нужно постоянно дудосить БД и спрашивать у неё пароль пользователя. Только сам сервер знает секрет, по которому определяется итоговый токен, и только он способен как-то менять итоговый токен, в противном случае токен будет просто не валидный. Помимо этого в этом же токене мы можем передавать какою-то информацию, например время действия токена (да, оно записывается именно там) и получается так, что токен сам по себе знает когда он кончается и изменить это никак нельзя, иначе изменится его подпись. Токен это чёто типа заклинания которое само знает когда оно перестанет действовать и изменить его срок никто не может, потому что подпись не будет совпадать

# Схема взаимодействия клиента и сервера авторизации
![[Pasted image 20250829023425.png]]
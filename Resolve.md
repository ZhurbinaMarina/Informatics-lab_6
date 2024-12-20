## Выполнение лабораторной работы # 6

1. Для начала создадим две папки, Пети и Вани соответсвенно:
```
mkdir petya
mkdir vanya
```

2. Генерация пары ключей, открытый / закрытый:
   
   Первым шагом Пети, как получателя, является создание пары ключей в своей папке
```
cd petya
gpg --batch --generate-key
```

  При этом будет сгенерирована пара ключей и сохранена в файле ryanpubring.kbx keyring в том же месте.

3. Давайте посмотрим, как вводится открытый ключ в файл keyring:
```
gpg --keyring ./petyapubring.kbx --no-default-keyring --list-keys
```
Вывод:
```
./petyapubring.kbx
-----------------
pub   rsa3072 2019-10-27 [SCEA] [expires: 2019-11-26]
      120C528F1D136BCF7AAACEE6D6BA055613B064D7
uid           [ unknown] Petya <petya@somewhere.com>
sub   rsa3072 2019-10-27 [SEA] [expires: 2019-11-26]
```

Индикатор pub указывает на открытый ключ.

Аналогичным образом мы можем просмотреть запись закрытого ключа:
```
gpg --keyring ./petyapubring.kbx --no-default-keyring --list-secret-keys
```
Вывод:
```
./petyapubring.kbx
-----------------
sec   rsa3072 2019-10-27 [SCEA] [expires: 2019-11-26]
      120C528F1D136BCF7AAACEE6D6BA055613B064D7
uid           [ultimate] Petya <petya@somewhere.com>
ssb   rsa3072 2019-10-27 [SEA] [expires: 2019-11-26]
```
Здесь sec указывает, что это секретный ключ.

4. После успешной генерации ключа Петя может экспортировать открытый ключ из своей связки ключей в файл:
```
gpg --keyring ./petyapubring.kbx --no-default-keyring --armor --output petyapubkey.gpg --export petya@somewhere.com
```
При этом будет сгенерирован новый файл petyapubkey.gpg, содержащий открытый ключ.

Посмотрим на содержание этого файла:
```
cat petyapubkey.gpg
```
Вывод:
```
-----BEGIN PGP PUBLIC KEY BLOCK-----
mQGNBF21KQoBDACs7bgjl22TPyQDKjLTMlZrBgQrXZOIkNcH3z1f87XQYoLjVPU3
ymg1hweHm1RsIxO+GdD42pkU/ob5YdWgvVBRdIZPeTXciTa8TtxZKNNtr+IL0pwY
...
-----END PGP PUBLIC KEY BLOCK-----
```

5. Теперь Петя может поделиться этим файлом с Ваней по защищенным или незащищенным каналам. В нашем случае сделаем простую копию файла для обмена открытым ключом:
```
cp petyapubkey.gpg ../vanya
```

6. Теперь работа за Ваней, сначала перейдем в его папку:
```
cd ../vanya
```

7. Затем импортируем открытый ключ Пети в файл связки ключей Вани:
```
gpg --keyring ./vanyapubring.kbx --no-default-keyring --import petyapubkey.gpg
```

Это создаст новый файл набора ключей vanyapubring.kbx и добавит к нему открытый ключ.

8. Теперь мы можем просмотреть импортированный ключ:
```
gpg --keyring ./vanyapubring.kbx --no-default-keyring --list-keys
```
Вывод:
```
uid           [ unknown] Ryan <ryan@somewhere.com>
```

Значение [unknown] указывает, что информация о достоверности ключа недоступна. Чтобы избежать предупреждений в будущем, давайте изменим это значение на trusted:
```
gpg --keyring ./vanyapubring.kbx --no-default-keyring --edit-key "petya@somewhere.com" trust
```

В заданном вопросе уровня доверия давайте определим наш выбор как 5 = я доверяю в конечном итоге и подтвердим как да. После этого давайте наберем quit для выхода из командной строки.

9. Теперь у Вани все готово для шифрования файла, который может прочитать только Петя. Давайте создадим образец файла:
```
echo "Hello, Petya!" > greetings.txt
```

10. После этого давайте укажем Петю в качестве получателя в команде encrypt:
```
gpg --keyring ./vanyapubring.kbx --no-default-keyring --encrypt --recipient "petya@somewhere.com" greetings.txt
```

Это создает файл greetings.txt.gpg в том же месте и зашифрованный с использованием открытого ключа Пети. Теперь Ваня может поделиться этим файлом с Петей по защищенным или незащищенным каналам.

Как и раньше, сделаем простую копию файла для совместного использования зашифрованного файла:
```
cp greetings.txt.gpg  ../petya
```

11. Теперь давайте вернемся к папке Пети, чтобы прочитать зашифрованный файл, который он получил от Вани:
```
cd ../petya
```

12. Мы будем использовать команду decrypt, чтобы создать новый файл greetings.txt, расшифрованный с использованием закрытого ключа Пети:
```
gpg --keyring ./ryanpubring.kbx --no-default-keyring --pinentry-mode=loopback --passphrase "ryanpassword" --output greetings.txt --decrypt greetings.txt.gpg
```

13. Теперь проверим, прошла ли расшифровка успешно:
```
diff -s greetings.txt ../sam/greetings.txt
```
Вывод, который позволяет нам понять, что все прошло успешно:
```
Files greetings.txt and ../sam/greetings.txt are identical
```

## Лабораторная работа выполнена!

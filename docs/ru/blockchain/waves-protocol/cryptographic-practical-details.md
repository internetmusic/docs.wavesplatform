# Криптографические практические детали

## Описание

В этом разделе описываются детали криптографических алгоритмов, которые используются для:

1. Создания приватных и публичных ключей из сида.  
2. Создания адресов из публичного ключа.  
3. Создания блоков и подписи транзакций.

Мы используем:

1. `Blake2b256` и `Keccak256` алгоритмы (в виде цепочки хэшей) для создания криптографических хэшей.
2. `Curve25519` (ED25519 с ключами X25519) для создания и верификации подписей.  
3. `Base58` для создания стринга байтов.

**Примечание**: Мы используем KECCAK, который слегка отличается от SHA-3 \(FIPS-202\).

## Кодировка байтов в Base58

Все массивы байтов в проекте кодируются алгоритмом Base58 с биткойн алфавитом для удобочитаемости.

### Пример

Строка `teststring` закодирована в байты `[5, 83, 9, -20, 82, -65, 120, -11]`. Байты `[1, 2, 3, 4, 5]` закодированы в строку `7bWpTW`.

## Создание приватного ключа из сида

Сид - это представление энтропии, из которого вы можете детерминистически воссоздать все закрытые ключи для одного кошелька. Сид должен быть достаточно длинным, чтобы вероятность выбора была нереально незначительной.

Фактически, сид должен быть массивом байтов, но для простоты запоминания кошелек использует [Brainwallet] (https://en.bitcoin.it/wiki/Brainwallet), чтобы сид состоял из слов, которые легко записать или запомнить. Приложение берет UTF-8 байты строки и использует их для создания ключей и адресов.

Например сид строка `manage manual recall harvest series desert melt police rose hollow moral pledge kitten position add` после прочтения в UTF-8 байтах и кодировки в Base58, получится следующее `xrv7ffrv2A9g5pKSxt7gHGrPYJgRnsEMDyc4G7srbia6PhXYLDKVsDxnqsEqhAVbbko7N1tDyaSrWCZBoMyvdwaFNjWNPjKdcoZTKbKr2Vw9vu53Uf4dYpyWCyvfPbRskHfgt9q`.

Сид строка задействуется при создании закрытых ключей. Чтобы создать закрытый ключ с использованием официального веб-кошелька или ноды, необходимо 4 байта поля int 'nonce' \(представление с прямым порядком байтов\), с изначальным значением 0, которое увеличивается при каждом создании нового адреса, добавляясь к байтам сида. Затем мы используем этот массив байтов для вычисления хеша `keccak256 (blake2b256 (bytes))`. Получившийся массив байтов называется `account seed`, из него вы можете сгенерировать одну пару закрытых и открытых ключей. Затем этот байтовый хэш передается в метод создания пары открытого и закрытого ключей алгоритма `Curve25519`.

Waves использует `Curve25519`-`ED25519` подпись с ключами X25519 \(форма Монтгомери\), но большинство встроенных криптографических устройств и библитек не поддерживают ключи X25519.

Существуют [библиотеки с функциями конверсии](https://download.libsodium.org/doc/advanced/ed25519-curve25519) из:

* ED25519 ключей в X25519 \(Curve25519\) crypto\_sign\_ed25519\_pk\_to\_curve25519\(curve25519\_pk, ed25519\_pk\) для **публичного ключа**.
* Crypto\_sign\_ed25519\_sk\_to\_curve25519\(curve25519\_sk, ed25519\_skpk\) для **приватного ключа**.

Например, при использовании ключей ED25519 и подписи внутри приложения Ledger, затем необходимо преобразовать ключи из устройства в формат X25519, используя эту функцию на стороне клиента **и создать адрес waves из публичного ключа X25519**. [Есть пример преобразования libsodium ED25519 ключей и подписи в Curve25519](https://gist.github.com/Tolsi/d64fcb09db4ead75e5eeeab445284c93).

**Примечание**: Не все случайные 32 байта могут использоваться как закрытые ключи \(но любые байты любого размера могут быть сидом\). Схема подписи для ED25519 вводит ограничения для ключей, поэтому создавайте ключи только с помощью методов библиотек Curve25519 и обязательно проверяйте способность подписывать данные с помощью закрытого ключа, а затем проверять их с помощью открытого ключа, каким бы очевидным ни казался этот тест.

Существуют валидные реализации Curve25519 для разных языков:

* [Java](https://github.com/signalapp/curve25519-java/)
* [C](https://github.com/signalapp/curve25519-java/tree/master/android/jni)
* [Python](https://github.com/tgalal/python-axolotl-curve25519)
* [JS](https://github.com/wavesplatform/curve25519-js)

Кроме того, некоторые библиотеки `Curve25519` \(как и используемые в нашем проекте\) имеют встроенное хеширование `Sha256`, а некоторые нет \(например, большинство библиотек c/c++/python\), поэтому вам может потребоваться применить его вручную. Обратите внимание, что закрытый ключ ограничен, поэтому не любые случайные 32 байта могут быть валидным закрытым ключом.

### Пример

сид-строка Brainwallet

```bash
manage manual recall harvest series desert melt police rose hollow moral pledge kitten position add
```

закодирована как UTF-8

```bash
xrv7ffrv2A9g5pKSxt7gHGrPYJgRnsEMDyc4G7srbia6PhXYLDKVsDxnqsEqhAVbbko7N1tDyaSrWCZBoMyvdwaFNjWNPjKdcoZTKbKr2Vw9vu53Uf4dYpyWCyvfPbRskHfgt9q
```

Сид байты учетной записи с nonce 0 перед применением хэш-функции в Base58

```bash
1111xrv7ffrv2A9g5pKSxt7gHGrPYJgRnsEMDyc4G7srbia6PhXYLDKVsDxnqsEqhAVbbko7N1tDyaSrWCZBoMyvdwaFNjWNPjKdcoZTKbKr2Vw9vu53Uf4dYpyWCyvfPbRskHfgt9q
```

blake2b256 \(байты сида аккаунта\)

```bash
6sKMMHVLyCQN7Juih2e9tbSmeE5Hu7L8XtBRgowJQvU7
```

Сид аккаунта \(keccak256\(blake2b256\(байты сида аккаунта\)\)\)

```bash
H4do9ZcPUASvtFJHvESapnxfmQ8tjBXMU7NtUARk9Jrf
```

Сид аккаунта после хэширования `Sha256` \(опционально, сделайте самостоятельно, если ваша библиотека этого не может\)

```bash
49mgaSSVQw6tDoZrHSr9rFySgHHXwgQbCRwFssboVLWX
```

Созданный приватный ключ

```bash
3kMEhU5z3v8bmer1ERFUUhW58Dtuhyo9hE5vrhjqAWYT
```

Созданный публичный ключ

```bash
HBqhfdFASRQ5eBBpu2y6c6KKi1az6bMx8v1JxX4iW1Q8
```

## Создание адреса из публичного ключа

Сетевой адрес, полученный из открытого ключа, зависит от байта chainId («T» для testnet, «W» для mainnet, «S» для stagenet), поэтому разные сети получили разные адреса для одного сида (и, следовательно, открытые ключи) , Создание байтовых адресов описано более подробно [здесь](/en/blockchain/binary-format).

**Пример**:

Для публичного ключа

```bash
HBqhfdFASRQ5eBBpu2y6c6KKi1az6bMx8v1JxX4iW1Q8
```

в сети mainnet \(chainId 'W'\) будет создан следующий адрес

```bash
3PPbMwqLtwBGcJrTA5whqJfY95GqnNnFMDX
```

## Подпись

`Curve25519` используется для всех подписей проекта.

Применяется следующий процесс: сначала нужно создать специальные байты для подписи, для транзакции или для блока (описание представлено [тут](/en/blockchain/binary-format)), затем создать подпись, используя эти байты и байты закрытого ключа.

Для валидации подписи, достаточно байтов подписи, подписанных байтами объекта и публичным ключом.

Не забывайте, что существует много валидных \(не уникальных!\) подписей для одного массива байтов \(блок или транзакция\). Также не следует предполагать, что id блока или транзакции является уникальным. Однажды может произойти коллизия! Они уже имели место для некоторых слабых ключей.

### Пример

Данные транзакции:

| Поле | Значение |
| --- | --- |
| Адрес отправителя \(не применяется, просто для информации\) | 3N9Q2sdkkhAnbR4XCveuRaSMLiVtvebZ3wp |
| Приватный ключ \(используется для подписи, не в данных транзакции\) | 7VLYNhmuvAo5Us4mNGxWpzhMSdSSdEbEPFUDKSnA6eBv |
| Публичный ключ | EENPV1mRhUD9gSKbcWt84cqnfSGQP5LkCu5gMBfAanYH |
| Адрес получателя | 3NBVqYXrapgJP9atQccdBPAgJPwHDKkh6A8 |
| id актива | BG39cCNUFWPQYeyLnu7tjKHaiUGRxYwJjvntt9gdDPxG |
| Сумма | 1 |
| Комиссия | 1 |
| id актива комиссиии | BG39cCNUFWPQYeyLnu7tjKHaiUGRxYwJjvntt9gdDPxG |
| Временная отметка | 1479287120875 |
| Вложение \(как массив байтов\) | \[1, 2, 3, 4\] |

Байты:

| \# | Имя поля | Тип | Позиция | Длина | Значение | Значение в Base58 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Тип транзакции \(0x04\) | Byte | 0 | 1 | 4 | 5 |
| 2 | Публичный ключ отправителя | Bytes | 1 | 32 | ... | EENPV1mRhUD9gSKbcWt84cqnfSGQP5LkCu5gMBfAanYH |
| 3 | Актив \(0-Waves, 1-Asset\) | Byte | 33 | 1 | 1 | 2 |
| 4 | ID актива\(\*if used\) | Bytes | 34 | 0 \(32\*\) | ... | BG39cCNUFWPQYeyLnu7tjKHaiUGRxYwJjvntt9gdDPxG |
| 5 | Актив комиссии \(0-Waves, 1-Asset\) | Byte | 34 \(66\*\) | 1 | 1 | 2 |
| 6 | ID актива комиссии \(\*\*if used\) | Bytes | 35 \(67\*\) | 0 \(32\*\*\) | ... | BG39cCNUFWPQYeyLnu7tjKHaiUGRxYwJjvntt9gdDPxG |
| 7 | Временная отметка | Long | 35 \(67_\) \(99\*_\) | 8 | 1479287120875 | 11frnYASv |
| 8 | Сумма | Long | 43 \(75_\) \(107\*_\) | 8 | 1 | 11111112 |
| 9 | Комиссия | Long | 51 \(83_\) \(115\*_\) | 8 | 1 | 11111112 |
| 10 | Адрес получателя | Bytes | 59 \(91_\) \(123\*_\) | 26 | ... | 3NBVqYXrapgJP9atQccdBPAgJPwHDKkh6A8 |
| 11 | Длина вложения \(N\) | Short | 85 \(117_\) \(149\*_\) | 2 | 4 | 15 |
| 12 | Байты вложения | Bytes | 87 \(119_\) \(151\*_\) | N | \[1,2,3,4\] | 2VfUX |

_**Все байты данных для подписи:**_

`Ht7FtLJBrnukwWtywum4o1PbQSNyDWMgb4nXR5ZkV78krj9qVt17jz74XYSrKSTQe6wXuPdt3aCvmnF5hfjhnd1gyij36hN1zSDaiDg3TFi7c7RbXTHDDUbRgGajXci8PJB3iJM1tZvh8AL5wD4o4DCo1VJoKk2PUWX3cUydB7brxWGUxC6mPxKMdXefXwHeB4khwugbvcsPgk8F6YB`

_**Подпись байтов данных транзакции**_ **\(одна из бесконечного количества валидных подписей\):**

`2mQvQFLQYJBe9ezj7YnAQFq7k9MxZstkrbcSKpLzv7vTxUfnbvWMUyyhJAc1u3vhkLqzQphKDecHcutUrhrHt22D`

_**Все байты данных транзакции с подписью:**_

`6zY3LYmrh981Qbzj7SRLQ2FP9EmXFpUTX9cA7bD5b7VSGmtoWxfpCrP4y5NPGou7XDYHx5oASPsUzB92aj3623SUpvc1xaaPjfLn6dCPVEa6SPjTbwvmDwMT8UVoAfdMwb7t4okLcURcZCFugf2Wc9tBGbVu7mgznLGLxooYiJmRQSeAACN8jYZVnUuXv4V7jrDJVXTFNCz1mYevnpA5RXAoehPRXKiBPJLnvVmV2Wae2TCNvweHGgknioZU6ZaixSCxM1YzY24Prv9qThszohojaWq4cRuRHwMAA5VUBvUs`

## Расчёт ID транзакции

ID транзакции не сохраняется в байтах транзакции, и для большинства транзакций \(за исключением платежа\) его можно легко рассчитать из специальных байтов для подписи с использованием `blake2b256 (bytes_for_signing)`. Для платежной транзакции ID является только подписью этой транзакции.
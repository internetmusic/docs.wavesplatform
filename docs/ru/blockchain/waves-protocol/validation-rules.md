---
sidebarDepth: 2
---

# Правила валидации

## Валидация аккаунта

Аккаунт валиден когда он является валидной строкой Base58 с длиной соответствующего массива в 26 байт. Версия адреса \(1й байт\) равна 1. Тип сети \(2й байт\) равен ID сети. Checksum адреса \(последние 4 байта\) верен.

## Валидация транзакций

### Транзакция перевода

Транзакция перевода валидна, когда:

1. Адрес получателя валиден. Если нет, тогда результатом валидации будет _InvalidAddress_.
2. Размер вложения меньше или равен _MaxAttachementSize_ \(140 байт\). В противном случае, результатом валидации будет _TooBigArray_.
3. Сумма транзакции больше чем 0. В противном случае, результатом валидации будет _NegativeAmount_.
4. Комиссия транзакции больше 0. В противном случае, результатом валидации будет _InsufficientFee_.
5. Добавление комиссии к общей сумме не приводит к Long overflow. В противном случае, результатом валидации будет _OverflowError_.
6. Подпись транзакции валидна. В противном случае, результатом валидации будет _InvalidSignature_.

### Транзакция выпуска

Транзакция выпуска валидна, когда:

1. Адрес отправителя валиден. В противном случае, результатом валидации будет _InvalidAddress_.
2. Количество ассетов больше 0. В противном случае, результатом валидации будет _NegativeAmount_.
3. Комиссия траназкции больше или равна _MinFee_ \(100000000 wavelets = 1 Wave\). В противном случае, результатом валидации будет _InsufficientFee_.
4. Размер описания меньше или равен _MaxDescriptionLength_ \(1000 байт\). В противном случае, результатом валидации будет _TooBigArray_.
5. Размер имени больше или равен _MinAssetNameLength_ и меньше или равен _MaxAssetNameLength_. В противном случае, результатом валидации будет _InvalidName _.
6. Количество знаков после запятой больше 0 и меньше или равно _MaxDecimals_. В противном случае, результатом валидации будет _TooBigArray_.
7. Подпись транзакции валидна. В противном случае, результатом валидации будет _InvalidSignature_.

### Транзакция перевыпуска

Транзакция перевыпуска валидна, когда:

1. Аккаунт отправителя валиден. В противном случае, результатом валидации будет _InvalidAddress_.
2. Количество больше 0. В противном случае, результатом валидации будет _NegativeAmount_.
3. Комиссия транзакции больше 0. В противном случае, результатом валидации будет _InsufficienFee_.
4. Подпись транзакции валидна. В противном случае, результатом валидации будет _InvalidSignature_.

## Валидация блока

Блок валиден, когда:

1. Цепочка блоков содержит блок.
2. Подпись блока валидна.
3. КОнсенсус блока валиден.
4. Транзакции блока валидны.

### Валидация данных консенсуса

Консенсус блока валиден, когда:

1. Время создания блока не больше чем _MaxTimeDrift_ \(15 секунд\) в будущем.
2. Транзакции блока отсортированы. Это правило работает только после 1477958400000 в Testnet и 1479168000000 в Mainnet.
3. Цепочка блоков содержит родительский блок или высота блокчейна равна 1.
4. Base target блока валиден.
5. Подпись генератора блока валидна.
6. Баланс генератора больше или равен _MinimalEffectiveBalanceForGeneration_ \(1000000000000 wavelets\). Это правило всегда работает для Testnet и работает для Mainnet после 1479168000000.
7. Hit блока меньше чем расчётный target.

### Валидация данных транзакции

Транзакции блока валидны, когда:

1. Время создания каждой транзакции в блоке меньше чем время создания блока не более чем на _MaxTxAndBlockDiff_ \(2 часа\).
2. Все транзакции валидны по статусу against state.

#### Валидация транзакций Against State

Транзакции валидны, когда:

1. Транзакция валидна согласно правилам валидации транзакции.
2. Время создания транзакции больше чем время создания блока не более чем на _MaxTimeForUnconfirmed_ \(90 минут\). Это правило всегда работает для Testnet и работает для Mainnet после 1479168000000.
3. Транзакция не может привести к временному отрицательному балансу. Это правило работает после 1479168000000 на Mainnet и после 1477958400000 на Testnet.
4. Изменения сделанные транзакцией должны быть отсортированы по сумме. Это правило работает для Mainnet и Testnet после 1479416400000.
5. Транзакция не приводит текущий баланс к Long overflow.
6. После выполнения всех транзакций блока, задействованные балансы не должны быть отрицательными.

## Валидация пула неподтверженных транзакций

Транзакции могут быть вставлены в пул неподтвержденных, когда:

1. Транзакция валидна согласно правилам валидации транзакций.
2. Комиссия транзакции больше или равна минимальной комиссии, заданной вледльцем ноды.
3. Есть место для новой транзакции в пуле неподтверженных транзакций. По умолчанию пул ограничен до 1000 транзакций.
4. Пул неподтвержденных транзакций не содержит транзакции с одинаковыми ID.
5. Транзакция создана не позднее чем _MaxTimeForUncofimed_ \(90 минут\) после создания последнего блока.
6. Время создания транзакции не больше чем _MaxTimeDrift_ \(15 секунд\) в будущем.
7. Транзакция валидна по статусу against state.
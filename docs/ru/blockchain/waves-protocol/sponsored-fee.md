# Транзакции спонсорской комисии

## Сценарии использования

Пользователи могут устанавливать комиссию за транзакцию в заданном активе. Тем не менее, владельцы ноды должен разрешить комиссию за транзакции в заданном активе путем ручного редактирования файла конфигурации ноды. В противном случае нода не сможет майнить блок с этими транзакциями.

Спонсорство может быть задано для актива. В этом случае за обработку транзакций майнер получит комиссию (в Waves), которая задана в спонсируемом активе.

После подтверждения этой транзакции, становится возможным использовать актив в качестве комиссии (автоматически для всех майнеров). Когда появляется транзакция с комиссией в спонсорском активе комиссии, любой майнер помещает ее в forged блок. Вместо того, чтобы просто переводить комиссионный актив на баланс майнера, блокчейн совершает нечто иное: он автоматически перемещает комиссионный актив на счет спонсора (эмитента) и переводит стандартную стоимость транзакции в Waves со спонсорского счета на счет майнера. Фактически два майнера получат эти Waves из-за распределения платы NG 40/60.

Только эмитент актива может настроить спонсорство. Спонсорство устанавливается путем указания ставки, по которой плата в активе конвертируется в Waves.

## Реализация

### Представление комиссии спонсорской транзакции

Транзакция спонсорской комиссии в бинарном формате:

| \# | Имя поля | Тип | Позиция | Длина |
| --- | ---: | --- | --- | --- |
| 1 | Тип транзакции (0x0e) | Byte | 0 | 1 |
| 2 | Версия (0x01) |  Byte | 1 | 1 |
| 3 | Публичный ключ отправителя | Bytes | 2 | 32 |
| 4 | ID ассета | Bytes | 34 | 32 |
| 5 | Минимальная комиссия в ассетах\* | Long | 66 | 8 |
| 6 | Комиссия | Long | 74 | 8 |
| 7 | Временная отметка | Long | 82 | 8 |
| 8 | Пруфы\*\* | Bytes | 90 | 64 |

\* Нулевое значение означает отмену спонсосртва.

\*\* В данный момент только пподпись, подпись имеет Length = 64

Пример в JSON:

```js
{
  "type" : 14,
  "id" : "CwHecsEjYemKR7wqRkgkZxGrb5UEfD8yvZpFF5wXm2Su",
  "sender" : "3FjTpAg1VbmxSH39YWnfFukAUhxMqmKqTEZ",
  "senderPublicKey" : "5AzfA9UfpWVYiwFwvdr77k6LWupSTGLb14b24oVdEpMM",
  "minSponsoredAssetFee": 100000, // minimum amount of assets require for fee, set equal to null to cancel sponsorship
  "fee" : 100000000,
  "timestamp" : 1520945679531,
  "proofs" : [ "4huvVwtbALH9W2RQSF5h1XG6PFYLA6nvcAEgv79nVLW7myCysWST6t4wsCqhLCSGoc5zeLxG6MEHpcnB6DPy3XWr" ],
  "version" : 1,
  "height" : 303
}
```

## Комиссии

### Комиссия за транзакцию спонсорской комисии

Комиссия за спонсора оплачивается только в WAVES. Плата за эту транзакцию фиксирована и составляет 1,0 WAVES.

### Комиссия майнера в WAVES

Общую **комиссию майнера за транзакцию в WAVES** с комиссией в спонсорских ассетах можно посчитать с помощью следующей формулы:

```bash
    feeInWaves = assetFee * feeUnit / sponsorship
```

где:

* `assetFee` - комиссия в ассетах транзакции
* `feeUnit` - для спонсорства равен 100000
* `sponsorship` - значение `minSponsoredAssetFee` из транзакции спонсорской комисии для заданного ассета

Но общая **комиссия блока** за блок со спонсорской транзакцией можно посчитать как сумму транзакций которые имеют **комиссию только в WAVES**. Например, если есть блок только со спонсорскими транзакциями, комиссия для такого блока будет равна 0.

## API

`POST /assets/sponsor` подписывает и отправляет транзакцию запуска/обновления спонсорства. Для этой конечной точки требуется ключ API. Пример ввода приведен ниже:

```js
{
  "version": 1,
  "sender": "3FjTpAg1VbmxSH39YWnfFukAUhxMqmKqTEZ",
  "assetId":"AP5dp4LsmdU7dKHDcgm6kcWmeaqzWi2pXyemrn4yTzfo",
  "minSponsoredAssetFee": 100000,
  "fee": 100000000
}
```

`POST /asset/sponsor` подписывает и отправляет транзакцию отмены спонсорства. Для этой конечной точки требуется ключ API. Пример ввода приведен ниже:

```js
{
  "version": 1,
  "sender": "3FjTpAg1VbmxSH39YWnfFukAUhxMqmKqTEZ",
  "assetId":"AP5dp4LsmdU7dKHDcgm6kcWmeaqzWi2pXyemrn4yTzfo",
  "minSponsoredAssetFee": null,
  "fee": 100000000
}
```

Информация про спонсорские ассеты предаталена в статье [описание ассетов](/en/waves-node/node-api/asset-transactions/public-functions#get-assetsdetailsassetid).

## Ограничения

Спонсировать актив может только эмитент.

## Связанные изменения

Минимальная комиссия была перенесена в консенсус.
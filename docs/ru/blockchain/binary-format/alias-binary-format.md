# Бинарный формат псевдонима

> Узнать больше о [псевдониме](/ru/blockchain/account/alias)

| Порядковый номер поля | Поле | Тип поля | Размер поля в байтах | Комментарии |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Тип сущности | Байт | 1 | Значение должно равняться 2 |
| 2 | [Байт сети](/ru/blockchain/blockchain-network/chain-id) | Байт | 1 | Значение равно:<br>84 — для [тестовой сети](/ru/blockchain/blockchain-network/test-network)<br>87 — для [основной сети](/ru/blockchain/blockchain-network/main-network)<br>83 — для [экспериментальной сети](/ru/blockchain/blockchain-network/stage-network) |
| 3 | Число символов в псевдониме | Короткое целое | 2 | |
| 4 | Псевдоним | Массив байтов	| От 4 до 30 включительно | | |

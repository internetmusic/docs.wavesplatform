# WriteSet (for Standard Library version 3)

> :warning: The structure is disabled in Standard library version 4. Use `BinaryEntry`, `BooleanEntry`, `IntegerEntry`, and `StringEntry` directly, see the [Callable function](/en/ride/functions/callable-function) section.

Structure of a list of data records of an [account data storage](/en/blockchain/account/account-data-storage).

## Constructor

``` ride
WriteSet(data: List[DataEntry])
```

## Fields

|   #   | Name | Data type | Description |
| :--- | :--- | :--- | :--- |
| 1 | data | [List](/en/ride/data-types/list)[[DataEntry](/en/ride/structures/common-structures/data-entry)] | List of data records of an account data storage |

# Features

**Feature** is a new functionality added to the [Waves blockchain](/en/blockchain/blockchain) during a new [release](https://github.com/wavesplatform/Waves/releases).

Each feature has a name and a unique feature ID.

## Feature Status

Each feature can be in one of three statuses:

1. Voting
2. Approved
3. Activated

## Activation Process of New Features

When some new feature is developed and released, it must be activated.

* Activation process consists of "voting" period and "activation" period. Each mining node (which generates blocks) can set in its configuration a parameter to vote for this new feature. After that all blocks generated by such a node contain the vote for the feature.

* Every 3000 blocks on testnet (10000 blocks on mainnet) - it's an "election period". If there are not less than 2700 blocks on testnet (8000 blocks on mainnet) with support of the feature during the election period, the feature becomes "voted". After it's "voted" there is a period of "activation" - 3000 blocks for testnet (10000 blocks for mainnet). After they all passed, the feature start working. At this moment all nodes which can not supports such feature (old versions) stops.

Features are getting different statuses according to the [feature activation protocol](/en/waves-node/features/feature-activation-protocol).
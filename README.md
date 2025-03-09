![images (23)](https://github.com/user-attachments/assets/438f0a1a-6a19-49e4-a9e3-121eb07cca95)
![Screenshot_20250202-215641_Telegram](https://github.com/user-attachments/assets/8de6cca9-158b-4c23-8a24-514c6f1d4a47)
![1500x500](https://github.com/user-attachments/assets/9884c44f-896d-4a73-940b-cf152ad5a980)
![ton-telegram](https://github.com/user-attachments/assets/004eb431-b2ba-4440-a7db-642b0f568542)
![Screenshot_20250224-220411_Gallery](https://github.com/user-attachments/assets/0554fdf6-a23b-4488-bbbf-456286ef9214)
![Screenshot_20250225-180206_Telegram](https://github.com/user-attachments/assets/eb71ca02-3b6c-4b77-9e56-a4ca16a1aeca)
![Screenshot_20250226-155657_One UI Home](https://github.com/user-attachments/assets/026c4922-0f8a-4e20-86d9-02ef3dded925)
![Screenshot_20250226-190848_Chrome](https://github.com/user-attachments/assets/a061b826-2e51-4919-9f3b-80b826a80fc0)
![Screenshot_20250308-212956_Telegram](https://github.com/user-attachments/assets/0cdbb8ef-c296-42a5-8ea6-aff564c15c57)
![6c601c01bbd9459ab7d0e8c808634791](https://github.com/user-attachments/assets/2a5e1ed5-498d-4c02-8da4-dc9e141dbef6)
# W5: wallet smart contract v5

New version of wallet smart contract, the previous one was [v4r2](https://github.com/ton-blockchain/wallet-contract).

The entire concept is proposed by the [Tonkeeper team](https://tonkeeper.com/).

New Features:

- Send up to 255 messages at once;

- Signed actions can be sent not only by external message, but also by internal messages (can be used for gasless transactions);

- Unlimited extensions;

- Extension can prohibit signed actions in the wallet (can be used for 2fa or key recovery);

- Optimizations to reduce network fees;

- Better foolproofing safety - reply-protection for external messages, wallet id rethinking;

## Project structure

-   [Specification](Specification.md)
-   `contracts` - source code of all the smart contracts of the project and their dependencies.
-   `wrappers` - wrapper classes (implementing `Contract` from ton-core) for the contracts, including any [de]serialization primitives and compilation functions.
-   `tests` - tests for the contracts.
-   `scripts` - scripts used by the project, mainly the deployment scripts, additionally contains utilities for gas optimisation.
-   `fift` - contains standard Fift v0.4.4 library including the assembler and disassembler for gas optimisation utilities.

## How to use

### Build

`npm run build`

### Test

`npm run test`

### Deployment

Deploy wallet: `npm run deploy-wallet`

### Known issues

1) Since the `valid_until` is uint32 it will not work after 2106 year. We believe new versions of wallet smart contract will be available by then.

2) If the `action_send_msg` content is invalid and the sendmode has +2, the error will not be ignored. An update of the node is planned where this behaviour will be changed (with +2 sendmode and `action_send_msg` invalid content the error will be ignored).

3) It would be good to do `end_parse()` for messages and contract data. But this is not done within optimisations.

### Gasless flow

1. When sending an USDt (or other Jetton) the user signs one message containing two outgoing USDt transfers:

     * USDt transfer to the recipient's address.

     * Transfer of a small amount of USDt in favor of the Service.

2. This signed message is sent offchain by HTTPS to the Service backend. The Service backend checks message and sends it to the TON blockchain paying Toncoins for network fees.

### Gasless known issues

1) By requesting a gasless service, a user can have time to increase the seqno on his own, or via another service. 

    In this case, the gasless service will incur gas costs. 

    However, this is a non-scalable scenario, as it requires the user to incur gas costs as well. 

    A blacklist on the service backend side solves the problem.

2) The user can request a gasless service and by means of a specialised extension have time to withdraw the entire balance of Jettons without change seqno.

    In this case, the Jetton transfer message from the service will encounter a balance shortage and the Toncoins attached to message will return to the user's wallet.

    However, this is a non-scalable scenario, as it requires the user to incur gas costs as well. 

    A blacklist on the service backend side solves the problem.

### Suggested extensions

1) Decentralised subscriptions. The extension can withdraw a given number of Toncoins or Jettons once in a given period.

2) 2FA: Multisig extension is added, extension prohibits wallet signature;

3) Key recovery: 2FA, but in multisig extension there is an option to change the control keys. Possible cooldown period when the other party can cancel the key change.

4) Key compromise: An extension with a new key is added, extension prohibits wallet signature;


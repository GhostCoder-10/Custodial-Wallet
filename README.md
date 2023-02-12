## Custodial Wallet

### About
Our Wallet is a self-custodial crypto wallet designed with power and ease of use in mind. Unlike most crypto wallets, this focuses on user experience and human-friendliness, while not compromising on features. this is unopinionated, it can be connected to any dApp and it supports most of the popular EVM networks. This wallet is also a Web3 superapp: you can swap, lend, borrow, perform cross-chain transfers, deposit FIAT, all without the app.

It's built on smart contract wallet technology, enabling powerful features such as transaction batching, account recovery, multisigs, key rotation and paying for transactions in stablecoins (gas abstractions).

## Running

**NOTE: we test on 5ire,** because it's cheap enough and it's a real environment with all the supported protocols.


### Running the wallet

Then run the Ambire Wallet:
```
npm i
npm start
```


### Testing Ledger

**Important:** to make the Ledger integration work, you need to be accessing Wallet through HTTPS. The easiest way to do this in a development environment is to [use localtunnel](https://github.com/localtunnel/localtunnel): for example, `lt --port 3000`

## Building plugins

To see how to build plugins for Ambire, please [read our plugin docs](/how-to-create-a-plugin.md).

## Code style and recommendations

* No semicolons
* 2 spaces for identation
* Single quote (') instead of double (")
* Error handling: make sure to catch all errors that may originate in external IO (expected errors) and display them in a human friendly way with `addToast`; also, at a top-level, every time you spawn an async operation, make sure you `.catch` the entire thing to catch unexpected errors
* Camel case

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will reload if you make edits.\
You will also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can’t go back!**

If you aren’t satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you’re on your own.

You don’t have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn’t feel obligated to use this feature. However we understand that this tool wouldn’t be useful if you couldn’t customize it when you are ready for it.

## UX decisions

### Terms
* Signer: the signer is an address used to sign transactions and messages. It's normally an EOA (externally owned address) such as a Trezor address, Ledger address, Metamask address, a double-keypair representing an email/passphrase authentication, or even another smart wallet address (eg Gnosis Safe). We use this term to distinct it from "account", which is the actual smart wallet account, which can have one or more signers.


### Multi-account behavior
* When adding an account with Trezor, Ledger or a web3 wallet, we create one automatically if it doesn't exist; if we control multiple, we add all those accounts and show a toast notification "this key controls N accounts"
* We also auto-add all accounts FORMERLY controlled by the connected signer
* When trying to sign a transaction/message with an added account, and the signer is not available, we show a toast notification explaining to the user they have to connect the relevant signer; unless it's a HW Wallet, in which case we can just prompt the user to connect it
* When an account is added with a signer, but that signer no longer controls it, we should warn the user: "You are currently not authorized to send transactions from <> on <network>. Please re-add the account with a key that controls it."
* If someone goes through "add account"/"email login" but the given account is already added, we won't attempt to warn them early; the reason for this is 1) for simplicity and 2) cause re-adding an acc will change to the latest used signer (eg former quickacc, readding as trezor-controlled) 3) cause re-adding an account might reset it's state and fix tech issues in the future


## Internal data formats

#### Account
```
{
	id, // address (checksummed) of the account itself
	signer, // object, either { address } or { quickAccManager, one, two, timelock }
        salt, identityFactoryAddr, baseIdentityAddr, bytecode // all hex strings, account identity deploy data; all required
	email, // optional: only in case of quick accounts
        primaryKeyBackup, // optional, only in case of quick accounts, stringified JSON in a keystore format
}
```

#### Signing request

This is used by the WalletConnect and Gnosis Safe Apps hooks for the queue of signing requests: those could be transactions, personal messages, etc.

```
{
	id, // numeric unique ID of the request
	type, // type of the signing request, currently set to the RPC method (eg eth_sendTransaction)
	txn, // only set when it's eth_sendTransaction, contains to/data/value/gas
	chainId, // chainId the request is for
	account, // account address the request is for
}
```

`resolveMany` response:
```
{
	success, // boolean
	message, // string, optional, if success is false
	result, // string or object, optional, if success is true, depending on the request; normally a string, eg eth_sendTransaction would be answered with a hex hash (0x...)
}
```

## Deployed contracts
* Factory: 0xBf07a0Df119Ca234634588fbDb5625594E2a5BCA
* Base identity: 0x2A2b85EB1054d6f0c6c2E37dA05eD3E5feA684EF
* QuickAccManager: 0xfF3f6D14DF43c112aB98834Ee1F82083E07c26BF
* Batcher: 0x460fad03099f67391d84c9cc0ea7aa2457969cea
* WALLET token: 0x88800092ff476844f74dc2fc427974bbee2794ae
* xWALLET staking: 0x47Cd7E91C3CBaAF266369fe8518345fc4FC12935
* SupplyController: 0x6FDb43bca2D8fe6284242d92620156205d4fA028

Those contracts (except Ethereum-specific WALLET, xWALLET and SupplyController) are deployed cross-chain on the same addresses across Ethereum, Polygon, BSC, Fantom, Avalanche, Arbitrum, Moonbeam, Moonriver, Cronos, Metis, Gnosis Chain (formerly xDAI), NEAR Aurora

## Change log

### First private beta (v0.1.0)

* Email/password accounts
* Ledger support
* Trezor support
* Can create an account with a Web3 wallet
* Multiple accounts
* Multiple networks: Ethereum, Polygon, Avalanche
* WalletConnect v1
* WalletConnect: multi-dApp support
* Transaction preview: transactions parsed to display meaning
* Transaction batching: add multiple transactions to automatically batch them
* Notifications for pending to sign, ability to click notifications to go directly to the tab
* Deposit page
* Deposit through on-ramps
* Transfer page: send tokens
* Dashboard: display all tokens you own with their $ values
* Dashboard: display all NFTs you own
* Plugin system: iframe-based plugins supported
* Swap: ability to swap tokens through an embedded Sushiswap plugin
* Earn: ability to easily deposit/withdraw to and from Aave
* Security: add/remove authorized signers
* Relayerless mode: ability to function without being connected to the relayer
* Fee payment in stablecoins: auto-detects what tokens you have left after your transaction batch
* Transactions: list all the transaction history
* OTP (Authenticator) support for extra security with email/pass accounts


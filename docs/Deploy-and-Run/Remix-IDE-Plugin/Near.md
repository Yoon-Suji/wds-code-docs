---
sidebar_position: 1
---

## Choose a wallet
In order to start using the `multichain-builder` you need to install WELLDONE Wallet

![Select Wallet](img/select-wallet.png?raw=true "Select Wallet")

`multichain-builder` will support NEAR wallet soon.

## Connect
Before connecting `WELLDONE wallet`, please create a NEAR account in `WELLDONE wallet`.

![Connect Wallet](img/connect-wallet.png?raw=true "Connect Wallet")

## Create Project

Generate sample contract code templates written in AssemblyScript or Rust. The source of these templates is [NEAR Examples](https://examples.near.org/).

![Template Tree](img/template-tree.png?raw=true "Template Tree")

Or, if you want a new project, you can create it by clicking the New Project button.

## Compile

- Select the project to compile in `PROJECT TO COMPILE`.
- Select the language of the code to compile.
- Click the Compile button.
- When compilation is complete, a wasm file is returned.

![Project Compile](img/project-compile.png?raw=true "Project Compile")

## Deploy

- If there is a compiled contract code(.wasm), enter a value for `receive_id` and click the deploy button.
- Use your wallet to sign the transaction.

![Deploy](img/deploy.png?raw=true "Deploy")

- Once the contract is deployed, you can run functions.

![Deployed Contract](img/deployed-contract.png?raw=true "Deployed Contract")

## View and Call function

- You can call an existing deployed contract by entering the contract id in `At Address`.
- Select the method you want to call.
- Add parameters as needed.
- Read values ​​or send transactions through `View` or `Call` .

![View Function](img/view-function.png?raw=true "View Function")


![Function Call](img/function-call.png?raw=true "Function Call")



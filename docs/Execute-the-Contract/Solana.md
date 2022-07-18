## Params

```typescript
type serializedTransaction = string;
```

solana에서 트랜젝션을 보내기 위해선 `serializedTransaction`을 params로 넘겨야 합니다. 해당 값은 `@solana/web3.js` 라이브러리를 통해 얻을 수 있으며, 자세한 사용 방식은 아래 예시를 통해 이해할 수 있습니다.

## Example

### get_count

```jsx live
function sendTransaction() {
  const CHAIN_NAME = "solana";
  const programId = new PublicKey("FGbjtxeYmT5jUP7aNavo9k9mQ3rGQ815WdvwWndR7FF9"); // 배포한 프로그램 Id
  const COUNTER_SEED = "welldone";
  const [accounts, setAccounts] = React.useState(null);
  const [txHash, setTxHash] = React.useState(null);
  const getSerializedTransaction = async () => {
    try {
      const solana = new Connection(clusterApiUrl("devnet"), "confirmed");
      const fromPubkey = new PublicKey(accounts);
      const { blockhash } = await solana.getLatestBlockhash();

      const counterPubkey = await PublicKey.createWithSeed(
        fromPubkey,
        COUNTER_SEED,
        programId
      );

      console.log(counterPubkey.toBase58());

      // make a transaction
      const instruction = new TransactionInstruction({
        keys: [{ pubkey: counterPubkey, isSigner: false, isWritable: true }],
        programId,
        data: Buffer.alloc(0)
      });
      console.log(instruction);
      const transaction = new Transaction().add(instruction);

      // return serialized transaction
      return transaction.serialize({ verifySignatures: false }).toString("hex");
    } catch (error) {
      /* error */
      console.log(error);
    }
  };
  async function handleGetAccount() {
    try {
      const accounts = await dapp.request(CHAIN_NAME, {
        method: "dapp:accounts"
      });
      setAccounts(accounts[CHAIN_NAME].address);
    } catch (error) {
      alert(error.message);
    }
  }
  async function handleSendTransaction() {
    try {
      const serializedTransaction = await getSerializedTransaction();
      const response = await dapp.request(CHAIN_NAME, {
        method: "dapp:sendTransaction",
        params: [`0x${serializedTransaction}`]
      });
      const txHash = response;

      setTxHash(txHash);
    } catch (error) {
      console.log(error);
      alert(`Error Message: ${error.message}\nError Code: ${error.code}`);
    }
  }
  return (
    <>
      {accounts ? (
        <>
          <Button onClick={handleSendTransaction} type="button">
            Send a Transaction
          </Button>
          <ResultTooltip style={{ background: "#3B48DF" }}>
            <b>Accounts:</b> {accounts}
          </ResultTooltip>
        </>
      ) : (
        <>
          <Button onClick={handleGetAccount} type="button">
            Get Account
          </Button>
          <div>You have to get account first!</div>
        </>
      )}
      {txHash && (
        <ResultTooltip style={{ background: "#F08080" }}>
          <b>Transaction Hash:</b> {txHash}
        </ResultTooltip>
      )}
    </>
  );
}
```

### increment

### reset

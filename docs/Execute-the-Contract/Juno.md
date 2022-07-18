:::note
Juno는 Cosmos SDK로 만들어진 상호 운용이 가능한 스마트 컨트랙트 네트워크입니다. Juno에 대한 자세한 설명은 공식 [docs](https://docs.junonetwork.io/juno/readme) 에서 확인할 수 있습니다.
:::
## Params
```
interface TransactionParameters {
  signerData: {
    accountNumber: string;
    sequence: string;
    chainId: string;
  };
  fee: {
    amount: [
      {
        denom: string;
        amount: string;
      },
    ];
    gas: string;
  };
  memo: string;
  msgs: [
    {
      typeUrl: string;
      value: {

      };
    },
  ];
  sequence: string;
}
```
트랜잭션에는 컨트랙트 내부의 상태를 변경하는 트랜잭션과, 컨트랙트 내부의 상태 변경 없이 상태를 가져오기만 하는 트랜잭션 두 종류가 있습니다. 전자의 경우에는 인자의 `typeUrl` 에 `'/cosmwasm.wasm.v1.MsgExecuteContract'`이 들어가고, 후자의 경우에는 `/cosmwasm.wasm.v1.QuerySmartContractState` 가 들어갑니다. 또한 `typeUrl`의 종류에 따라서 그리고 실행하고자 하는 컨트랙트의 함수 종류에 따라서 `value`의 인자값도 달라집니다. 

## Example
Juno Testnet (`uni-3`)에 미리 배포한 카운터 컨트랙트와 통신하는 예제를 같이  살펴보겠습니다. 컨트랙트 내부의 상태를 변경하는 트랜잭션을 실행하기 위해서는 가스 비용을 지불해야 하므로 다음 링크를 참고하여 미리 [faucet](https://docs.junonetwork.io/validators/joining-the-testnets#get-some-testnet-tokens)을 요청해주세요.
### get_count
`get_count` 메소드는 카운터 값을 읽어와서 보여주는 것으로 컨트랙트 내부의 상태 변경 없이 상태를 가져오기만 하는 트랜잭션이기 때문에 `typeUrl`에 `/cosmwasm.wasm.v1.QuerySmartContractState`가 들어가는 것을 확인할 수 있습니다. 
먼저 지갑에 연결을 요청한 후에, `get_count` 메소드를 실행시키는 트랜잭션을 보내고 카운터 값을 출력한다면 성공입니다.

```jsx live 
function sendTransaction() {
  const CHAIN_NAME = 'juno';
  const sequence = '10';
  const chainId = 'uni-3';
  const contractAddress = "juno1yfp9zyx9zhqe77d05yqjx3ctqjhzha0xn5d9x8zxcpp658ks2hvqlfjt72";
  const [accounts, setAccounts] = React.useState(null);
  const [txHash, setTxHash] = React.useState(null);
  // connect to wallet
  async function handleGetAccount() {
    try {
      const accounts = await dapp.request(CHAIN_NAME, {
        method: 'dapp:accounts',
      });
      setAccounts(accounts[CHAIN_NAME].address);
    } catch (error) {
      alert(error.message);
    }
  }
  // execute the contract
  async function handleSendTransaction() {
    try {
      const transactionParameters = {
        signerData: {
          accountNumber: accounts,
          sequence,
          chainId,
        },
        fee: {
          amount: [
            {
              denom: 'ujunox',
              amount: '10000',
            },
          ],
          gas: '180000', // 180k
        },
        memo: '',
        msgs: [
          {
            typeUrl: '/cosmwasm.wasm.v1.QuerySmartContractState',
            value: MsgExecuteContract.fromPartial({
                sender: accounts,
                contract: contractAddress,
                msg: toUtf8(JSON.stringfy({
                    "get_count" : {}
                }))
            }),
          },
        ],
        sequence: `${sequence}`,
      };
      const response = await dapp.request(CHAIN_NAME, {
        method: 'dapp:sendTransaction',
        params: [JSON.stringify(transactionParameters)],
      });
      const txHash = response.transactionHash;

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
          <ResultTooltip style={{ background: '#3B48DF' }}>
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
        <ResultTooltip style={{ background: '#F08080' }}>
          <b>Transaction Hash:</b> {txHash}
        </ResultTooltip>
      )}
    </>
  );
}
```

### increment
`increment` 메소드는 컨트랙트 내부의 카운터 값에 인자로 전달한 count 값 만큼 더해주는 메소드로, 컨트랙트 내부의 상태를 변화시키기 때문에 `typeUrl`에 `'/cosmwasm.wasm.v1.MsgExecuteContract'`이 들어가는 것을 확인할 수 있습니다.
먼저 지갑에 연결을 요청한 후에, `increment` 메소드를 실행하는 트랜잭션을 보내고 트랜잭션 해시값을 출력한다면 성공입니다. 
```jsx live 
function sendTransaction() {
  const CHAIN_NAME = 'juno';
  const sequence = '10';
  const chainId = 'uni-3';
  const contractAddress = "juno1yfp9zyx9zhqe77d05yqjx3ctqjhzha0xn5d9x8zxcpp658ks2hvqlfjt72";
  const [accounts, setAccounts] = React.useState(null);
  const [txHash, setTxHash] = React.useState(null);
  // connect to wallet
  async function handleGetAccount() {
    try {
      const accounts = await dapp.request(CHAIN_NAME, {
        method: 'dapp:accounts',
      });
      setAccounts(accounts[CHAIN_NAME].address);
    } catch (error) {
      alert(error.message);
    }
  }
  // execute the contract
  async function handleSendTransaction() {
    try {
      const transactionParameters = {
        signerData: {
          accountNumber: accounts,
          sequence,
          chainId,
        },
        fee: {
          amount: [
            {
              denom: 'ujunox',
              amount: '10000',
            },
          ],
          gas: '180000', // 180k
        },
        memo: '',
        msgs: [
          {
            typeUrl: '/cosmwasm.wasm.v1.MsgExecuteContract',
            value: MsgExecuteContract.fromPartial({
                sender: accounts,
                contract: contractAddress,
                msg: toUtf8(JSON.stringfy({
                    "increment" : {
                        "count": 1
                    }
                }))
            }),
          },
        ],
        sequence: `${sequence}`,
      };
      const response = await dapp.request(CHAIN_NAME, {
        method: 'dapp:sendTransaction',
        params: [JSON.stringify(transactionParameters)],
      });
      const txHash = response.transactionHash;

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
          <ResultTooltip style={{ background: '#3B48DF' }}>
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
        <ResultTooltip style={{ background: '#F08080' }}>
          <b>Transaction Hash:</b> {txHash}
        </ResultTooltip>
      )}
    </>
  );
}
```

### reset
`reset` 메소드는 컨트랙트 내부의 카운터 값을 인자로 전달한 count 값으로 초기화하는 메소드로, 컨트랙트 내부의 상태를 변화시키기 때문에 `typeUrl`에 `'/cosmwasm.wasm.v1.MsgExecuteContract'`이 들어가는 것을 확인할 수 있습니다.
먼저 지갑에 연결을 요청한 후에, `reset` 메소드를 실행하는 트랜잭션을 보내고 트랜잭션 해시값을 출력한다면 성공입니다. 
```jsx live 
function sendTransaction() {
  const CHAIN_NAME = 'juno';
  const sequence = '10';
  const chainId = 'uni-3';
  const contractAddress = "juno1yfp9zyx9zhqe77d05yqjx3ctqjhzha0xn5d9x8zxcpp658ks2hvqlfjt72";
  const [accounts, setAccounts] = React.useState(null);
  const [txHash, setTxHash] = React.useState(null);
  // connect to wallet
  async function handleGetAccount() {
    try {
      const accounts = await dapp.request(CHAIN_NAME, {
        method: 'dapp:accounts',
      });
      setAccounts(accounts[CHAIN_NAME].address);
    } catch (error) {
      alert(error.message);
    }
  }
  // execute the contract
  async function handleSendTransaction() {
    try {
      const transactionParameters = {
        signerData: {
          accountNumber: accounts,
          sequence,
          chainId,
        },
        fee: {
          amount: [
            {
              denom: 'ujunox',
              amount: '10000',
            },
          ],
          gas: '180000', // 180k
        },
        memo: '',
        msgs: [
          {
            typeUrl: '/cosmwasm.wasm.v1.MsgExecuteContract',
            value: MsgExecuteContract.fromPartial({
                sender: accounts,
                contract: contractAddress,
                msg: toUtf8(JSON.stringfy({
                    "reset" : {
                        "count": 0
                    }
                }))
            }),
          },
        ],
        sequence: `${sequence}`,
      };
      const response = await dapp.request(CHAIN_NAME, {
        method: 'dapp:sendTransaction',
        params: [JSON.stringify(transactionParameters)],
      });
      const txHash = response.transactionHash;

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
          <ResultTooltip style={{ background: '#3B48DF' }}>
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
        <ResultTooltip style={{ background: '#F08080' }}>
          <b>Transaction Hash:</b> {txHash}
        </ResultTooltip>
      )}
    </>
  );
}
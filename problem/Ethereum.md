# **Ethereum**
## **1、token合约 transfer 和 transferfrom 区别。**

同事在测试网发布token合约如下：

[ERC-20合约](https://github.com/ConsenSys/Tokens/blob/fdf687c69d998266a95f15216b1955a4965a0a6d/contracts/eip20/EIP20.sol)

测试时有a、b两个账户（两个账户都有token和eth），发现可以a可以调用 transfer 向b发送代币。但a调用 transferfrom(a,b,money) 时，合约报错，无法转账成功。我排查了一遍，发现问题在于 transferfrom 方法上。代码如下：

```js
function transferFrom(address _from, address _to, uint256 _value)
    public returns (bool success) {

        uint256 allowance = allowed[_from][msg.sender];

        require(balances[_from] >= _value && allowance >= _value);

        balances[_to] += _value;

        balances[_from] -= _value;

        if (allowance < MAX_UINT256) {

            allowed[_from][msg.sender] -= _value;

        }

        emit Transfer(_from, _to, _value);
         //solhint-disable-line indent, no-unused-vars

        return true;
}
```

注意到 require 中对 allowance 作了检查。而 allowed 这个 mapping 是由 approve 方法赋值的，可以通过 allowance 方法查看 owner 给 spender 的额度。虽然 a 调用了 transferFrom(a,b,money) ， 但a没有事先给自己使用token的限额，allowance中没有<a,<a,money>>的记录，故无法通过 transferFrom 转账。但 transfer 直接检查发送者 token 是否充足，所以可以转账成功。实际上 EIP-20 上有说明，参见[eip-20.md](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md). 整个合约的转账分两种：
* 一种是 transfer 类的直接转账，即交易发起者即转账人，只需填写收款人和数量即可
* 另一种是 transferFrom 类的转账，类似于代付机制。a 先给 b 一部分额度可以供 b 使用， b 调用 transferFrom 方法让 a 代替 b 付款。可以类比信用卡的支付模式。

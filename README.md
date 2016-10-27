# 使用以太坊创造你自己的加密货币

## 数字货币
我们将要创造一种数字代币。在以太坊的生态系统中,代币可以代表任何可以交换的商品:币,忠诚点数,黄金证书,借据,游戏道具等。而且因为所有的代币都标准化继承一些基本属性, 这也就意味着你的代币可以立即在以太坊钱包和其他客户端(或者其他使用共同标准的智能合约)中共存。

### 程序代码
如果你只想复制粘贴代码,可以使用这个:
```
pragma solidity ^0.4.2;
contract tokenRecipient { function receiveApproval(address _from, uint256 _value, address _token, bytes _extraData); }

contract MyToken {
    /* Public variables of the token */
    /* 代币的公有变量 */
    string public standard = 'Token 0.1';
    string public name;
    string public symbol;
    uint8 public decimals;
    uint256 public totalSupply;

    /* This creates an array with all balances */
    /* 余额数组 */
    mapping (address => uint256) public balanceOf;
    mapping (address => mapping (address => uint256)) public allowance;

    /* This generates a public event on the blockchain that will notify clients */
    /* 在区块链上创建了一个公开的事件,事件触发时他将通知客户端 */
    event Transfer(address indexed from, address indexed to, uint256 value);

    /* Initializes contract with initial supply tokens to the creator of the contract */
    /* 初始化合约,同时给合约的创建者一些初始代币 */
    function MyToken(
        uint256 initialSupply,
        string tokenName,
        uint8 decimalUnits,
        string tokenSymbol
        ) {
        balanceOf[msg.sender] = initialSupply;              // Give the creator all initial tokens 给创建者所有的初始代币
        totalSupply = initialSupply;                        // Update total supply 更新发行总量
        name = tokenName;                                   // Set the name for display purposes 设置代币的名称
        symbol = tokenSymbol;                               // Set the symbol for display purposes 设置代币的标识
        decimals = decimalUnits;                            // Amount of decimals for display purposes 展示的小数点
    }

    /* Send coins 发送币*/
    function transfer(address _to, uint256 _value) {
        if (balanceOf[msg.sender] < _value) throw;           // Check if the sender has enough 检查发送者是否拥有足够余额
        if (balanceOf[_to] + _value < balanceOf[_to]) throw; // Check for overflows 检查是否溢出
        balanceOf[msg.sender] -= _value;                     // Subtract from the sender 从发送者减掉发送额
        balanceOf[_to] += _value;                            // Add the same to the recipient 给接收者加上相同的量
        Transfer(msg.sender, _to, _value);                   // Notify anyone listening that this transfer took place 通知任何监听该交易的客户端
    }

    /* Allow another contract to spend some tokens in your behalf 允许另外的合约花费代币*/
    function approve(address _spender, uint256 _value)
        returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        return true;
    }

    /* Approve and then comunicate the approved contract in a single tx 批准然后和被批准的合约通信*/
    function approveAndCall(address _spender, uint256 _value, bytes _extraData)
        returns (bool success) {
        tokenRecipient spender = tokenRecipient(_spender);
        if (approve(_spender, _value)) {
            spender.receiveApproval(msg.sender, _value, this, _extraData);
            return true;
        }
    }

    /* A contract attempts to get the coins 货币转移*/
    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {
        if (balanceOf[_from] < _value) throw;                 // Check if the sender has enough
        if (balanceOf[_to] + _value < balanceOf[_to]) throw;  // Check for overflows
        if (_value > allowance[_from][msg.sender]) throw;   // Check allowance
        balanceOf[_from] -= _value;                          // Subtract from the sender
        balanceOf[_to] += _value;                            // Add the same to the recipient
        allowance[_from][msg.sender] -= _value;
        Transfer(_from, _to, _value);
        return true;
    }

    /* This unnamed function is called whenever someone tries to send ether to it */
    function () {
        throw;     // Prevents accidental sending of ether
    }
}
```
### 最小化可运行代币
上面的代币合约有些复杂。但是本质上他是很简单的
```
contract MyToken {
    /* This creates an array with all balances */
    mapping (address => uint256) public balanceOf;

    /* Initializes contract with initial supply tokens to the creator of the contract */
    function MyToken(
        uint256 initialSupply
        ) {
        balanceOf[msg.sender] = initialSupply;              // Give the creator all initial tokens
    }

    /* Send coins */
    function transfer(address _to, uint256 _value) {
        if (balanceOf[msg.sender] < _value) throw;           // Check if the sender has enough
        if (balanceOf[_to] + _value < balanceOf[_to]) throw; // Check for overflows
        balanceOf[msg.sender] -= _value;                     // Subtract from the sender
        balanceOf[_to] += _value;                            // Add the same to the recipient
    }
}
```
### 理解合约代码
![](https://ethereum.org/images/tutorial/deploy-new-contract.png)
我们从最简单的开始,打开钱包app,进入`CONTRACTS`,然后点击`DELPLOY NEW CONTRACT`。在*SOLIDITY CONTRACT SOURCE CODE*文本输入框中,写入以下代码:
```
contract MyToken {
    /* This creates an array with all balances */
    mapping (address => uint256) public balanceOf;
}
```
一个*mapping*代表一个地址与余额的映射数组。地址是16进制字符串,余额是整形(0 -- 115*quattuorvigintillion*)。如果你不知道*quattuorvigintillion*有多大,你只要知道比你想像中的大多了。*public*关键字, 意味着这个变量可以被区块链上的任何人访问,意味着所有的余额是公开的(为了在客户端展示他们,也必须这样)。
![](https://ethereum.org/images/tutorial/edit-contract.png)
如果你马上发布你的合约,他会启动但是没有什么用:它本来应该可以查询任何地址的余额,但是因为你没有发行任何一个coin,每一个地址的余额都将返回0. 因此我们将要创建一些代币在初创的时候。添加如下代码到*mapping...*之后,最后一个大括号之前:
```
function MyToken() {
    balanceOf[msg.sender] = 21000000;
}
```
注意到: 这个方法*MyToken*和合约拥有相同的名字。这是非常重要的,如果你重命名其中一个,你必须重命名另外一个:这是一个特殊的初始函数,他只会被执行一次,且是在首次被上传到区块链网络中时执行。这个方法将会设置*msg.sender(合约的发布者)*的余额为2100万。

直接在代码中写2100万是不太好的方式,虽然你可以把他改成任何你想要的数字。但是有更好的方法:把数字通过函数的参数传递进去,像这样:
```
function MyToken(uint256 initialSupply) {
    balanceOf[msg.sender] = initialSupply;
}
```
看图片的右边那一列,你会看到一个下拉选择框,写着*Pick a contract*。 选择合约*MyToken*,你将看到一个叫做*CONTRUCTOR PARAMETERS*的分区,这些是你的代币的参数。你可以通过改变这些变量来重用你的代码。
![](https://ethereum.org/images/tutorial/function-picker.png)

现在你有了一个合约,并且拥有了代币。但是因为没有方法去转移代币,所有的代币都只存在同一个账号下。所以我们马上实现一个转移代币的方法。将以下代码写入最后一个大括号前。
```
/* Send coins */
function transfer(address _to, uint256 _value) {
    /* Add and subtract new balances */
    balanceOf[msg.sender] -= _value;
    balanceOf[_to] += _value;
}
```
这是一个非常简单的方法:方法拥有2个参数,一个接收者和转移的值。无论何时,谁调用他,就会从调用者的账户上减去*_value*的代币,然后把它加到*_to*的余额里。现在,有一个很明显的问题:如果一个人想转移比他拥有的还多的代币呢?因为在这个合约里我们没有相关欠款的处理,我们就简单的做一个快速检测,如果发送者没有足够的余额,合约就终止执行。我们也检测溢出,避免数值过大之后又变成了0。

在合约执行期间终止合约的执行,我们可以使用**return**或者**throw**。前者不回滚操作,在终止执行前对合约的操作将表现在最终的状态里,只花费较少的gas。另一方面,*throw*将会回滚该交易所做的所有更改,但是会丢失所有的gas。但是因为以太坊客户端能探测到合约有异常抛出,一旦异常发生,客户端会弹出警告,因此可以防止ether被花费。
```
function transfer(address _to, uint256 _value) {
    /* Check if sender has balance and for overflows */
    if (balanceOf[msg.sender] < _value || balanceOf[_to] + _value < balanceOf[_to])
        throw;

    /* Add and subtract new balances */
    balanceOf[msg.sender] -= _value;
    balanceOf[_to] += _value;
}
```

现在只剩下一些关于合约的基本信息没有提及了。在之后,基本信息可以在代币登记的时候处理,但是现在我们先把他们直接加到合约里:
```
string public name;
string public symbol;
uint8 public decimals;
```
现在,我们更新**构造方法**,让这些变量在一开始就被设置:
```
/* Initializes contract with initial supply tokens to the creator of the contract */
function MyToken(uint256 initialSupply, string tokenName, uint8 decimalUnits, string tokenSymbol) {
    balanceOf[msg.sender] = initialSupply;              // Give the creator all initial tokens
    name = tokenName;                                   // Set the name for display purposes
    symbol = tokenSymbol;                               // Set the symbol for display purposes
    decimals = decimalUnits;                            // Amount of decimals for display purposes
}
```
最后我们来看**Events**。这些特殊的、空的方法可以帮助客户端,如以太坊钱包,追踪合约的活动状态。事件方法名称以大写字母开头。将下面一行代码加入合约的开头,用于声明事件:
```
event Transfer(address indexed from, address indexed to, uint256 value);
```
然后,只需要将下列代码加入*transfer*方法就可以通知客户端有更新了。
```
    /* Notify anyone listening that this transfer took place */
    Transfer(msg.sender, _to, _value);
```
现在你的代币已经准备好了!

### 如何部署
现在,打开以太坊钱包,去*CONTRACTS*栏,点击*DEPLOY NEW CONTRACT*。

现在复制代币源代码并粘贴到*SOLIDITY CONTRACT SOURCE CODE*文本输入框。如果代码编译没有问题,你应该可以在右边看见*SELECT CONTRACT TO DEPLOY*分区。选择合约*MyToken*,在下面,你将看到所有需要自定义的参数,你可以随意指定参数值。但是现在在这个例子里,我们推荐*_supply*值为10,000, 名字随意,*_symbol*为*%*,*_decimal*为2。如图所示:
![](https://ethereum.org/images/tutorial/Ethereum-Wallet-Screenshot-2015-12-03-at-3.50.36-PM-10.png)

拉到页面的最后,你可以看到系统评估的部署合约需要的花费,你可以自由选择你愿意支付的手续费。**任何没有被花费的*ether*都讲返还给你**,所以你可以不管这个值,直接默认就好。点击*DEPLOY*,输入你的账户密码,等待生成交易。
![](https://ethereum.org/images/tutorial/Ethereum-Wallet-Screenshot-2015-12-03-at-3.50.36-PM-11.png)

确认完发送交易,你会被重定向到*Send funds*栏, 点击名为*Etherbase(你的主账户)*的账户,不到一分钟时间,你应该可以看到你拥有100%你刚刚创建的代币。发送一些代币给你的朋友:选择*send*,选择你要发送哪种资产(ehter 或者你刚刚创建的代币), 将你朋友的代币地址填入*to*文本框,最后点击*send*。
![](https://ethereum.org/images/tutorial/Screen-Shot-2015-12-03-at-9.48.15-AM.png)

如果你给你的朋友发送了一些代币,在他们的钱包里看不到任何变化。这是因为钱包只追踪已经知道的代币,你必须手动添加代币地址。现在进入*CONTRACTS*栏,你应该可以看到你最近创建的合约的链接。点击链接,打开一个新界面,因为这是一个非常简单的合约,直接点击*COPY ADDRESS*,复制合约地址。

返回合约界面,选择*WATCH TOKEN*, 钱包会弹出一个界面,直接粘贴合约地址到文本框。名称,标识,和小数点应该自动的填充,但是如果没有你可以填任何值(只会在你的钱包中展示)。一旦你完成,你将自动接收该代币你拥有的余额,你也可以把他发送给其他人。
![](https://ethereum.org/images/tutorial/Screen-Shot-2015-12-03-at-9.44.42-AM.png)

现在你拥有了你自己的加密代币。可以被用于[价值交换](https://en.wikipedia.org/wiki/Local_currency),或者[工作时间追踪](https://en.wikipedia.org/wiki/Time-based_currency)或者其他项目。但是我们能否创造一种货币,其本身就拥有价值。

## 改善代币
你可以不写一句代码就部署你的加密代币。但是当你开始个性化代码的时候,真正牛逼的事情就发生了。接下来的几节,我们将会对上面的代币做更多的改造以便适应需求。

### 中心化的管理者
所有的去中心化应用默认是完全去中心化的,但是那不意味着不能拥有一些中心化的管理。或许你想要挖矿的功能,或许你想更多的人使用你的资产。你可以添加这些功能,但是只能在最开始部署合约之前添加,所以代币的拥有者在他们决定拥有该代币之前,就已经知道游戏规则。

为了以上功能,你需要一个资产中心化管理者。它可以是一个简单的账户,但是也可以是一个合约,任何创造代币的决定都要取决于该合约:如果是一个民主机构,那么将投票表决,或者只是一个限制代币拥有者权利的方法。

为了做到这一点,我们将要学习一个非常有用的功能:**继承**。继承允许合约读取父合约的属性,而不需要重新定义他们。这使得代码更简洁并更容易重用。添加一下合约到合约*MyToken*之前:
```
contract owned {
    address public owner;

    function owned() {
        owner = msg.sender;
    }

    modifier onlyOwner {
        if (msg.sender != owner) throw;
        _
    }

    function transferOwnership(address newOwner) onlyOwner {
        owner = newOwner;
    }
}
```
上面的代码创建了一个非常简单的合约,该合约没做其他什么事,只是简单的定义了一下方法,表示合约可以被*拥有*。现在只需要你自定义的合约继承该合约就可以了。
```
contract MyToken is owned {
    /* the rest of the contract as usual */
```

这意味着,现在*MyToken*内部所有的方法都可以访问变量*owner*和*modifier onlyOwner*,合约也得到了一个方法去转移拥有权。再者或许有需求需要在部署的时候设置合约的拥有者,你可以把这个功能加入构造方法:
```
function MyToken(
    uint256 initialSupply,
    string tokenName,
    uint8 decimalUnits,
    string tokenSymbol,
    address centralMinter
    ) {
    if(centralMinter != 0 ) owner = centralMinter;
```

### 中心化挖矿
假设你需要能够流通的货币。当你的代币代表一种真实资产(黄金或者法币)的时候,并且你希望用虚拟资产来代表真实资产。也有可能是代币持有人希望能在一定程度上控制代币的价格,想要在流通过程中发行或回收一些代币。

首先,我们需要添加存储发行总量的变量**totalSupply**,并且在构造方法中给它赋值。
```
contract MyToken {
    uint256 public totalSupply;

    function MyToken(...) {
        totalSupply = initialSupply;
        ...
    }
    ...
}
```

现在,我们增加一个新的方法,能够让合约的拥有者创造一些新的代币:
```
function mintToken(address target, uint256 mintedAmount) onlyOwner {
    balanceOf[target] += mintedAmount;
    totalSupply += mintedAmount;
    Transfer(0, owner, mintedAmount);
    Transfer(owner, target, mintedAmount);
}
```
注意在方法声明的最后的`onlyOwner`,代表这个方法在编译的时候会继承我们之前定义的**modifier onlyOwner**。这个方法的代码将会被插入到*modifier*方法的下划线声明处,意味着这个方法只有合约的拥有者可以调用。使用这样的方法你就可以创建更多可流通的代币。

### 冻结资产
基于你的应用场景,你可能需要监管功能以便你能控制谁可以/谁不可以使用你创建的代币合约。为了实现这样的功能,你可以添加一个参数,使得合约拥有者可以冻结/解冻资产。

在合约的任意地方添加以下的代码。你可以把他们放到任意地方,但是我们建议将mapping跟其他mapping放在一起,event跟其他event放在一起。
```
mapping (address => bool) public frozenAccount;
event FrozenFunds(address target, bool frozen);

function freezeAccount(address target, bool freeze) onlyOwner {
    frozenAccount[target] = freeze;
    FrozenFunds(target, freeze);
}
```
在最开始,所有的账号都是解冻的,但是合约拥有者可以任意冻结账号。但是现在,冻结资产没有任何用处,因为我们还没有将资产冻结检查加入转账方法:
```
function transfer(address _to, uint256 _value) {
    if (frozenAccount[msg.sender]) throw;
```
现在,任意被冻结的账号仍然拥有他们的资产,但是将不能转移他们。所有的账号默认都是解冻的除非主动冻结他们,但是你可以很容易的翻转整个操作,大家可以想象一下怎么做。

### 自动交易
目前为止,你的代币很实用,你也很相信他的价值。但是如果你想让**ether**(或其他代币)为你的代币进行背书,以便可以市场价自动化买卖代币,我们可以这么做。

首先,让我们设置买卖价格:
```
uint256 public sellPrice;
uint256 public buyPrice;

function setPrices(uint256 newSellPrice, uint256 newBuyPrice) onlyOwner {
    sellPrice = newSellPrice;
    buyPrice = newBuyPrice;
}
```
因为每一次价格的变动都会执行一个transaction并消耗一定的ether,只有价格不会经常变化的时候这个方法才是可以接受的。如果你想要一个持续浮动的价格,我们推荐你调查[标准数据源](https://github.com/ethereum/wiki/wiki/Standardized_Contract_APIs#data-feeds)

接下来是构造买卖方法:
```
function buy() returns (uint amount){
    amount = msg.value / buyPrice;                     // calculates the amount
    if (balanceOf[this] < amount) throw;               // checks if it has enough to sell
    balanceOf[msg.sender] += amount;                   // adds the amount to buyer's balance
    balanceOf[this] -= amount;                         // subtracts amount from seller's balance
    Transfer(this, msg.sender, amount);                // execute an event reflecting the change
    return amount;                                     // ends function and returns
}

function sell(uint amount) returns (uint revenue){
    if (balanceOf[msg.sender] < amount ) throw;        // checks if the sender has enough to sell
    balanceOf[this] += amount;                         // adds the amount to owner's balance
    balanceOf[msg.sender] -= amount;                   // subtracts the amount from seller's balance
    revenue = amount * sellPrice;
    if (!msg.sender.send(revenue)) {                   // sends ether to the seller: it's important
        throw;                                         // to do this last to prevent recursion attacks
    } else {
        Transfer(msg.sender, this, amount);             // executes an event reflecting on the change
        return revenue;                                 // ends function and returns
    }
}
```
注意,这不会创造新的代币只是改变合约拥有者的余额。这个合约可以保存自己代币和ether。合约的拥有者可以设置代币的价格或者在一定情况下创造新的代币。但是不能修改银行里的代币或ether。唯一能改变资产的就是通过买卖。

**注意**买卖的价格单位不是ether,而是*wei*,这是系统中最小的单位(就像美元里的美分,比特币里的聪)。1 ether = 1000000000000000000 wei。因此使用ether设置价格的时候,在最后加18个0。

当创建合约的时候,**发送足够多的ether作为代币的背书**,否则你的合约就是破产的,你的用户就不能够卖掉他们的代币。

之前的那个例子,当然,只描述了一个买家一个卖家的情况。一个更真实更有趣的合约将会允许市场中的任何人出任何价格,或者可能直接从一个指定的数据源获取价格。


### AUTOREFILL
每次在以太坊平台中生成一个交易,你都需要给区块的矿工支付一定的费用,用于计算合约执行结果的报酬。[未来或许不需要了](https://github.com/ethereum/EIPs/issues/28),现在手续费只能使用ether支付,因此你的代币的所有用户都需要ether。如果账户里的余额不足以支付手续费,交易就会暂停。但是在一些使用场景中,你可能不想让你的用户想到以太坊,区块链或者是怎么去获取ether, 一个可能的方法就是只要检测到用户余额过低,就自动填充用户余额。

为了做到这一点,首先你需要设置一个变量,标识余额下限,然后有一个方法改变他。如果你不知道值,设置成**0.5 finney**(0.005ether).
```
uint minBalanceForAccounts;

function setMinBalance(uint minimumBalanceInFinney) onlyOwner {
     minBalanceForAccounts = minimumBalanceInFinney * 1 finney;
}
```
接着,添加如下代码到方法**transfer**:
```
/* Send coins */
function transfer(address _to, uint256 _value) {
    ...
    if(msg.sender.balance<minBalanceForAccounts)
        sell((minBalanceForAccounts-msg.sender.balance)/sellPrice);
}
```

你也可以让接收方支付手续费:
```
/* Send coins */
function transfer(address _to, uint256 _value) {
    ...
    if(_to.balance<minBalanceForAccounts)
        _to.send(sell((minBalanceForAccounts-_to.balance)/sellPrice));
}
```

### 工作量证明
有几种方法可以让你的代币供应与数学公式对应起来。最简单的方法就是和ether*合并挖矿*,也就是说任何人只要在以太坊链上挖到了一个区块,也会获得你的合约的奖励。你可以使用[特殊关键字coinbase](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#block-and-transaction-properties)去指代找到该区块的矿工。
```
function giveBlockReward() {
    balanceOf[block.coinbase] += 1;
}
```

新增一个数学公式也是可能的,任何会数学计算的人都会赢得奖励。接下来的这个例子,你必须计算当前谜题的立方根并设置下一个谜题。
```
uint currentChallenge = 1; // Can you figure out the cubic root of this number?

function rewardMathGeniuses(uint answerToCurrentReward, uint nextChallenge) {
    if (answerToCurrentReward**3 != currentChallenge) throw; // If answer is wrong do not continue
    balanceOf[msg.sender] += 1;         // Reward the player
    currentChallenge = nextChallenge;   // Set the next challenge
}
```

当然,对一些人来说,手算立方根是困难的,但是使用计算器却又非常简单。所以这个游戏很容易被计算机破解。而且因为最后一个胜利者可以选择下一个谜题,他们可以选择他们知道的值,所以对其他玩家来说不太公平。对一个公平的系统来说,计算机必须经过复杂的计算才能得到结果,但又不是非常难验证结果。一个好的候选方案是使用hash猜谜,猜谜者必须持续求值的hash,直到找到一个值,他的hash小于一个指定的困难度。

这个过程由Adam Back在1997年首次在[Hashcash](https://en.wikipedia.org/wiki/Hashcash)提出,之后在2008年被中本聪引入比特币中,作为**工作量证明**.以太坊现在也使用的是这套算法,但是正计划从PoW迁移到[Casper](https://blog.ethereum.org/2015/12/28/understanding-serenity-part-2-casper/)

但是,你也可以创建自己的PoW:
```
bytes32 public currentChallenge;                         // The coin starts with a challenge
uint public timeOfLastProof;                             // Variable to keep track of when rewards were given
uint public difficulty = 10**32;                         // Difficulty starts reasonably low

function proofOfWork(uint nonce){
    bytes8 n = bytes8(sha3(nonce, currentChallenge));    // Generate a random hash based on input
    if (n < bytes8(difficulty)) throw;                   // Check if it's under the difficulty

    uint timeSinceLastProof = (now - timeOfLastProof);  // Calculate time since last reward was given
    if (timeSinceLastProof <  5 seconds) throw;         // Rewards cannot be given too quickly
    balanceOf[msg.sender] += timeSinceLastProof / 60 seconds;  // The reward to the winner grows by the minute

    difficulty = difficulty * 10 minutes / timeSinceLastProof + 1;  // Adjusts the difficulty

    timeOfLastProof = now;                              // Reset the counter
    currentChallenge = sha3(nonce, currentChallenge, block.blockhash(block.number-1));  // Save a hash that will be used as the next proof
}
```

同样的,修改**构造方法**:
```
    timeOfLastProof = now;
```

一旦合约部署在线,选择方法*proofOfWork*, 添加你最喜欢的数字作为**随机数**,并执行。如果确认窗口给出一个红色警告并写着*Data can't be execute*,返回选择另外一个数字直到交易可以继续:这个过程是随机的。如果你找到了你就会得到一定的奖励, 并且会调整困难度。

这个试图找到一个数值并获得奖励的过程就叫做*mining*:如果困难度增加,得到幸运数字会变得很困难,但是很容易去验证你找到的值是否正确。

## 完善后的合约

### 完善合约代码
如果你把所有的特性加上, 最后的合约代码如下:
![](https://ethereum.org/images/tutorial/advanced-token-deploy.png)
```
pragma solidity ^0.4.2;
contract owned {
    address public owner;

    function owned() {
        owner = msg.sender;
    }

    modifier onlyOwner {
        if (msg.sender != owner) throw;
        _
    }

    function transferOwnership(address newOwner) onlyOwner {
        owner = newOwner;
    }
}

contract tokenRecipient { function receiveApproval(address _from, uint256 _value, address _token, bytes _extraData); }

contract token {
    /* Public variables of the token */
    string public standard = 'Token 0.1';
    string public name;
    string public symbol;
    uint8 public decimals;
    uint256 public totalSupply;

    /* This creates an array with all balances */
    mapping (address => uint256) public balanceOf;
    mapping (address => mapping (address => uint256)) public allowance;

    /* This generates a public event on the blockchain that will notify clients */
    event Transfer(address indexed from, address indexed to, uint256 value);

    /* Initializes contract with initial supply tokens to the creator of the contract */
    function token(
        uint256 initialSupply,
        string tokenName,
        uint8 decimalUnits,
        string tokenSymbol
        ) {
        balanceOf[msg.sender] = initialSupply;              // Give the creator all initial tokens
        totalSupply = initialSupply;                        // Update total supply
        name = tokenName;                                   // Set the name for display purposes
        symbol = tokenSymbol;                               // Set the symbol for display purposes
        decimals = decimalUnits;                            // Amount of decimals for display purposes
        msg.sender.send(msg.value);                         // Send back any ether sent accidentally
    }

    /* Send coins */
    function transfer(address _to, uint256 _value) {
        if (balanceOf[msg.sender] < _value) throw;           // Check if the sender has enough
        if (balanceOf[_to] + _value < balanceOf[_to]) throw; // Check for overflows
        balanceOf[msg.sender] -= _value;                     // Subtract from the sender
        balanceOf[_to] += _value;                            // Add the same to the recipient
        Transfer(msg.sender, _to, _value);                   // Notify anyone listening that this transfer took place
    }

    /* Allow another contract to spend some tokens in your behalf */
    function approve(address _spender, uint256 _value)
        returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        tokenRecipient spender = tokenRecipient(_spender);
        return true;
    }

    /* Approve and then comunicate the approved contract in a single tx */
    function approveAndCall(address _spender, uint256 _value, bytes _extraData)
        returns (bool success) {
        tokenRecipient spender = tokenRecipient(_spender);
        if (approve(_spender, _value)) {
            spender.receiveApproval(msg.sender, _value, this, _extraData);
            return true;
        }
    }

    /* A contract attempts to get the coins */
    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {
        if (balanceOf[_from] < _value) throw;                 // Check if the sender has enough
        if (balanceOf[_to] + _value < balanceOf[_to]) throw;  // Check for overflows
        if (_value > allowance[_from][msg.sender]) throw;   // Check allowance
        balanceOf[_from] -= _value;                          // Subtract from the sender
        balanceOf[_to] += _value;                            // Add the same to the recipient
        allowance[_from][msg.sender] -= _value;
        Transfer(_from, _to, _value);
        return true;
    }

    /* This unnamed function is called whenever someone tries to send ether to it */
    function () {
        throw;     // Prevents accidental sending of ether
    }
}

contract MyAdvancedToken is owned, token {

    uint256 public sellPrice;
    uint256 public buyPrice;
    uint256 public totalSupply;

    mapping (address => bool) public frozenAccount;

    /* This generates a public event on the blockchain that will notify clients */
    event FrozenFunds(address target, bool frozen);

    /* Initializes contract with initial supply tokens to the creator of the contract */
    function MyAdvancedToken(
        uint256 initialSupply,
        string tokenName,
        uint8 decimalUnits,
        string tokenSymbol,
        address centralMinter
    ) token (initialSupply, tokenName, decimalUnits, tokenSymbol) {
        if(centralMinter != 0 ) owner = centralMinter;      // Sets the owner as specified (if centralMinter is not specified the owner is msg.sender)
        balanceOf[owner] = initialSupply;                   // Give the owner all initial tokens
    }

    /* Send coins */
    function transfer(address _to, uint256 _value) {
        if (balanceOf[msg.sender] < _value) throw;           // Check if the sender has enough
        if (balanceOf[_to] + _value < balanceOf[_to]) throw; // Check for overflows
        if (frozenAccount[msg.sender]) throw;                // Check if frozen
        balanceOf[msg.sender] -= _value;                     // Subtract from the sender
        balanceOf[_to] += _value;                            // Add the same to the recipient
        Transfer(msg.sender, _to, _value);                   // Notify anyone listening that this transfer took place
    }


    /* A contract attempts to get the coins */
    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {
        if (frozenAccount[_from]) throw;                        // Check if frozen
        if (balanceOf[_from] < _value) throw;                 // Check if the sender has enough
        if (balanceOf[_to] + _value < balanceOf[_to]) throw;  // Check for overflows
        if (_value > allowance[_from][msg.sender]) throw;   // Check allowance
        balanceOf[_from] -= _value;                          // Subtract from the sender
        balanceOf[_to] += _value;                            // Add the same to the recipient
        allowance[_from][msg.sender] -= _value;
        Transfer(_from, _to, _value);
        return true;
    }

    function mintToken(address target, uint256 mintedAmount) onlyOwner {
        balanceOf[target] += mintedAmount;
        totalSupply += mintedAmount;
        Transfer(0, this, mintedAmount);
        Transfer(this, target, mintedAmount);
    }

    function freezeAccount(address target, bool freeze) onlyOwner {
        frozenAccount[target] = freeze;
        FrozenFunds(target, freeze);
    }

    function setPrices(uint256 newSellPrice, uint256 newBuyPrice) onlyOwner {
        sellPrice = newSellPrice;
        buyPrice = newBuyPrice;
    }

    function buy() {
        uint amount = msg.value / buyPrice;                // calculates the amount
        if (balanceOf[this] < amount) throw;               // checks if it has enough to sell
        balanceOf[msg.sender] += amount;                   // adds the amount to buyer's balance
        balanceOf[this] -= amount;                         // subtracts amount from seller's balance
        Transfer(this, msg.sender, amount);                // execute an event reflecting the change
    }

    function sell(uint256 amount) {
        if (balanceOf[msg.sender] < amount ) throw;        // checks if the sender has enough to sell
        balanceOf[this] += amount;                         // adds the amount to owner's balance
        balanceOf[msg.sender] -= amount;                   // subtracts the amount from seller's balance
        if (!msg.sender.send(amount * sellPrice)) {        // sends ether to the seller. It's important
            throw;                                         // to do this last to avoid recursion attacks
        } else {
            Transfer(msg.sender, this, amount);            // executes an event reflecting on the change
        }
    }
}
```

### 部署合约
滚动到页面下方,你会看到预估的部署花费。你可以左滑设置一个更小的费用,但是如果价格低于市场平均,你的交易可能花费较长时间。点击*DEPLOY*并输入密码。几秒钟以后,你将被重定向到主界面,在**LATEST TRANSACTIONS**部分,你将看到一行写着*creating contract*。等待一些时间,其他节点开始调起你的交易,你可以看到一个缓慢增长的蓝色长方形,代表着有多少其他节点已经看到你的交易并确认了他们。越多的确认出现,也就越加保证了你的代码被部署了。
![](https://ethereum.org/images/tutorial/created-token.png)

点击写着*admin page*的链接,你就进入了世界上最简单的中央银行的管理界面,在这里,你可以对你新建的合约进行各种操作。

在左边的*READ FROM CONTRACT*栏,你可以免费的使用方法读取合约的值。如果你的合约有其他拥有者,这里会显示他的地址。复制地址然后粘贴在*Balance of*文本框,就会显示出该账号的余额。

在右边的*WRITE TO CONTRACT*栏,你可以看到所有可用的方法。执行这些方法需要消耗gas。如果你的合约可以挖矿,你应该有一个方法叫做*Mint Token*,选择这个方法:
![](https://ethereum.org/images/tutorial/manage-central-dollar.png)

选择代币接收地址和数量(如果代币的小数点为2,则增加两个0在数量后面)。在**Execute from**选择一个账号作为owner,将*Send ETHER*置零,最后点击*EXECUTE*。

在经过几次确认以后,接收者账户的余额就更新了。但是接收者的钱包可以不会自动展示出来:为了监听自定义代币,钱包必须手动添加合约地址到监听列表。复制你的合约地址(在管理页面,点击*copy address*),然后发送给你的接收者。如果他们还没有监听合约,他们可以到*CONTRACTS*栏, 点击*WATCH TOKEN*,然后将地址填在那里。合约基本属性会自动展示(用户也可以自定义)。主图标是不能改变的,用户发送接收代币的时候要确认是否选择了正确的代币。
![](https://ethereum.org/images/tutorial/add-token.png)

# -Source Code
                                   Árvore de preparação da Conexão Coin  0.1.0 


                              Trabalho em andamento (testável) - CONEXÃO COIN - COINN


 
                                                 Conexão Coin - COINN
 

VERSÃO  0.1.0  
 
Trabalho em andamento (testável)
INSTRUÇÕES
CRIE UMA CARTEIRA.
LOJA DE FRASES DE SEMENTES EM UM LUGAR SEGURO!
CARREGUE COM UMA PEQUENA QUANTIDADE DE ETH (0.00015 É MUITO).
CARREGUE COM UM PEQUENO QUANTO DE TOKEN SELECIONADO NA CARTEIRA PARA ATIVAR A FUNÇÃO.
 2018-02-12
Versão de teste pronta!
Adicionados novos tokens e integrações, além de uma nova interface de hiperlink.
Ainda com alguns problemas com o iOS e alguns navegadores, as soluções para iOS foram descobertas e serão adicionadas este ano.
Adicionará informações de marketcap para integrações de token no próximo patch.
Passaremos algum tempo em que a comunidade testará a carteira para prepará-la para distribuição.
2018-02-12
Várias correções de erros e adição de vários recursos. Atualizaremos este leia-me para ser mais preciso em breve. Desculpe, aproveitamos nossas férias e esquecemos de atualizar isso = X
2018-02-12
Funcionalidade Etherscan adicionada para carteiras de usuários
Limites inferiores ajustados para que o consumo de gás exija 0,00025 eth para operação da carteira (abaixo de 0,00033)
2018-02-12
NOTAS
Financie a Carteira virtual com US $ A e uma pequena quantidade de éter (0,0001 servirá) ou ela não funcionará corretamente
Alguns navegadores não funcionam bem com o uso deste Dapp
Funções 2fa virão em breve!
Os usuários do iOS ganharão funcionalidade no próximo patch (esperamos)
Também para o próximo patch ... Tokens ERC20 adicionais ... Possivelmente a troca na carteira também
Adicionaremos uma configuração para ajustar o gwei no próximo patch também!
Detalhes.
Se você não possui uma carteira ERC20, pode criar uma (armazena todas as ERC20s)
Os preços do gás são padronizados nas configurações MAIS BAIXAS através da rede ethereum
Observe que não há limites no momento. Em breve, adicionaremos mais complexidade para proteger nossos usuários ... mas se você inserir entradas incorretas para o Endereço / Valor, isso poderá causar perda de fundos!
O endereço padrão é a carteira de $ A e sempre aceitamos contribuições! =)
 
 
Código-fonte do contrato verificado
/**
 *Submitted for verification at Etherscan.io on 2018-02-12
*/

pragma solidity ^0.4.15;

contract Token {
    /* This is a slight change to the ERC20 base standard.
    function totalSupply() constant returns (uint256 supply);
    is replaced with:
    uint256 public totalSupply;
    This automatically creates a getter function for the totalSupply.
    This is moved to the base contract since public getter functions are not
    currently recognised as an implementation of the matching abstract
    function by the compiler.
    */
    /// total amount of tokens
    uint256 public totalSupply;

    /// @param _owner The address from which the balance will be retrieved
    /// @return The balance
    function balanceOf(address _owner) constant returns (uint256 balance);

    /// @notice send `_value` token to `_to` from `msg.sender`
    /// @param _to The address of the recipient
    /// @param _value The amount of token to be transferred
    /// @return Whether the transfer was successful or not
    function transfer(address _to, uint256 _value) returns (bool success);

    /// @notice send `_value` token to `_to` from `_from` on the condition it is approved by `_from`
    /// @param _from The address of the sender
    /// @param _to The address of the recipient
    /// @param _value The amount of token to be transferred
    /// @return Whether the transfer was successful or not
    function transferFrom(address _from, address _to, uint256 _value) returns (bool success);

    /// @notice `msg.sender` approves `_spender` to spend `_value` tokens
    /// @param _spender The address of the account able to transfer the tokens
    /// @param _value The amount of tokens to be approved for transfer
    /// @return Whether the approval was successful or not
    function approve(address _spender, uint256 _value) returns (bool success);

    /// @param _owner The address of the account owning tokens
    /// @param _spender The address of the account able to transfer the tokens
    /// @return Amount of remaining tokens allowed to spent
    function allowance(address _owner, address _spender) constant returns (uint256 remaining);

    event Transfer(address indexed _from, address indexed _to, uint256 _value);
    event Approval(address indexed _owner, address indexed _spender, uint256 _value);
}

contract StandardToken is Token {

    function transfer(address _to, uint256 _value) returns (bool success) {
        //Default assumes totalSupply can't be over max (2^256 - 1).
        //If your token leaves out totalSupply and can issue more tokens as time goes on, you need to check if it doesn't wrap.
        //Replace the if with this one instead.
        //require(balances[msg.sender] >= _value && balances[_to] + _value > balances[_to]);
        require(balances[msg.sender] >= _value);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        Transfer(msg.sender, _to, _value);
        return true;
    }

    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {
        //same as above. Replace this line with the following if you want to protect against wrapping uints.
        //require(balances[_from] >= _value && allowed[_from][msg.sender] >= _value && balances[_to] + _value > balances[_to]);
        require(balances[_from] >= _value && allowed[_from][msg.sender] >= _value);
        balances[_to] += _value;
        balances[_from] -= _value;
        allowed[_from][msg.sender] -= _value;
        Transfer(_from, _to, _value);
        return true;
    }

    function balanceOf(address _owner) constant returns (uint256 balance) {
        return balances[_owner];
    }

    function approve(address _spender, uint256 _value) returns (bool success) {
        allowed[msg.sender][_spender] = _value;
        Approval(msg.sender, _spender, _value);
        return true;
    }

    function allowance(address _owner, address _spender) constant returns (uint256 remaining) {
      return allowed[_owner][_spender];
    }

    /* Approves and then calls the receiving contract */
    function approveAndCall(address _spender, uint256 _value, bytes _extraData) returns (bool success) {
        allowed[msg.sender][_spender] = _value;
        Approval(msg.sender, _spender, _value);

        //call the receiveApproval function on the contract you want to be notified. This crafts the function signature manually so one doesn't have to include a contract in here just for this.
        //receiveApproval(address _from, uint256 _value, address _tokenContract, bytes _extraData)
        //it is assumed that when does this that the call *should* succeed, otherwise one would use vanilla approve instead.
        require(_spender.call(bytes4(bytes32(sha3("receiveApproval(address,uint256,address,bytes)"))), msg.sender, _value, this, _extraData));
        return true;
    }

    mapping (address => uint256) balances;
    mapping (address => mapping (address => uint256)) allowed;
} 

contract AbstractSingularDTVFund {
    function softWithdrawRewardFor(address forAddress) returns (uint);
}

/// @title Token contract - Implements token issuance.
/// @author Stefan George - <stefan.george@consensys.net>
/// @author Milad Mostavi - <milad.mostavi@consensys.net>
contract SingularDTVToken is StandardToken {
    string public version = "0.1.0";

    /*
     *  External contracts
     */
    AbstractSingularDTVFund public singularDTVFund;

    /*
     *  Token meta data
     */
    string public name;
    string public symbol;
    uint8 public constant decimals = 18;

    /// @dev Transfers sender's tokens to a given address. Returns success.
    /// @param to Address of token receiver.
    /// @param value Number of tokens to transfer.
    function transfer(address to, uint256 value)
        returns (bool)
    {
        // Both parties withdraw their reward first
        singularDTVFund.softWithdrawRewardFor(msg.sender);
        singularDTVFund.softWithdrawRewardFor(to);
        return super.transfer(to, value);
    }

    /// @dev Allows allowed third party to transfer tokens from one address to another. Returns success.
    /// @param from Address from where tokens are withdrawn.
    /// @param to Address to where tokens are sent.
    /// @param value Number of tokens to transfer.
    function transferFrom(address from, address to, uint256 value)
        returns (bool)
    {
        // Both parties withdraw their reward first
        singularDTVFund.softWithdrawRewardFor(from);
        singularDTVFund.softWithdrawRewardFor(to);
        return super.transferFrom(from, to, value);
    }

    function SingularDTVToken(address sDTVFundAddr, address _wallet, string _name, string _symbol, uint _totalSupply) {
        if(sDTVFundAddr == 0 || _wallet == 0) {
            // Fund and Wallet addresses should not be null.
            revert();
        }

        balances[_wallet] = _totalSupply;
        totalSupply = _totalSupply;

        name = _name;
        symbol = _symbol;

        singularDTVFund = AbstractSingularDTVFund(sDTVFundAddr);

        Transfer(this, _wallet, _totalSupply);
    }
}


 
VERSÃO  0.1.0  
 
 
Trabalho em andamento (testável)

INSTRUÇÕES

CRIE UMA CARTEIRA.

LOJA DE FRASES DE SEMENTES EM UM LUGAR SEGURO!

CARREGUE COM UMA PEQUENA QUANTIDADE DE ETH (0.00001 É MUITO).

CARREGUE COM UM PEQUENO QUANTO DE TOKEN SELECIONADO NA CARTEIRA PARA ATIVAR A FUNÇÃO.

 2018-02-12
 
Versão de teste pronta!

Adicionados novos tokens e integrações, além de uma nova interface de hiperlink.

Ainda com alguns problemas com o iOS e alguns navegadores, as soluções para iOS foram descobertas e serão adicionadas este ano.
Adicionará informações de marketcap para integrações de token no próximo patch.

Passaremos algum tempo em que a comunidade testará a carteira para prepará-la para distribuição.
2018-02-12
Várias correções de erros e adição de vários recursos. Atualizaremos este leia-me para ser mais preciso em breve. Desculpe,

aproveitamos nossas férias e esquecemos de atualizar isso = X
2018-02-12

Funcionalidade Etherscan adicionada para carteiras de usuários

Limites inferiores ajustados para que o consumo de gás exija 0,00025 eth para operação da carteira (abaixo de 0,00033)
2018-02-12

NOTAS

Financie a Carteira virtual com US $ A e uma pequena quantidade de éter (0,0001 servirá) ou ela não funcionará corretamente

Alguns navegadores não funcionam bem com o uso deste Dapp

Funções 2fa virão em breve!

Os usuários do iOS ganharão funcionalidade no próximo patch (esperamos)

Também para o próximo patch ... Tokens ERC20 adicionais ... Possivelmente a troca na carteira também

Adicionaremos uma configuração para ajustar o gwei no próximo patch também!
Detalhes.

Se você não possui uma carteira ERC20, pode criar uma (armazena todas as ERC20s)

Os preços do gás são padronizados nas configurações MAIS BAIXAS através da rede ethereum

Observe que não há limites no momento. Em breve, adicionaremos mais complexidade para proteger nossos usuários ... mas se 

você inserir entradas incorretas para o Endereço / Valor, isso poderá causar perda de fundos!

O endereço padrão é a carteira de $ A e sempre aceitamos contribuições! =)


Licença

A conexão coin é lançado sob os termos da licença MIT. Consulte COPYING para obter mais informações ou https://opensource.org/licenses/MIT .

Processo de desenvolvimento

O masterramo deve ser estável. O desenvolvimento normalmente é feito em ramificações separadas. As tags são criadas para indicar novas versões oficiais e estáveis ​​da conexão coin.

O fluxo de trabalho de contribuição é descrito em CONTRIBUTING.md .

Teste

Teste e revisão de código são o gargalo do desenvolvimento; recebemos mais solicitações pull do que podemos revisar e testar em pouco tempo. Seja paciente e ajude testando as solicitações de recebimento de outras pessoas, e lembre-se de que este é um projeto crítico para a segurança, onde qualquer erro pode custar muito dinheiro às pessoas.

Teste automatizado

Os desenvolvedores são fortemente encorajados a escrever testes de unidade para o novo código e enviar novos testes de unidade para o código antigo. Os testes de unidade pode ser compilado e executado (assumindo que eles não foram desabilitados na configure) com: make check. Detalhes adicionais sobre a execução e a extensão de testes de unidade.

Teste de garantia de qualidade manual (QA)

As alterações devem ser testadas por alguém que não seja o desenvolvedor que escreveu o código. Isso é especialmente importante para alterações grandes ou de alto risco. É útil adicionar um plano de teste à descrição da solicitação pull, se o teste das alterações não for direto.

Traduções

Alterações nas traduções e novas traduções podem ser enviadas para a página da conexão coin .

Entre em contato e saiba mais:
 
Fale com nossa equipe 
https://conexaocoin.com/ 

https://conexaocoin.com/contato 

servidor@conexaocoin.com 

https://twitter.com/conexaocoin

https://www.instagram.com/conexaocoin/

https://www.facebook.com/conexaocoin/

https://discord.gg/bhQReAB 

https://www.reddit.com/user/conexaocoin 

https://t.me/conexaocoin   

 

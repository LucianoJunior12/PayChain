// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IERC20 {
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
}

contract PaymentSystem {
    IERC20 public usdcToken;
    address public owner;

    struct Payment {
        uint256 tableId;  // ID da mesa
        uint256 amount;   // Valor (em USDC, 6 decimais)
        address payer;    // Será preenchido quando o cliente pagar
        bool isPaid;
        bool isCancelled;
    }

    uint256 public paymentCount;
    mapping(uint256 => Payment) public payments;

    event PaymentCreated(uint256 paymentId, uint256 tableId, uint256 amount);
    event PaymentCompleted(uint256 paymentId, address payer);
    event PaymentCancelled(uint256 paymentId, address payer);

    modifier onlyOwner() {
        require(msg.sender == owner, "Apenas o dono pode executar esta funcao.");
        _;
    }

    constructor(address _usdcTokenAddress) {
        usdcToken = IERC20(_usdcTokenAddress);
        owner = msg.sender;
    }

    // Função para criar uma fatura (chamada pelo owner/estabelecimento, via relayer ou backend)
    function createPaymentByOwner(uint256 tableId, uint256 amount) public onlyOwner returns (uint256) {
        paymentCount++;
        payments[paymentCount] = Payment(tableId, amount, address(0), false, false);
        emit PaymentCreated(paymentCount, tableId, amount);
        return paymentCount;
    }

    // Função para completar o pagamento (chamada pelo cliente)
    function completePayment(uint256 paymentId) public {
        Payment storage payment = payments[paymentId];
        require(!payment.isPaid, "Pagamento ja realizado.");
        require(!payment.isCancelled, "Pagamento foi cancelado.");
        uint256 amount = payment.amount;
        require(usdcToken.balanceOf(msg.sender) >= amount, "Saldo insuficiente.");
        require(usdcToken.transferFrom(msg.sender, owner, amount), "Falha na transferencia de USDC.");
        payment.isPaid = true;
        payment.payer = msg.sender;
        emit PaymentCompleted(paymentId, msg.sender);
    }
    
    // Função de consulta da fatura
    function getPayment(uint256 paymentId) public view returns (uint256 tableId, uint256 amount, address payer, bool isPaid, bool isCancelled) {
        Payment memory p = payments[paymentId];
        return (p.tableId, p.amount, p.payer, p.isPaid, p.isCancelled);
    }

    // Função para cancelar (se necessário) – somente o owner
    function cancelPayment(uint256 paymentId) public onlyOwner {
        Payment storage payment = payments[paymentId];
        require(!payment.isPaid, "Pagamento ja realizado, nao pode ser cancelado.");
        require(!payment.isCancelled, "Pagamento ja foi cancelado.");
        payment.isCancelled = true;
        emit PaymentCancelled(paymentId, payment.payer);
    }
}

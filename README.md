# nexora-bank
Simulação de banco digital com login, contas, cartões e sistema de transações, desenvolvido em HTML, CSS e JavaScript (front-end only).
<!DOCTYPE html>
<html lang="pt">

<head>
    <meta charset="UTF-8">
    <title>Nexora Bank</title>
    <link rel="stylesheet" href="styles.css">
</head>

<body>

<header>
    <h1>Nexora Bank</h1>
</header>

<!-- LOGIN -->
<section id="login">
    <h2>Login</h2>
    <input id="nif" placeholder="NIF (9 números)">
    <button id="loginBtn">Entrar</button>
</section>

<!-- APP -->
<section id="app" class="hidden">

    <h2>Conta</h2>

    <p>NIF: <span id="userNif"></span></p>
    <p>Cartão: <span id="card"></span></p>
    <p>Saldo: <span id="balance"></span> €</p>

    <h3>Pagar com cartão</h3>
    <input id="payAmount" placeholder="Valor">
    <input id="cardInput" placeholder="Número do cartão">
    <button id="payBtn">Pagar</button>

    <h3>Admin (só para ti)</h3>
    <input id="targetNif" placeholder="NIF utilizador">
    <input id="amount" placeholder="Valor">
    <button id="adminBtn">Carregar dinheiro</button>

    <h3>Histórico</h3>
    <ul id="history"></ul>

    <button id="logout">Sair</button>

</section>

<script src="script.js"></script>

</body>
</html>

<style>
   body {
    font-family: Arial;
    background: #050816;
    color: white;
    padding: 20px;
}

h1 {
    color: #8b5cf6;
}

input {
    display: block;
    margin: 10px 0;
    padding: 10px;
    width: 250px;
    background: #0f172a;
    color: white;
    border: 1px solid #7c3aed;
}

button {
    padding: 10px;
    background: #7c3aed;
    color: white;
    border: none;
    cursor: pointer;
    margin: 5px 0;
}

button:hover {
    background: #6d28d9;
}

.hidden {
    display: none;
}

ul {
    margin-top: 10px;
}
</style>

<script>
    let currentUser = null;

const ADMIN = "123456789";

/* STORAGE */
function getUsers() {
    return JSON.parse(localStorage.getItem("users")) || [];
}

function saveUsers(users) {
    localStorage.setItem("users", JSON.stringify(users));
}

/* CARTÃO */
function generateCard(nif) {
    if (nif === ADMIN) {
        return "NEX-ADMIN-0000-0000";
    }
    return "NEX-" + Math.floor(Math.random() * 999999999);
}

/* UPDATE UI */
function updateUI() {

    document.getElementById("userNif").textContent = currentUser.nif;
    document.getElementById("card").textContent = currentUser.card;
    document.getElementById("balance").textContent = currentUser.balance;

    const list = document.getElementById("history");
    list.innerHTML = "";

    currentUser.history.forEach(h => {
        const li = document.createElement("li");
        li.textContent = h;
        list.appendChild(li);
    });

    // esconder admin para utilizadores normais
    if (currentUser.nif !== ADMIN) {
        document.getElementById("targetNif").style.display = "none";
        document.getElementById("amount").style.display = "none";
        document.getElementById("adminBtn").style.display = "none";
    }
}

/* LOGIN */
document.getElementById("loginBtn").addEventListener("click", () => {

    const nif = document.getElementById("nif").value.trim();

    if (nif.length !== 9 || isNaN(nif)) {
        alert("NIF inválido");
        return;
    }

    let users = getUsers();
    let user = users.find(u => u.nif === nif);

    if (!user) {
        user = {
            nif,
            balance: 0,
            card: generateCard(nif),
            history: ["Conta criada"]
        };
        users.push(user);
    }

    currentUser = user;

    saveUsers(users);

    document.getElementById("login").classList.add("hidden");
    document.getElementById("app").classList.remove("hidden");

    updateUI();
});

/* ADMIN CARREGAR DINHEIRO */
document.getElementById("adminBtn").addEventListener("click", () => {

    if (currentUser.nif !== ADMIN) return;

    const nif = document.getElementById("targetNif").value.trim();
    const amount = Number(document.getElementById("amount").value);

    let users = getUsers();
    let user = users.find(u => u.nif === nif);

    if (!user || isNaN(amount) || amount <= 0) {
        alert("Erro");
        return;
    }

    user.balance += amount;
    user.history.push("+" + amount + "€ carregado pelo banco");

    saveUsers(users);

    alert("Dinheiro carregado");
});

/* PAGAMENTO */
document.getElementById("payBtn").addEventListener("click", () => {

    const amount = Number(document.getElementById("payAmount").value);
    const card = document.getElementById("cardInput").value.trim();

    if (card !== currentUser.card) {
        alert("Cartão inválido");
        return;
    }

    if (currentUser.balance < amount) {
        alert("Saldo insuficiente");
        return;
    }

    currentUser.balance -= amount;
    currentUser.history.push("-" + amount + "€ pagamento cartão");

    let users = getUsers();
    let index = users.findIndex(u => u.nif === currentUser.nif);
    users[index] = currentUser;

    saveUsers(users);

    updateUI();
});

/* LOGOUT */
document.getElementById("logout").addEventListener("click", () => {
    location.reload();
});
</script>

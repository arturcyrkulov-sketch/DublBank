<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=yes, viewport-fit=cover">
  <title>ДубльБанк | Цифровая система</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
    }
    body {
      background: #0a0f1e;
      color: #e0e4f0;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 20px;
    }
    .app-container {
      max-width: 1300px;
      width: 100%;
      background: #111827;
      border-radius: 32px;
      box-shadow: 0 25px 60px rgba(0, 0, 0, 0.6);
      padding: 24px;
      display: flex;
      flex-direction: column;
      gap: 24px;
      border: 1px solid #2a3441;
    }
    h1,
    h2,
    h3 {
      color: #f0f4ff;
      font-weight: 600;
    }
    .header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
      gap: 15px;
      border-bottom: 1px solid #2d3748;
      padding-bottom: 15px;
    }
    .logo {
      font-size: 28px;
      font-weight: 700;
      background: linear-gradient(135deg, #3b82f6, #8b5cf6);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }
    .auth-panel {
      display: flex;
      gap: 10px;
      align-items: center;
      background: #1e293b;
      padding: 8px 15px;
      border-radius: 40px;
    }
    input,
    select,
    textarea,
    button {
      background: #1e293b;
      border: 1px solid #334155;
      color: #f1f5f9;
      border-radius: 12px;
      padding: 10px 16px;
      outline: none;
      font-size: 14px;
    }
    button {
      background: #2563eb;
      border: none;
      font-weight: 600;
      cursor: pointer;
      transition: 0.2s;
      display: flex;
      align-items: center;
      gap: 6px;
    }
    button:hover {
      background: #3b82f6;
    }
    button.danger {
      background: #b91c1c;
    }
    button.danger:hover {
      background: #dc2626;
    }
    .dashboard {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
      gap: 20px;
    }
    .card {
      background: #1e293b;
      border-radius: 24px;
      padding: 20px;
      border: 1px solid #2d3a4f;
      box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
    }
    .tabs {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      margin: 15px 0;
    }
    .tab-btn {
      background: #1e293b;
      border-radius: 20px;
      padding: 8px 18px;
    }
    .tab-btn.active {
      background: #2563eb;
      box-shadow: 0 0 15px #2563eb70;
    }
    .hidden {
      display: none !important;
    }
    .grid-2col {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 16px;
    }
    .chat-window {
      background: #0f172a;
      border-radius: 16px;
      padding: 12px;
      height: 250px;
      overflow-y: auto;
      margin: 10px 0;
      border: 1px solid #334155;
    }
    .message {
      margin-bottom: 12px;
      padding: 8px 12px;
      border-radius: 18px;
      max-width: 85%;
      background: #1e3a5f;
    }
    .message.own {
      background: #2563eb;
      margin-left: auto;
    }
    .news-item {
      background: #1e293b;
      padding: 12px;
      border-radius: 14px;
      margin-bottom: 8px;
      border-left: 4px solid #3b82f6;
    }
    hr {
      border-color: #2d3748;
      margin: 15px 0;
    }
    .flex-row {
      display: flex;
      gap: 10px;
      align-items: center;
      flex-wrap: wrap;
    }
    .notice {
      background: #1e3a5f;
      border: 1px solid #3b82f6;
      color: #93c5fd;
      padding: 12px;
      border-radius: 12px;
      margin: 10px 0;
    }
    .error-message {
      color: #f87171;
      font-size: 12px;
      margin-top: 4px;
    }
  </style>
</head>
<body>
  <div class="app-container" id="app">
    <div class="header">
      <span class="logo">🏦 ДубльБанк</span>
      <div class="auth-panel" id="authBlock">
        <input type="password" id="authCodeInput" placeholder="6-значный код" maxlength="6" inputmode="numeric" autocomplete="off">
        <button id="loginBtn">🔐 Войти</button>
        <span id="currentUserDisplay" style="margin-left:8px; color:#94a3b8;"></span>
        <button id="logoutBtn" class="hidden">Выйти</button>
      </div>
    </div>
    <div id="mainUI" class="hidden">
      <div class="tabs" id="tabContainer"></div>
      <div id="tabContent" style="margin-top:10px;"></div>
    </div>
  </div>

  <script>
    (function() {
      // ---------- ХРАНИЛИЩЕ ----------
      const State = {
        balanceGos: 16888567556067,
        currencyRates: { USD: 2.3478, EUR: 2.5123, CNY: 0.3245, RUB: 0.0256 },
        commissionDefault: 1.2,
        clients: new Map(),
        news: [],
        documentTemplates: [
          { id: "t1", name: "Заявление на кредит", content: "Я, [ФИО], прошу выдать кредит на сумму [сумма] дублей." },
          { id: "t2", name: "Договор микрозайма", content: "Микрозайм между [ФИО] и банком на [сумма] дублей." }
        ],
        submittedDocs: [],
        chats: new Map(),
        transferHistory: [],
        loanRate: 12,
        microLoanRate: 0.8,
        depositRate: 5,
        nextClientId: 1001
      };

      function saveState() {
        const data = {
          balanceGos: State.balanceGos,
          currencyRates: State.currencyRates,
          commissionDefault: State.commissionDefault,
          clients: Array.from(State.clients.entries()),
          news: State.news,
          documentTemplates: State.documentTemplates,
          submittedDocs: State.submittedDocs,
          chats: Array.from(State.chats.entries()),
          transferHistory: State.transferHistory,
          loanRate: State.loanRate,
          microLoanRate: State.microLoanRate,
          depositRate: State.depositRate,
          nextClientId: State.nextClientId
        };
        localStorage.setItem('dublbank_data', JSON.stringify(data));
      }

      function loadState() {
        const saved = localStorage.getItem('dublbank_data');
        if (saved) {
          try {
            const data = JSON.parse(saved);
            State.balanceGos = data.balanceGos || 16888567556067;
            State.currencyRates = data.currencyRates || { USD: 2.3478, EUR: 2.5123, CNY: 0.3245, RUB: 0.0256 };
            State.commissionDefault = data.commissionDefault || 1.2;
            State.clients = new Map(data.clients || []);
            State.news = data.news || [];
            State.documentTemplates = data.documentTemplates || [
              { id: "t1", name: "Заявление на кредит", content: "Я, [ФИО], прошу выдать кредит на сумму [сумма] дублей." },
              { id: "t2", name: "Договор микрозайма", content: "Микрозайм между [ФИО] и банком на [сумма] дублей." }
            ];
            State.submittedDocs = data.submittedDocs || [];
            State.chats = new Map(data.chats || []);
            State.transferHistory = data.transferHistory || [];
            State.loanRate = data.loanRate || 12;
            State.microLoanRate = data.microLoanRate || 0.8;
            State.depositRate = data.depositRate || 5;
            State.nextClientId = data.nextClientId || 1001;
            return true;
          } catch (e) {
            console.error('Ошибка загрузки:', e);
            return false;
          }
        }
        return false;
      }

      if (!loadState()) {
        State.clients.set("admin", {
          id: "admin",
          name: "Главный Администратор",
          code: "676776",
          balance: 1000000000,
          isAdmin: true,
          cardBlocked: false,
          customCommission: null
        });
        saveState();
      } else if (!State.clients.has("admin")) {
        State.clients.set("admin", {
          id: "admin",
          name: "Главный Администратор",
          code: "676776",
          balance: 1000000000,
          isAdmin: true,
          cardBlocked: false,
          customCommission: null
        });
        saveState();
      }

      let currentUser = null;
      const authInput = document.getElementById('authCodeInput');
      const loginBtn = document.getElementById('loginBtn');
      const logoutBtn = document.getElementById('logoutBtn');
      const currentUserDisplay = document.getElementById('currentUserDisplay');
      const mainUI = document.getElementById('mainUI');
      const tabContainer = document.getElementById('tabContainer');
      const tabContent = document.getElementById('tabContent');

      // Безопасность: скрываем коды при входе
      authInput.addEventListener('input', (e) => {
        e.target.value = e.target.value.replace(/\D/g, '').slice(0, 6);
      });

      // Автовход при сохранённой сессии
      const savedSession = localStorage.getItem('dublbank_session');
      if (savedSession) {
        try {
          const sessionData = JSON.parse(savedSession);
          const savedUser = State.clients.get(sessionData.userId);
          if (savedUser && savedUser.code === sessionData.code) {
            currentUser = savedUser;
            setTimeout(() => updateUIAfterAuth(), 50);
          } else {
            localStorage.removeItem('dublbank_session');
          }
        } catch (e) {
          localStorage.removeItem('dublbank_session');
        }
      }

      loginBtn.addEventListener('click', () => {
        const code = authInput.value.trim();
        if (code.length !== 6) { alert("Введите 6 цифр кода"); return; }
        let found = null;
        for (let [id, c] of State.clients) {
          if (c.code === code) { found = c; break; }
        }
        if (!found) { alert("Неверный код авторизации"); return; }
        currentUser = found;
        localStorage.setItem('dublbank_session', JSON.stringify({ userId: found.id, code: found.code }));
        updateUIAfterAuth();
        authInput.value = '';
      });

      logoutBtn.addEventListener('click', () => {
        currentUser = null;
        mainUI.classList.add('hidden');
        logoutBtn.classList.add('hidden');
        currentUserDisplay.textContent = '';
        localStorage.removeItem('dublbank_session');
        saveState();
      });

      window.addEventListener('beforeunload', () => saveState());
      setInterval(() => { if (currentUser) saveState(); }, 5000);

      function updateUIAfterAuth() {
        if (!currentUser) return;
        mainUI.classList.remove('hidden');
        logoutBtn.classList.remove('hidden');
        currentUserDisplay.textContent = `👤 ${currentUser.name} (${currentUser.isAdmin ? 'Админ' : 'Клиент'})`;
        renderNavigation();
        switchTab(currentUser.isAdmin ? 'админ' : 'главная');
      }

      function renderNavigation() {
        const tabs = currentUser.isAdmin
          ? ['📊 Админ', '💰 Кредиты', '📉 Микрозайм', '📈 Взнос', '💱 Курсы', '📰 Новости', '📄 Шаблоны', '📨 Подача', '💬 Чаты', '📜 История', '🤖 ИИ']
          : ['🏠 Главная', '💱 Курсы', '💸 Переводы', '📄 Шаблоны', '📨 Подача', '💬 Чаты', '📜 История', '🤖 ИИ'];
        tabContainer.innerHTML = tabs.map(t => {
          let tabKey = t.split(' ')[1]?.toLowerCase() || t.toLowerCase();
          return `<button class="tab-btn" data-tab="${tabKey}">${t}</button>`;
        }).join('');
        document.querySelectorAll('.tab-btn').forEach(btn => {
          btn.addEventListener('click', (e) => switchTab(e.target.dataset.tab));
        });
      }

      function switchTab(tab) {
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
        const activeBtn = Array.from(document.querySelectorAll('.tab-btn')).find(b => b.dataset.tab === tab);
        if (activeBtn) activeBtn.classList.add('active');
        renderTabContent(tab);
      }

      function renderTabContent(tab) {
        if (!currentUser) return;
        switch (tab) {
          case 'админ': renderAdminPanel(); break;
          case 'главная': renderClientDashboard(); break;
          case 'кредиты': renderLoans(); break;
          case 'микрозайм': renderMicroLoan(); break;
          case 'взнос': renderDeposit(); break;
          case 'курсы': renderCurrencyRates(); break;
          case 'переводы': renderTransfers(); break;
          case 'новости': renderNews(); break;
          case 'шаблоны': renderTemplates(); break;
          case 'подача': renderDocSubmission(); break;
          case 'чаты': renderChats(); break;
          case 'история': renderHistory(); break;
          case 'ии': renderAI(); break;
          default: tabContent.innerHTML = '<p>Выберите раздел</p>';
        }
      }

      // ========== АДМИН-ПАНЕЛЬ ==========
      function renderAdminPanel() {
        if (!currentUser || !currentUser.isAdmin) {
          tabContent.innerHTML = '<p>⛔ Доступ запрещён</p>';
          return;
        }
        let html = `<div class="dashboard">
        <div class="card"><h3>🏛 Баланс государства</h3><h2>${State.balanceGos.toLocaleString()} дублей</h2>
        <input id="newGosBalance" type="number" placeholder="Новый баланс"><button id="updateGosBalance">Обновить</button></div>
        <div class="card"><h3>📊 ВВП</h3><h2>${(State.balanceGos * State.currencyRates.USD).toLocaleString()} $</h2></div>
        <div class="card"><h3>💱 Курс USD</h3><input id="usdRateInput" value="${State.currencyRates.USD}" step="0.0001"><button id="updateUsdRate">Изменить</button></div>
        <div class="card"><h3>👥 Клиенты</h3><div id="clientsList"></div>
        <input id="newClientName" placeholder="Имя"><input id="newClientCode" placeholder="6-значный код" maxlength="6"><button id="addClientBtn">➕ Добавить</button></div>
        <div class="card"><h3>⚙️ Комиссия переводов</h3><input id="globalCommission" value="${State.commissionDefault}" type="number" step="0.1"><button id="updateCommission">Обновить общую</button></div>
        <div class="card"><h3>📈 Ставки</h3>
        <label>Кредит: <input id="loanRate" value="${State.loanRate}" type="number" step="0.1">%</label>
        <label>Микрозайм: <input id="microLoanRate" value="${State.microLoanRate}" type="number" step="0.01">%</label>
        <label>Взнос: <input id="depositRate" value="${State.depositRate}" type="number" step="0.1">%</label>
        <button id="updateRates">Обновить ставки</button></div>
      </div>`;
        tabContent.innerHTML = html;
        renderClientsList();

        document.getElementById('updateGosBalance').addEventListener('click', () => {
          let val = parseFloat(document.getElementById('newGosBalance').value);
          if (!isNaN(val) && val > 0) { State.balanceGos = val; saveState(); renderAdminPanel(); }
        });
        document.getElementById('updateUsdRate').addEventListener('click', () => {
          let val = parseFloat(document.getElementById('usdRateInput').value);
          if (!isNaN(val) && val > 0) { State.currencyRates.USD = val; saveState(); renderAdminPanel(); }
        });
        document.getElementById('updateCommission').addEventListener('click', () => {
          let val = parseFloat(document.getElementById('globalCommission').value);
          if (!isNaN(val) && val >= 0) { State.commissionDefault = val; saveState(); }
        });
        document.getElementById('updateRates').addEventListener('click', () => {
          State.loanRate = parseFloat(document.getElementById('loanRate').value) || 12;
          State.microLoanRate = parseFloat(document.getElementById('microLoanRate').value) || 0.8;
          State.depositRate = parseFloat(document.getElementById('depositRate').value) || 5;
          saveState();
        });
        document.getElementById('addClientBtn').addEventListener('click', () => {
          let name = document.getElementById('newClientName').value.trim();
          let code = document.getElementById('newClientCode').value.trim();
          if (!name || code.length !== 6 || !/^\d+$/.test(code)) {
            alert("Введите имя и 6-значный цифровой код");
            return;
          }
          let codeExists = false;
          for (let [id, c] of State.clients) { if (c.code === code) { codeExists = true; break; } }
          if (codeExists) { alert("Клиент с таким кодом уже существует!"); return; }
          let newId = (State.nextClientId++).toString();
          State.clients.set(newId, { id: newId, name, code, balance: 0, isAdmin: false, cardBlocked: false, customCommission: null });
          saveState();
          renderClientsList();
          document.getElementById('newClientName').value = '';
          document.getElementById('newClientCode').value = '';
        });
      }

      function renderClientsList() {
        const container = document.getElementById('clientsList');
        if (!container) return;
        let html = '';
        let hasClients = false;
        for (let [id, c] of State.clients) {
          if (c.isAdmin) continue;
          hasClients = true;
          html += `<div style="margin:5px 0; background:#0f172a; padding:10px; border-radius:8px;">
          <strong>${c.name}</strong> (ID:${id})<br>💰 Баланс: ${c.balance.toLocaleString()} дублей<br>
          <div style="margin-top:8px; display:flex; gap:5px; flex-wrap:wrap;">
          <button data-del="${id}" class="danger">❌</button>
          <button data-block="${id}">${c.cardBlocked?'🔓 Разблок':'🔒 Заблок'}</button>
          <button data-balance="${id}">💰 Баланс</button>
          <button data-code="${id}">🔑 Код</button>
          <input data-comm="${id}" placeholder="Комиссия %" size="4" value="${c.customCommission??''}" style="width:80px;">
          </div></div>`;
        }
        if (!hasClients) html = '<p style="color:#94a3b8;">Нет клиентов.</p>';
        container.innerHTML = html;
        container.querySelectorAll('[data-del]').forEach(b => b.addEventListener('click', (e) => {
          if (confirm('Удалить клиента?')) { State.clients.delete(e.target.dataset.del); saveState(); renderClientsList(); }
        }));
        container.querySelectorAll('[data-block]').forEach(b => b.addEventListener('click', (e) => {
          let cl = State.clients.get(e.target.dataset.block);
          if (cl) { cl.cardBlocked = !cl.cardBlocked; saveState(); renderClientsList(); }
        }));
        container.querySelectorAll('[data-comm]').forEach(inp => inp.addEventListener('change', (e) => {
          let cl = State.clients.get(e.target.dataset.comm);
          if (cl) { cl.customCommission = parseFloat(e.target.value) || null; saveState(); }
        }));
        container.querySelectorAll('[data-code]').forEach(b => b.addEventListener('click', (e) => {
          let cl = State.clients.get(e.target.dataset.code);
          if (cl) {
            let newCode = prompt(`Новый 6-значный код для ${cl.name}:`, cl.code);
            if (newCode && newCode.length === 6 && /^\d+$/.test(newCode)) {
              let exists = [...State.clients.values()].some(c => c.code === newCode && c.id !== cl.id);
              if (exists) alert("Код уже используется!");
              else { cl.code = newCode; saveState(); renderClientsList(); }
            }
          }
        }));
        container.querySelectorAll('[data-balance]').forEach(b => b.addEventListener('click', (e) => {
          let cl = State.clients.get(e.target.dataset.balance);
          if (cl) {
            let newBal = prompt(`Новый баланс для ${cl.name}:`, cl.balance);
            if (newBal !== null && !isNaN(newBal) && parseFloat(newBal) >= 0) {
              cl.balance = parseFloat(newBal); saveState(); renderClientsList();
            }
          }
        }));
      }

      // ========== КУРСЫ ВАЛЮТ ==========
      function renderCurrencyRates() {
        let html = '<div class="grid-2col">';
        for (let [cur, rate] of Object.entries(State.currencyRates)) {
  

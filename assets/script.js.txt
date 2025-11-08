// =========================
// Variabel Global
// =========================
let currentUserType = null;
let currentSection = null;
let formData = []; // data disimpan sementara di memori (hilang kalau refresh)

// =========================
// Inisialisasi Aplikasi
// =========================
document.addEventListener('DOMContentLoaded', initializeApp);

async function initializeApp() {
  setupEventListeners();
  console.log("App initialized (tanpa SDK Canva)");
}

// =========================
// Fungsi Login / Logout
// =========================
function setupEventListeners() {
  document.getElementById('loginForm').addEventListener('submit', handleLogin);

  document.querySelectorAll('.nav-card').forEach(card => {
    card.addEventListener('click', () => {
      navigateToSection(card.dataset.section);
    });
  });
}

function handleLogin(event) {
  event.preventDefault();

  const username = document.getElementById('username').value.trim();
  const password = document.getElementById('password').value.trim();

  // contoh login dummy
  if ((username === 'admin' && password === '1234') || (username === 'user' && password === '1234')) {
    currentUserType = username === 'admin' ? 'admin' : 'user';
    document.getElementById('loginScreen').style.display = 'none';
    document.getElementById('mainScreen').classList.remove('hidden');
    document.getElementById('userSubtitle').textContent = `Login sebagai ${currentUserType.toUpperCase()}`;
  } else {
    alert('Username atau password salah.');
  }
}

function logout() {
  currentUserType = null;
  document.getElementById('mainScreen').classList.add('hidden');
  document.getElementById('loginScreen').style.display = 'flex';
  document.getElementById('loginForm').reset();
  formData = [];
  document.getElementById('contentArea').innerHTML = '';
  document.querySelectorAll('.nav-card').forEach(card => card.classList.remove('active'));
}

// =========================
// Navigasi antar bagian
// =========================
function navigateToSection(section) {
  currentSection = section;
  document.querySelectorAll('.nav-card').forEach(card => {
    card.classList.remove('active');
    if (card.dataset.section === section) card.classList.add('active');
  });

  const content = document.getElementById('contentArea');

  switch (section) {
    case 'escalator':
      content.innerHTML = renderForm('Safety Eskalator');
      break;
    case 'elevator':
      content.innerHTML = renderForm('Safety Elevator');
      break;
    case 'history':
      content.innerHTML = renderHistory();
      break;
    default:
      content.innerHTML = '';
  }

  setupDynamicForm();
}

// =========================
// Form Generator
// =========================
function renderForm(title) {
  return `
    <div class="form-section">
      <h2 class="form-title">${title}</h2>
      <form id="safetyForm">
        <div class="form-group">
          <label class="form-label">Nama Lokasi</label>
          <input type="text" id="locationName" class="form-input" placeholder="Masukkan nama lokasi" required>
        </div>
        <div class="form-group">
          <label class="form-label">Nama Teknisi</label>
          <input type="text" id="technicianName" class="form-input" placeholder="Masukkan nama teknisi" required>
        </div>
        <div class="form-group">
          <label class="form-label">Tanggal Pemeriksaan</label>
          <input type="date" id="inspectionDate" class="form-input" required>
        </div>
        <div class="form-group">
          <label class="form-label">Catatan Pemeriksaan</label>
          <textarea id="inspectionNotes" class="form-input" placeholder="Masukkan catatan" rows="4"></textarea>
        </div>
        <button type="submit" class="btn-primary">Simpan Data</button>
      </form>
    </div>
  `;
}

function setupDynamicForm() {
  const form = document.getElementById('safetyForm');
  if (!form) return;

  form.addEventListener('submit', e => {
    e.preventDefault();
    const newEntry = {
      id: Date.now(),
      type: currentSection,
      location: document.getElementById('locationName').value.trim(),
      technician: document.getElementById('technicianName').value.trim(),
      date: document.getElementById('inspectionDate').value,
      notes: document.getElementById('inspectionNotes').value.trim(),
      status: 'pending'
    };

    formData.push(newEntry);
    showMessage("Data berhasil disimpan (sementara).", "success");
    form.reset();
  });
}

// =========================
// Halaman History
// =========================
function renderHistory() {
  if (formData.length === 0) {
    return `<p class="text-center text-gray-300">Belum ada data tersimpan.</p>`;
  }

  const items = formData.map(item => `
    <div class="history-item">
      <h3>${item.location} (${item.type})</h3>
      <p><strong>Teknisi:</strong> ${item.technician}</p>
      <p><strong>Tanggal:</strong> ${item.date}</p>
      <p><strong>Status:</strong> ${item.status}</p>
      <p><strong>Catatan:</strong> ${item.notes}</p>
      ${currentUserType === 'admin' && item.status === 'pending'
        ? `<button class="btn-approve" onclick="approveData(${item.id})">Approve</button>`
        : ''}
    </div>
  `).join('');

  return `<div class="form-section"><h2>History Pemeriksaan</h2>${items}</div>`;
}

// =========================
// Fungsi Admin Approve
// =========================
async function approveData(itemId) {
  if (currentUserType !== 'admin') return;

  const item = formData.find(d => d.id === itemId);
  if (!item) return;

  item.status = 'approved';
  showMessage("Data berhasil di-approve!", "success");
  navigateToSection('history');
}

// =========================
// Fungsi Utility
// =========================
function showMessage(msg, type = "info") {
  const color = type === "success" ? "#00e676" : type === "error" ? "#ff1744" : "#2196f3";
  const messageBox = document.createElement('div');
  messageBox.textContent = msg;
  messageBox.style.position = 'fixed';
  messageBox.style.bottom = '20px';
  messageBox.style.right = '20px';
  messageBox.style.background = color;
  messageBox.style.color = 'white';
  messageBox.style.padding = '12px 20px';
  messageBox.style.borderRadius = '8px';
  messageBox.style.zIndex = '9999';
  messageBox.style.boxShadow = '0 4px 12px rgba(0,0,0,0.3)';
  document.body.appendChild(messageBox);
  setTimeout(() => messageBox.remove(), 2500);
}

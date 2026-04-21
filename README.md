<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Çakan Lojistik | Gelişmiş Ekipman Yönetimi</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    
    <style>
        :root { --primary: #1a237e; --success: #2e7d32; --danger: #c62828; --warning: #ff8f00; }
        body { font-family: 'Inter', sans-serif; background-color: #f0f2f5; }
        .navbar { background: var(--primary); box-shadow: 0 2px 10px rgba(0,0,0,0.2); }
        .hero { background: linear-gradient(135deg, #1a237e 0%, #0d47a1 100%); color: white; padding: 40px 0; text-align: center; }
        .panel-card { display: none; border: none; border-radius: 12px; box-shadow: 0 5px 15px rgba(0,0,0,0.08); background: white; padding: 30px; margin-top: 20px; }
        .active-panel { display: block !important; animation: fadeIn 0.4s; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        .list-group-item { border-left: 5px solid #ddd; margin-bottom: 8px; border-radius: 8px !important; }
        .entry-border { border-left-color: var(--success); }
        .exit-border { border-left-color: var(--danger); }
        #loginOverlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.85); 
                        display: none; justify-content: center; align-items: center; z-index: 1000; backdrop-filter: blur(5px); }
    </style>
</head>
<body>

<nav class="navbar navbar-expand-lg navbar-dark">
    <div class="container">
        <a class="navbar-brand fw-bold" href="#" onclick="location.reload()">
            <i class="fas fa-warehouse me-2"></i> ÇAKAN DEPO V3.0
        </a>
        <div class="ms-auto">
            <button class="btn btn-sm btn-outline-light me-2" onclick="openLogin('staff')">Personel Girişi</button>
            <button class="btn btn-sm btn-warning" onclick="openLogin('admin')">Admin Paneli</button>
        </div>
    </div>
</nav>

<div id="mainView">
    <div class="hero" id="heroArea">
        <div class="container">
            <h2 class="fw-bold">Konteyner Takip & Ekipman Yönetimi</h2>
            <p>Acente, Ekipman Türü ve Adet Bazlı Operasyon</p>
        </div>
    </div>

    <div class="container pb-5">
        <div id="customerSection" class="panel-card active-panel">
            <div class="row justify-content-center">
                <div class="col-md-8 text-center">
                    <h4 class="mb-4 text-secondary">Hızlı Sorgulama</h4>
                    <div class="input-group mb-4 shadow-sm">
                        <input type="text" id="custSearchInput" class="form-control form-control-lg" placeholder="Konteyner veya Referans No...">
                        <button class="btn btn-primary px-4" onclick="customerSearch()">Sorgula</button>
                    </div>
                    <div id="searchResultArea"></div>
                </div>
            </div>
        </div>

        <div id="staffSection" class="panel-card">
            <div class="d-flex justify-content-between mb-4 border-bottom pb-2 align-items-center">
                <h4 class="text-success fw-bold"><i class="fas fa-user-hard-hat me-2"></i>Personel İşlem</h4>
                <div>
                    <button class="btn btn-dark btn-sm me-2" onclick="downloadExcel()">Excel İndir</button>
                    <button class="btn btn-outline-danger btn-sm" onclick="location.reload()">Sistemden Çık</button>
                </div>
            </div>
            <div class="row">
                <div class="col-md-5">
                    <div class="card p-3 bg-light shadow-sm">
                        <h6>İşlem Kaydı</h6>
                        <hr>
                        <input type="text" id="pContNo" class="form-control mb-2" placeholder="Konteyner No">
                        <select id="pType" class="form-select mb-2">
                            <option value="20'lik">20'lik</option>
                            <option value="40'lık">40'lık</option>
                        </select>
                        <select id="pAction" class="form-select mb-2" onchange="toggleRefField()">
                            <option value="Giriş">Giriş (Boş)</option>
                            <option value="Çıkış">Çıkış (Referanslı)</option>
                        </select>
                        
                        <div id="entryAgencyArea">
                            <input type="text" id="pEntryAgency" class="form-control mb-3" placeholder="Giriş Yapan Acente">
                        </div>

                        <div id="refFieldArea" style="display:none;">
                            <input type="text" id="pRefNo" class="form-control mb-2" placeholder="Referans No Giriniz" oninput="showRefInfo()">
                            <div id="refAgencyInfo" class="alert alert-info py-2 mb-2 small shadow-sm" style="display:none; border-left: 5px solid #0d47a1;"></div>
                        </div>

                        <button class="btn btn-success w-100" onclick="saveTransaction()">İşlemi Kaydet</button>
                    </div>
                </div>
                <div class="col-md-7">
                    <h6>Güncel Hareketler</h6>
                    <div id="staffLogList" class="list-group" style="max-height: 400px; overflow-y: auto;"></div>
                </div>
            </div>
        </div>

        <div id="adminSection" class="panel-card">
            <div class="d-flex justify-content-between mb-4 border-bottom pb-2 align-items-center">
                <h4 class="text-primary fw-bold"><i class="fas fa-user-shield me-2"></i>Admin Yönetimi</h4>
                <div>
                    <button class="btn btn-dark btn-sm me-2" onclick="downloadExcel()">Excel İndir</button>
                    <button class="btn btn-danger btn-sm" onclick="location.reload()">Güvenli Çıkış</button>
                </div>
            </div>
            
            <div class="row mb-4">
                <div class="col-md-4">
                    <div class="card p-3 border-warning shadow-sm">
                        <h6><i class="fas fa-plus-circle text-warning me-2"></i>Referans & Ekipman Tanımla</h6>
                        <input type="text" id="newRefInput" class="form-control mb-2" placeholder="Referans No">
                        <input type="text" id="newAgencyInput" class="form-control mb-2" placeholder="Acente Adı">
                        <select id="newEqType" class="form-select mb-2">
                            <option value="20'lik">20'lik</option>
                            <option value="40'lık">40'lık</option>
                        </select>
                        <input type="number" id="newCountInput" class="form-control mb-2" placeholder="Kaç Adet?" min="1">
                        <button class="btn btn-warning w-100 fw-bold" onclick="addReference()">AKTİF ET</button>
                        <hr>
                        <div id="activeRefsList" class="mt-2"></div>
                    </div>
                </div>
                <div class="col-md-8">
                    <h6>Sistem Hareket Kayıtları</h6>
                    <div class="table-responsive border rounded shadow-sm">
                        <table class="table table-hover mb-0">
                            <thead class="table-dark">
                                <tr>
                                    <th>Tarih</th>
                                    <th>İşlem</th>
                                    <th>Konteyner</th>
                                    <th>Tür</th>
                                    <th>Acente</th>
                                    <th>Referans</th>
                                </tr>
                            </thead>
                            <tbody id="adminTableBody"></tbody>
                        </table>
                    </div>
                    <button class="btn btn-link text-danger btn-sm mt-2" onclick="clearData()">Verileri Sıfırla</button>
                </div>
            </div>
        </div>
    </div>
</div>

<div id="loginOverlay">
    <div class="card p-4 shadow-lg" style="width: 320px;">
        <h5 id="loginTitle" class="text-center mb-3">Sistem Girişi</h5>
        <input type="password" id="passInput" class="form-control mb-3 text-center" placeholder="Şifre">
        <div class="d-flex gap-2">
            <button class="btn btn-light w-100" onclick="closeLogin()">İptal</button>
            <button class="btn btn-primary w-100" onclick="checkAuth()">Giriş</button>
        </div>
    </div>
</div>

<script>
    let transactions = JSON.parse(localStorage.getItem('chakanV3Data')) || [];
    let activeRefs = JSON.parse(localStorage.getItem('activeRefsV3')) || [];

    const PASSWORDS = { staff: '1234', admin: '9999' };
    let currentMode = '';

    function openLogin(mode) { currentMode = mode; document.getElementById('loginOverlay').style.display = 'flex'; }
    function closeLogin() { document.getElementById('loginOverlay').style.display = 'none'; }

    function checkAuth() {
        if(document.getElementById('passInput').value === PASSWORDS[currentMode]) {
            closeLogin();
            loadPanels();
        } else { alert('Hatalı Şifre!'); }
    }

    function loadPanels() {
        document.querySelectorAll('.panel-card').forEach(p => p.classList.remove('active-panel'));
        if(currentMode === 'staff') {
            document.getElementById('staffSection').classList.add('active-panel');
            renderStaffLogs();
        } else {
            document.getElementById('adminSection').classList.add('active-panel');
            renderAdminTable();
            renderActiveRefs();
        }
        document.getElementById('heroArea').style.display = 'none';
    }

    function toggleRefField() {
        const action = document.getElementById('pAction').value;
        document.getElementById('refFieldArea').style.display = (action === 'Çıkış') ? 'block' : 'none';
        document.getElementById('entryAgencyArea').style.display = (action === 'Giriş') ? 'block' : 'none';
    }

    // ADMİN: REFERANS + ACENTE + TÜR + ADET EKLEME
    function addReference() {
        const ref = document.getElementById('newRefInput').value.trim().toUpperCase();
        const agency = document.getElementById('newAgencyInput').value.trim().toUpperCase();
        const type = document.getElementById('newEqType').value;
        const count = document.getElementById('newCountInput').value;
        
        if(!ref || !agency || !count) return alert('Tüm alanları doldurun!');
        
        activeRefs.push({ ref, agency, type, count });
        localStorage.setItem('activeRefsV3', JSON.stringify(activeRefs));
        
        document.getElementById('newRefInput').value = '';
        document.getElementById('newAgencyInput').value = '';
        document.getElementById('newCountInput').value = '';
        renderActiveRefs();
    }

    function renderActiveRefs() {
        const area = document.getElementById('activeRefsList');
        area.innerHTML = '<h6>Aktif Referanslar:</h6>' + activeRefs.map(r => 
            `<div class="alert alert-warning py-1 px-2 mb-1 small" style="font-size: 11px;">
                <b>${r.ref}</b> | ${r.agency}<br>Tür: ${r.type} | Adet: ${r.count}
            </div>`).join('');
    }

    // PERSONEL: REF YAZINCA TÜM BİLGİLERİ GÖSTER
    function showRefInfo() {
        const refInput = document.getElementById('pRefNo').value.trim().toUpperCase();
        const infoDiv = document.getElementById('refAgencyInfo');
        const match = activeRefs.find(r => r.ref === refInput);

        if(match) {
            infoDiv.style.display = 'block';
            infoDiv.innerHTML = `
                <i class="fas fa-info-circle"></i> <b>ACENTE:</b> ${match.agency}<br>
                <i class="fas fa-truck-container"></i> <b>ONAYLI TÜR:</b> ${match.type}<br>
                <i class="fas fa-sort-numeric-up"></i> <b>ONAYLI ADET:</b> ${match.count}`;
        } else {
            infoDiv.style.display = 'none';
        }
    }

    // PERSONEL: KAYIT
    function saveTransaction() {
        const cont = document.getElementById('pContNo').value.trim().toUpperCase();
        const eqType = document.getElementById('pType').value;
        const action = document.getElementById('pAction').value;
        const now = new Date().toLocaleString('tr-TR');
        let agency = "";
        let ref = "-";

        if(!cont) return alert('Konteyner No giriniz!');

        if(action === 'Giriş') {
            agency = document.getElementById('pEntryAgency').value.trim().toUpperCase();
            if(!agency) return alert('Giriş acentesini yazın!');
        } else {
            ref = document.getElementById('pRefNo').value.trim().toUpperCase();
            const match = activeRefs.find(r => r.ref === ref);
            if(!match) return alert('HATA: Bu referans Admin onayında yok!');
            
            // Opsiyonel: Tür kontrolü uyarısı
            if(match.type !== eqType) {
                if(!confirm(`DİKKAT: Admin bu referans için ${match.type} onayı vermiş. Siz ${eqType} seçtiniz. Devam edilsin mi?`)) return;
            }
            agency = match.agency;
        }

        transactions.unshift({ cont, eqType, ref, action, agency, date: now });
        localStorage.setItem('chakanV3Data', JSON.stringify(transactions));
        
        alert('İşlem Başarılı!');
        renderStaffLogs();
        document.getElementById('pContNo').value = '';
        document.getElementById('pRefNo').value = '';
        document.getElementById('pEntryAgency').value = '';
        document.getElementById('refAgencyInfo').style.display = 'none';
    }

    function renderStaffLogs() {
        const list = document.getElementById('staffLogList');
        list.innerHTML = transactions.map(t => `
            <div class="list-group-item ${t.action === 'Giriş' ? 'entry-border' : 'exit-border'} shadow-sm">
                <div class="d-flex justify-content-between">
                    <strong>${t.cont} (${t.eqType})</strong>
                    <span class="badge ${t.action === 'Giriş' ? 'bg-success' : 'bg-danger'}">${t.action}</span>
                </div>
                <div class="small">Acente: <b>${t.agency}</b> | Ref: ${t.ref}</div>
                <small class="text-muted">${t.date}</small>
            </div>
        `).join('');
    }

    function renderAdminTable() {
        const body = document.getElementById('adminTableBody');
        body.innerHTML = transactions.map(t => `
            <tr>
                <td><small>${t.date}</small></td>
                <td><span class="badge ${t.action === 'Giriş' ? 'bg-success' : 'bg-danger'}">${t.action}</span></td>
                <td><b>${t.cont}</b></td>
                <td>${t.eqType}</td>
                <td>${t.agency}</td>
                <td>${t.ref}</td>
            </tr>
        `).join('');
    }

    function downloadExcel() {
        if(transactions.length === 0) return alert('Veri yok.');
        const worksheet = XLSX.utils.json_to_sheet(transactions);
        const workbook = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(workbook, worksheet, "Rapor");
        XLSX.writeFile(workbook, `Cakan_Depo_Rapor.xlsx`);
    }

    function customerSearch() {
        const input = document.getElementById('custSearchInput').value.trim().toUpperCase();
        const resultArea = document.getElementById('searchResultArea');
        if(!input) return;

        const found = transactions.find(t => t.cont === input || t.ref === input);

        if(found) {
            resultArea.innerHTML = `
                <div class="card p-4 border-0 shadow-lg text-start" style="border-left: 10px solid #1a237e !important;">
                    <h5 class="text-primary">Hareket Bilgisi</h5>
                    <div class="row">
                        <div class="col-6"><b>Konteyner:</b><br>${found.cont}</div>
                        <div class="col-6"><b>Tür:</b><br>${found.eqType}</div>
                        <div class="col-6 mt-2"><b>Acente:</b><br>${found.agency}</div>
                        <div class="col-6 mt-2"><b>Durum:</b><br>${found.action}</div>
                        <div class="col-12 mt-2"><b>Referans:</b> ${found.ref}</div>
                        <div class="col-12 text-muted small mt-1">İşlem Tarihi: ${found.date}</div>
                    </div>
                </div>`;
        } else {
            resultArea.innerHTML = `<div class="alert alert-danger shadow">Kayıt bulunamadı. Lütfen numarayı kontrol edin.</div>`;
        }
    }

    function clearData() { if(confirm('Tüm veriler silinsin mi?')) { localStorage.clear(); location.reload(); } }
</script>

</body>
</html>

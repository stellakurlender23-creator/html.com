<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Escale - Voyage & Partage</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Leaflet CSS pour la carte -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <!-- PWA Manifest -->
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#0284c7">
    <style>
        body { padding-bottom: 5rem; }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 font-sans">

    <!-- En-tête -->
    <header class="bg-sky-600 text-white p-4 shadow-md sticky top-0 z-50">
        <h1 class="text-xl font-bold text-center">✈️ Escale</h1>
    </header>

    <!-- Contenu des onglets -->
    <main class="p-4 max-w-md mx-auto">

        <!-- ONGLET 1 : FRAIS -->
        <section id="tab-frais" class="tab-content space-y-4">
            <div class="bg-white p-4 rounded-xl shadow-sm">
                <h2 class="font-bold text-lg mb-3 text-sky-700">1. Participants</h2>
                <div class="flex gap-2 mb-2">
                    <input type="text" id="participant-input" placeholder="Prénom..." class="border p-2 rounded-lg flex-1 text-sm">
                    <button onclick="addParticipant()" class="bg-sky-600 text-white px-4 py-2 rounded-lg text-sm font-semibold">Ajouter</button>
                </div>
                <div id="participants-list" class="flex flex-wrap gap-2 mt-2"></div>
            </div>

            <div class="bg-white p-4 rounded-xl shadow-sm">
                <h2 class="font-bold text-lg mb-3 text-sky-700">2. Ajouter une dépense</h2>
                <form id="expense-form" onsubmit="addExpense(event)" class="space-y-3">
                    <input type="text" id="expense-title" placeholder="Ex : Airbnb, Resto..." required class="border p-2 rounded-lg w-full text-sm">
                    <input type="number" step="0.01" id="expense-amount" placeholder="Montant en €" required class="border p-2 rounded-lg w-full text-sm">
                    <select id="expense-payer" required class="border p-2 rounded-lg w-full text-sm"></select>
                    <button type="submit" class="w-full bg-emerald-600 text-white py-2 rounded-lg text-sm font-semibold">Enregistrer la dépense</button>
                </form>
            </div>

            <div class="bg-white p-4 rounded-xl shadow-sm">
                <h2 class="font-bold text-lg mb-3 text-sky-700">3. Équilibre des comptes</h2>
                <div id="balances-list" class="text-sm space-y-2"></div>
            </div>
        </section>

        <!-- ONGLET 2 : CARTE -->
        <section id="tab-carte" class="tab-content hidden space-y-3">
            <div class="bg-white p-3 rounded-xl shadow-sm">
                <p class="text-xs text-slate-500 mb-2">Hébergements autour de vous (Exemple : Lisbonne)</p>
                <div id="map" class="w-full h-96 rounded-lg z-10"></div>
            </div>
        </section>

        <!-- ONGLET 3 : RENCONTRES -->
        <section id="tab-rencontres" class="tab-content hidden space-y-4">
            <div class="bg-white p-4 rounded-xl shadow-sm">
                <h2 class="font-bold text-lg mb-3 text-sky-700">Mon profil voyageur</h2>
                <input type="text" id="my-name" placeholder="Ton prénom" class="border p-2 rounded-lg w-full text-sm mb-2">
                <input type="text" id="my-city" placeholder="Ville actuelle" class="border p-2 rounded-lg w-full text-sm mb-2">
                <button onclick="saveProfile()" class="w-full bg-sky-600 text-white py-2 rounded-lg text-sm font-semibold">Mettre à jour</button>
            </div>

            <div class="space-y-3">
                <h2 class="font-bold text-lg text-sky-700">Voyageurs dans la même ville</h2>
                <div id="travelers-list" class="space-y-2">
                    <div class="bg-white p-3 rounded-xl shadow-sm flex justify-between items-center">
                        <div>
                            <p class="font-semibold text-sm">Lucas, 25 ans</p>
                            <p class="text-xs text-slate-500">📍 Lisbonne • Aime la rando et les technos</p>
                        </div>
                        <button onclick="alert('Message envoyé à Lucas !')" class="bg-sky-100 text-sky-700 px-3 py-1.5 rounded-lg text-xs font-semibold">Dire bonjour</button>
                    </div>
                </div>
            </div>
        </section>

    </main>

    <!-- Barre de navigation fixe -->
    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t border-slate-200 flex justify-around p-3 shadow-lg z-50">
        <button onclick="switchTab('frais')" id="btn-frais" class="text-sky-600 font-semibold text-xs flex flex-col items-center">
            <span>💶</span> Frais
        </button>
        <button onclick="switchTab('carte')" id="btn-carte" class="text-slate-400 font-semibold text-xs flex flex-col items-center">
            <span>🗺️</span> Carte
        </button>
        <button onclick="switchTab('rencontres')" id="btn-rencontres" class="text-slate-400 font-semibold text-xs flex flex-col items-center">
            <span>👋</span> Rencontres
        </button>
    </nav>

    <!-- Scripts Leaflet et Logique -->
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
        // --- NAVIGATION ---
        function switchTab(tabName) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.getElementById(`tab-${tabName}`).classList.remove('hidden');

            document.querySelectorAll('nav button').forEach(el => {
                el.classList.remove('text-sky-600');
                el.classList.add('text-slate-400');
            });
            document.getElementById(`btn-${tabName}`).classList.remove('text-slate-400');
            document.getElementById(`btn-${tabName}`).classList.add('text-sky-600');

            if (tabName === 'carte' && window.mapInitialized పోలీసు !== true) {
                initMap();
            }
        }

        // --- GESTION DES FRAIS ---
        let participants = JSON.parse(localStorage.getItem('escale_participants')) || ["Alice", "Thomas"];
        let expenses = JSON.parse(localStorage.getItem('escale_expenses')) || [];

        function saveData() {
            localStorage.setItem('escale_participants', JSON.stringify(participants));
            localStorage.setItem('escale_expenses', JSON.stringify(expenses));
            renderFrais();
        }

        function addParticipant() {
            const nameInput = document.getElementById('participant-input');
            const name = nameInput.value.trim();
            if (name && !participants.includes(name)) {
                participants.push(name);
                nameInput.value = '';
                saveData();
            }
        }

        function addExpense(e) {
            e.preventDefault();
            const title = document.getElementById('expense-title').value;
            const amount = parseFloat(document.getElementById('expense-amount').value);
            const payer = document.getElementById('expense-payer').value;

            if (participants.length === 0) {
                alert("Ajoute au moins un participant !");
                return;
            }

            expenses.push({ title, amount, payer });
            document.getElementById('expense-form').reset();
            saveData();
        }

        function renderFrais() {
            // Rendu participants
            const pList = document.getElementById('participants-list');
            pList.innerHTML = participants.map(p => `<span class="bg-sky-100 text-sky-800 px-3 py-1 rounded-full text-xs font-medium">${p}</span>`).join('');

            // Rendu select payeur
            const payerSelect = document.getElementById('expense-payer');
            payerSelect.innerHTML = participants.map(p => `<option value="${p}">${p}</option>`).join('');

            // Calcul des soldes
            let balances = {};
            participants.forEach(p => balances[p] = 0);
            let totalGeneral = 0;

            expenses.forEach(exp => {
                totalGeneral += exp.amount;
                balances[exp.payer] += exp.amount;
            });

            const partShare = participants.length > 0 ? totalGeneral / participants.length : 0;
            const bList = document.getElementById('balances-list');
            
            if (participants.length === 0) {
                bList.innerHTML = '<p class="text-slate-400">Ajoute des participants pour commencer.</p>';
                return;
            }

            let html = `<p class="font-semibold text-slate-600 mb-2">Total dépensé : ${totalGeneral.toFixed(2)} €</p>`;
            participants.forEach(p => {
                let diff = balances[p] - partShare;
                let color = diff >= 0 ? 'text-emerald-600' : 'text-rose-600';
                let text = diff >= 0 ? `a trop payé de +${diff.doit = diff.toFixed(2)} €` : `doit rembourser ${Math.abs(diff).toFixed(2)} €`;
                html += `<div class="flex justify-between border-b py-1"><span>${p}</span><span class="${color} font-semibold">${text}</span></div>`;
            });
            bList.innerHTML = html;
        }

        // --- CARTE LEAFLET ---
        let map;
        function initMap() {
            window.mapInitialized = true;
            map = L.map('map').setView([38.7169, -9.1399], 13); // Lisbonne par défaut
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '© OpenStreetMap'
            }).addTo(map);

            // Exemples d'hébergements
            const places = [
                { name: "Goodmorning Hostel", lat: 38.7138, lng: -9.1394, type: "Auberge" },
                { name: "The Vintage Hotel", lat: 38.7185, lng: -9.1490, type: "Hôtel" }
            ];

            places.forEach(p => {
                L.marker([p.lat, p.lng]).addTo(map)
                  .bindPopup(`<b>${p.name}</b><br>${p.type}`);
            });
        }

        // --- PROFIL ---
        function saveProfile() {
            const name = document.getElementById('my-name').value;
            if(name) alert(`Profil de ${name} enregistré !`);
        }

        // Initialisation au chargement
        renderFrais();
    </script>
</body>
</html>
# html.com
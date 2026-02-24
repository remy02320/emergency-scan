<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR SAFE - Urgence Médicale</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
</head>
<body class="bg-gray-50 flex items-center justify-center min-h-screen p-4">

    <div id="main-card" class="bg-white shadow-2xl rounded-3xl overflow-hidden max-w-md w-full hidden border-t-8 border-red-600">
        <div class="bg-red-50 p-6 text-center border-b border-red-100">
            <i class="fa-solid fa-truck-medical text-4xl text-red-600 mb-2"></i>
            <h1 class="text-2xl font-black text-gray-800 uppercase tracking-tight">Fiche de Secours</h1>
            <p class="text-red-600 font-bold italic">Informations Vitales</p>
        </div>

        <div class="p-6 space-y-6">
            <div>
                <label class="text-xs font-bold text-gray-400 uppercase">Titulaire</label>
                <p id="userName" class="text-xl font-bold text-gray-800 uppercase tracking-wide">---</p>
            </div>

            <div class="flex items-center bg-red-600 p-4 rounded-2xl shadow-lg">
                <i class="fa-solid fa-droplet text-white text-2xl mr-4"></i>
                <div>
                    <label class="text-xs font-bold text-red-100 uppercase">Groupe Sanguin</label>
                    <p id="bloodGroup" class="text-2xl font-black text-white">---</p>
                </div>
            </div>

            <div class="bg-blue-50 p-4 rounded-2xl border border-blue-100">
                <label class="text-xs font-bold text-blue-400 uppercase">Contact d'Urgence (ICE)</label>
                <p id="contact" class="text-lg font-bold text-blue-900">---</p>
                <a id="callLink" href="#" class="mt-2 inline-flex items-center text-blue-700 font-bold">
                    <i class="fa-solid fa-phone-flip mr-2"></i> Appeler le contact
                </a>
            </div>

            <div>
                <label class="text-xs font-bold text-gray-400 uppercase">Antécédents & Allergies</label>
                <div class="mt-1 p-4 bg-gray-100 rounded-xl border-l-4 border-gray-300">
                    <p id="affections" class="text-gray-700 leading-relaxed font-medium">---</p>
                </div>
            </div>
        </div>

        <div class="p-6 bg-gray-50 text-center text-xs text-gray-400 italic">
            Données transmises via l'application QR SAFE v2
        </div>
    </div>

    <div id="status" class="text-center">
        <i id="status-icon" class="fa-solid fa-circle-notch fa-spin text-4xl text-red-600 mb-4"></i>
        <p id="status-text" class="text-gray-500 font-bold">Recherche de la fiche médicale...</p>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-app.js";
        import { getFirestore, collection, query, where, getDocs } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyA_xkC72EPWgilaWOnE132xjAmlWW5NcO8",
            authDomain: "qrsafe-v2.firebaseapp.com",
            projectId: "qrsafe-v2",
            storageBucket: "qrsafe-v2.firebasestorage.app",
            messagingSenderId: "820028311316",
            appId: "1:820028311316:web:473cb45b919060c6d72401"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        // Récupérer l'ID du QR Code dans l'URL (?id=MED-XXXX)
        const params = new URLSearchParams(window.location.search);
        const qrId = params.get('id');

        async function fetchMedicalData() {
            if (!qrId) {
                updateStatus("Erreur : Aucun code détecté.", "fa-circle-exclamation");
                return;
            }

            try {
                // On cherche dans la collection 'medicalFiles' la fiche qui contient ce qrId dans son tableau 'linkedQRs'
                const q = query(collection(db, "medicalFiles"), where("linkedQRs", "array-contains-any", [{id: qrId, label: "Sticker"}, {id: qrId, label: "Casque VTT"}, {id: qrId, label: "Bracelet"}]));
                
                // Note : Pour faire simple pendant le pitch, on va chercher tous les docs et filtrer en JS 
                // si la requête complexe array-contains-any pose souci avec les objets imbriqués.
                const querySnapshot = await getDocs(collection(db, "medicalFiles"));
                let foundFiche = null;

                querySnapshot.forEach((doc) => {
                    const data = doc.data();
                    if (data.linkedQRs && data.linkedQRs.some(qr => qr.id === qrId)) {
                        foundFiche = data;
                    }
                });

                if (foundFiche) {
                    document.getElementById('userName').innerText = foundFiche.name;
                    document.getElementById('bloodGroup').innerText = foundFiche.bloodGroup;
                    document.getElementById('contact').innerText = foundFiche.contactUrgence;
                    document.getElementById('affections').innerText = foundFiche.affections;
                    
                    // Extraire le numéro pour l'appel
                    const phone = foundFiche.contactUrgence.replace(/\D/g,'');
                    document.getElementById('callLink').href = "tel:" + phone;

                    document.getElementById('status').classList.add('hidden');
                    document.getElementById('main-card').classList.remove('hidden');
                } else {
                    updateStatus("Fiche introuvable ou code non activé.", "fa-ghost");
                }
            } catch (error) {
                console.error(error);
                updateStatus("Erreur de connexion à la base de données.", "fa-wifi");
            }
        }

        function updateStatus(text, icon) {
            document.getElementById('status-text').innerText = text;
            document.getElementById('status-icon').className = `fa-solid ${icon} text-4xl text-gray-400 mb-4`;
        }

        fetchMedicalData();
    </script>
</body>
</html>

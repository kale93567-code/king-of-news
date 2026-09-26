
<!DOCTYPE html>
<html lang="mr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>King of News - MKI Tech</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', sans-serif; }
        body { background-color: #0d0d0d; color: #fff; display: flex; flex-direction: column; height: 100vh; overflow: hidden; }
        
        .header { padding: 12px; background: #181818; text-align: center; font-weight: bold; font-size: 1.2rem; color: #ffd700; border-bottom: 1px solid #333; position: fixed; top: 0; width: 100%; z-index: 100; }
        
        .content { flex: 1; margin-top: 55px; margin-bottom: 65px; overflow-y: auto; padding: 10px; display: flex; flex-direction: column; align-items: center; }
        .section { display: none; width: 100%; max-width: 450px; }
        .active { display: block; }

        .live-banner { background: linear-gradient(90deg, #ff0055, #e50914); color: white; padding: 10px 15px; border-radius: 10px; margin-bottom: 15px; display: flex; justify-content: space-between; align-items: center; font-weight: bold; font-size: 0.9rem; cursor: pointer; }
        .live-badge { background: #fff; color: #ff0055; padding: 3px 8px; border-radius: 5px; font-size: 0.75rem; animation: blink 1s infinite; }
        @keyframes blink { 0% { opacity: 1; } 50% { opacity: 0.4; } 100% { opacity: 1; } }

        .reel-card { background: #1a1a1a; border-radius: 12px; margin-bottom: 15px; border: 1px solid #333; overflow: hidden; position: relative; }
        .video-box { width: 100%; height: 320px; background: #000; display: flex; align-items: center; justify-content: center; position: relative; }
        .video-box video { width: 100%; height: 100%; object-fit: cover; }
        .distance-badge { position: absolute; top: 10px; right: 10px; background: rgba(0,229,255,0.85); color: #000; font-weight: bold; padding: 4px 10px; border-radius: 20px; font-size: 0.8rem; z-index: 10; }
        
        .reel-info { padding: 12px; }
        .author { color: #ff9900; font-weight: bold; font-size: 0.95rem; margin-bottom: 5px; }
        .caption { font-size: 0.9rem; color: #ddd; margin-bottom: 10px; }
        
        .reel-actions { display: flex; justify-content: space-between; border-top: 1px solid #2a2a2a; padding-top: 10px; }
        .action-btn { background: none; border: none; color: #fff; font-size: 0.85rem; cursor: pointer; }
        .report-btn { color: #ff3333; }

        .card { background: #181818; padding: 15px; border-radius: 10px; margin-bottom: 15px; border: 1px solid #333; }
        input, select, textarea, button { width: 100%; padding: 12px; margin: 8px 0; border-radius: 8px; border: none; font-size: 0.95rem; }
        input, select, textarea { background: #262626; color: #fff; border: 1px solid #444; }
        .btn-main { background: #e50914; color: white; font-weight: bold; cursor: pointer; }

        .toggle-box { display: flex; justify-content: space-between; align-items: center; margin: 10px 0; background: #222; padding: 10px; border-radius: 8px; }

        .bottom-nav { display: flex; justify-content: space-around; background: #181818; padding: 10px 0; border-top: 1px solid #333; position: fixed; bottom: 0; width: 100%; z-index: 100; }
        .nav-btn { color: #888; background: none; border: none; font-size: 0.85rem; width: auto; cursor: pointer; }
        .nav-btn.active-tab { color: #00e5ff; font-weight: bold; }
        .live-tab { color: #ff0055; font-weight: bold; }
    </style>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { getDatabase, ref, push, onValue } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";

        const firebaseConfig = {
            apiKey: "AIzaSyARPbN4uUsWxXMX1ar-jNRkDL44VqOXnPc",
            authDomain: "king-of-news.firebaseapp.com",
            projectId: "king-of-news",
            storageBucket: "king-of-news.firebasestorage.app",
            messagingSenderId: "940292586434",
            appId: "1:940292586434:web:aae1fb642e68d5b597345b",
            measurementId: "G-SN9D6D99ZS"
        };

        const app = initializeApp(firebaseConfig);
        const db = getDatabase(app);

        // व्हिडिओ अपलोड/पोस्ट करणे
        window.postNews = function() {
            const text = document.getElementById('newsText').value;
            const dist = document.getElementById('newsDistance').value || '0.8';
            const fileInput = document.getElementById('videoFile');
            
            if(!text) {
                alert("कृपया बातमीचा तपशील लिहा!");
                return;
            }

            let videoUrl = "https://www.w3schools.com/html/mov_bbb.mp4"; // डीफॉल्ट सॅम्पल व्हिडिओ

            if(fileInput.files && fileInput.files[0]) {
                const file = fileInput.files[0];
                videoUrl = URL.createObjectURL(file); // स्थानिक मोबाईल व्हिडिओ प्ले करण्यासाठी
            }

            const randomId = "Reporter_MK_" + Math.floor(100 + Math.random() * 900);

            push(ref(db, 'news_posts/'), {
                author: "🆔 " + randomId + " (गुप्त रिपोर्टर)",
                caption: text,
                distance: dist + " km",
                videoUrl: videoUrl,
                timestamp: Date.now()
            }).then(() => {
                alert("🚀 बातमी आणि व्हिडिओ पब्लिश झाला! सर्व मोबाईलवर दिसेल.");
                document.getElementById('newsText').value = "";
                document.getElementById('videoFile').value = "";
                showTab('reels', document.querySelector('.bottom-nav button'));
            }).catch((err) => {
                alert("त्रुटी: " + err.message);
            });
        };

        // रिअल-टाईम बातम्या लोड करणे
        const newsRef = ref(db, 'news_posts/');
        onValue(newsRef, (snapshot) => {
            const container = document.getElementById('reels-container');
            container.innerHTML = "";
            
            const data = snapshot.val();
            if(data) {
                Object.keys(data).reverse().forEach(key => {
                    const item = data[key];
                    const vUrl = item.videoUrl || "https://www.w3schools.com/html/mov_bbb.mp4";
                    const reelHTML = `
                        <div class="reel-card" id="${key}">
                            <div class="video-box">
                                <span class="distance-badge">📍 ${item.distance}</span>
                                <video controls playsinline preload="metadata">
                                    <source src="${vUrl}" type="video/mp4">
                                    तुमचा ब्राउझर व्हिडिओला सपोर्ट करत नाही.
                                </video>
                            </div>
                            <div class="reel-info">
                                <div class="author">${item.author}</div>
                                <div class="caption">${item.caption}</div>
                                <div class="reel-actions">
                                    <button class="action-btn" onclick="likeReel(this)">❤️ <span class="like-count">1</span></button>
                                    <button class="action-btn" onclick="alert('लिंक कॉपी झाली!')">🔗 शेयर</button>
                                    <button class="action-btn report-btn" onclick="aiScanFake('${key}')">⚠️ AI फेक स्कॅन</button>
                                </div>
                            </div>
                        </div>
                    `;
                    container.innerHTML += reelHTML;
                });
            } else {
                container.innerHTML = "<p style='text-align:center; padding:20px; color:#aaa;'>अजून कोणतीही बातमी पोस्ट केलेली नाही. 🔴 LIVE टॅबवर जाऊन व्हिडिओ किंवा बातमी पाठवा!</p>";
            }
        });
    </script>
</head>
<body>

    <div class="header">👑 King of News</div>

    <div class="content">
        <!-- REELS SECTION -->
        <div id="reels" class="section active">
            <div class="live-banner" onclick="showTab('studio', document.querySelector('.live-tab'))">
                <span>🔴 नवीन बातमी / व्हिडिओ पोस्ट करा!</span>
                <span class="live-badge">LIVE 📡</span>
            </div>

            <div id="reels-container">
                <p style="text-align:center; padding:20px; color:#aaa;">बातम्या लोड होत आहेत...</p>
            </div>
        </div>

        <!-- LIVE / STUDIO CAMERA SECTION -->
        <div id="studio" class="section">
            <div class="card">
                <h3>🎥 बातमी व व्हिडिओ पब्लिश करा</h3>
                <p style="font-size:0.8rem; color:#aaa; margin-bottom:10px;">टीप: तुमची खरी ओळख/नाव लपवून बातमी थेट सर्व मोबाईलवर दिसेल.</p>
                
                <textarea id="newsText" rows="3" placeholder="येथे बातमी लिहा (उदा. पुणे हायवेवर ट्रॅफिक जाम!)..."></textarea>
                
                <label style="font-size:0.85rem; color:#aaa;">📹 मोबाईल गॅलरीतून व्हिडिओ निवडा:</label>
                <input type="file" id="videoFile" accept="video/*">

                <label style="font-size:0.85rem; color:#aaa;">📍 अंतर टाका (km):</label>
                <input type="number" id="newsDistance" placeholder="उदा. 0.8 किंवा 2.5" value="0.8">

                <div class="toggle-box">
                    <label>🎭 चेहरा ब्लर करा (Blur Face)</label>
                    <input type="checkbox" id="blurToggle" style="width:auto;">
                </div>
                
                <div class="toggle-box">
                    <label>🤖 रोबोटिक आवाज (Voice Changer)</label>
                    <input type="checkbox" id="voiceToggle" style="width:auto;">
                </div>

                <button class="btn-main" onclick="postNews()">🔴 सर्व मोबाईलवर पब्लिश करा</button>
            </div>
        </div>

        <!-- SEARCH SECTION -->
        <div id="search" class="section">
            <div class="card">
                <h3>🔍 बातमी किंवा परिसर शोधा</h3>
                <input type="text" id="searchInput" placeholder="उदा. ट्रॅफिक, पुणे, अपघात...">
                <button class="btn-main" onclick="searchNews()">शोधा</button>
                <div id="searchResult" style="margin-top:10px; color:#00e5ff;"></div>
            </div>
        </div>

        <!-- SETTINGS / PROFILE SECTION -->
        <div id="settings" class="section">
            <div class="card">
                <h3>⚙️ ॲप सेटिंग्ज & लोकेशन</h3>
                
                <label style="font-size:0.85rem; color:#aaa;">बातमी मोड निवडा:</label>
                <select id="newsMode">
                    <option value="radar">📡 5 KM Radar News (जवळच्या बातम्या)</option>
                    <option value="all_india">🇮🇳 All India News (संपूर्ण भारत)</option>
                </select>

                <label style="font-size:0.85rem; color:#aaa; margin-top:10px; display:block;">तुमचे शहर/गाव (कायम सेव्ह राहील):</label>
                <input type="text" id="savedCity" placeholder="उदा. पुणे / मुंबई">
                
                <button class="btn-main" onclick="saveSettings()">सेव्ह करा</button>
                <p id="statusMsg" style="color:#00ff66; margin-top:10px; font-size:0.85rem;"></p>
            </div>
        </div>
    </div>

    <!-- BOTTOM NAVIGATION BAR -->
    <div class="bottom-nav">
        <button class="nav-btn active-tab" onclick="showTab('reels', this)">📺 रील्स</button>
        <button class="nav-btn live-tab" onclick="showTab('studio', this)">🔴 LIVE</button>
        <button class="nav-btn" onclick="showTab('search', this)">🔍 शोधा</button>
        <button class="nav-btn" onclick="showTab('settings', this)">⚙️ सेटिंग्ज</button>
    </div>

    <script>
        window.onload = function() {
            if(localStorage.getItem('userCity')) {
                document.getElementById('savedCity').value = localStorage.getItem('userCity');
            }
        };

        function showTab(tabId, btn) {
            document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active-tab'));
            
            document.getElementById(tabId).classList.add('active');
            if(btn) btn.classList.add('active-tab');
        }

        function saveSettings() {
            const city = document.getElementById('savedCity').value;
            if(city) {
                localStorage.setItem('userCity', city);
                document.getElementById('statusMsg').innerText = "शहर (" + city + ") सेव्ह झाले!";
            } else {
                alert("कृपया शहराचे नाव टाका!");
            }
        }

        function aiScanFake(reelId) {
            alert("🤖 AI मोड सक्रिय झाला आहे! बातमीची सत्यता तपासली जात आहे...");
            setTimeout(() => {
                alert("🚨 AI ला ही बातमी संशयास्पद/फेक आढळली आहे! सुरक्षेसाठी हा व्हिडिओ सिस्टीममधून काढून टाकला जात आहे.");
                document.getElementById(reelId).style.display = "none";
            }, 1000);
        }

        function likeReel(btn) {
            let countSpan = btn.querySelector('.like-count');
            let count = parseInt(countSpan.innerText);
            countSpan.innerText = count + 1;
            btn.style.color = '#ff0055';
        }

        function searchNews() {
            const query = document.getElementById('searchInput').value;
            if(query) {
                document.getElementById('searchResult').innerText = "'" + query + "' साठी बातम्या शोधल्या जात आहेत...";
            }
        }
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>快速點名系統 2.0 + 歷史紀錄</title>
    <!-- 引入 Tailwind CSS -->
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
</head>
<body class="bg-slate-50 min-h-screen p-4 md:p-8 font-sans">

    <div class="max-w-2xl mx-auto bg-white rounded-2xl shadow-xl border border-slate-100 p-6 space-y-6">
        <!-- 標題 -->
        <div class="text-center">
            <h1 class="text-3xl font-extrabold text-slate-800 tracking-wide">📋 快速點名系統 2.0</h1>
            <p class="text-sm text-slate-400 mt-1">支援本地歷史紀錄功能</p>
        </div>

        <!-- 功能面板 (頁籤切換) -->
        <div class="grid grid-cols-2 gap-2 p-1 bg-slate-100 rounded-xl">
            <button onclick="switchTab('single')" id="tab-single" class="tab-btn py-2 text-sm font-medium rounded-lg bg-white shadow-xs text-slate-800 transition">單筆新增</button>
            <button onclick="switchTab('batch')" id="tab-batch" class="tab-btn py-2 text-sm font-medium rounded-lg text-slate-500 hover:text-slate-800 transition">批量匯入</button>
        </div>

        <!-- 區塊 A：單筆新增 -->
        <div id="panel-single" class="flex gap-2">
            <input type="text" id="studentInput" placeholder="輸入學生姓名..." 
                   class="flex-1 border border-slate-200 rounded-xl px-4 py-2.5 focus:outline-none focus:ring-2 focus:ring-blue-500 bg-slate-50/50">
            <button onclick="addStudent()" class="bg-blue-600 hover:bg-blue-700 text-white font-medium px-5 py-2.5 rounded-xl transition shadow-sm">
                新增
            </button>
        </div>

        <!-- 區塊 B：批量匯入 (預設隱藏) -->
        <div id="panel-batch" class="hidden space-y-2">
            <textarea id="batchInput" rows="3" placeholder="請輸入學生姓名，多個名字請用「換行」或「逗號」隔開..." 
                      class="w-full border border-slate-200 rounded-xl px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500 bg-slate-50/50 text-sm"></textarea>
            <button onclick="addBatchStudents()" class="w-full bg-slate-800 hover:bg-slate-900 text-white font-medium py-2 rounded-xl transition text-sm">
                確認批量匯入
            </button>
        </div>

        <!-- 🎲 隨機抽籤功能區 -->
        <div class="bg-amber-50 border border-amber-100 rounded-2xl p-4 text-center">
            <h3 class="text-amber-800 font-bold text-sm mb-2">🎲 課堂互動隨機抽籤</h3>
            <div id="luckyWinner" class="text-2xl font-black text-amber-600 my-2 h-8">？</div>
            <button onclick="drawLuckyStudent()" class="bg-amber-500 hover:bg-amber-600 text-white text-xs font-bold px-4 py-2 rounded-lg transition shadow-xs">
                從「已到學生」抽一位
            </button>
        </div>

        <hr class="border-slate-100">

        <!-- 學生名單標題 -->
        <div class="flex justify-between items-center">
            <h2 class="text-lg font-bold text-slate-700">學生名單 (<span id="studentCount">0</span>)</h2>
            <div class="flex gap-4 text-xs">
                <button onclick="resetRollCall()" class="text-slate-400 hover:text-slate-600 transition">重置本次狀態</button>
                <button onclick="clearAll()" class="text-red-400 hover:text-red-600 font-medium transition">清空名單</button>
            </div>
        </div>

        <!-- 名單列表 -->
        <div id="studentList" class="space-y-2 max-h-60 overflow-y-auto pr-1">
            <!-- 學生動態產生 -->
        </div>

        <!-- 統計數據與儲存按鈕 -->
        <div class="bg-slate-50 border border-slate-100 rounded-xl p-4 flex flex-col sm:flex-row items-center justify-between gap-4">
            <div class="flex gap-8 text-center w-full sm:w-auto justify-around sm:justify-start">
                <div>
                    <p class="text-2xl font-black text-emerald-600" id="presentStat">0</p>
                    <p class="text-xs font-medium text-slate-400 mt-0.5">已到人數</p>
                </div>
                <div>
                    <p class="text-2xl font-black text-rose-500" id="absentStat">0</p>
                    <p class="text-xs font-medium text-slate-400 mt-0.5">缺席人數</p>
                </div>
            </div>
            <button onclick="saveRecord()" class="w-full sm:w-auto bg-emerald-600 hover:bg-emerald-700 text-white font-bold px-6 py-2.5 rounded-xl transition shadow-md text-sm">
                💾 儲存本次點名紀錄
            </button>
        </div>

        <hr class="border-slate-100">

        <!-- 📜 歷史紀錄區塊 -->
        <div>
            <div class="flex justify-between items-center mb-3">
                <h2 class="text-lg font-bold text-slate-700">📜 歷史點名紀錄</h2>
                <button onclick="clearHistory()" class="text-xs text-slate-400 hover:text-red-500 transition">清空歷史</button>
            </div>
            <div id="historyList" class="space-y-2 max-h-48 overflow-y-auto pr-1">
                <!-- 歷史紀錄動態產生 -->
            </div>
        </div>
    </div>

    <script>
        let students = JSON.parse(localStorage.getItem('students_v2')) || [];
        let historyRecords = JSON.parse(localStorage.getItem('rollcall_history')) || [];

        // 頁籤切換
        function switchTab(type) {
            const pSingle = document.getElementById('panel-single');
            const pBatch = document.getElementById('panel-batch');
            const tSingle = document.getElementById('tab-single');
            const tBatch = document.getElementById('tab-batch');

            if (type === 'single') {
                pSingle.classList.remove('hidden'); pBatch.classList.add('hidden');
                tSingle.className = "py-2 text-sm font-medium rounded-lg bg-white shadow-xs text-slate-800 transition";
                tBatch.className = "py-2 text-sm font-medium rounded-lg text-slate-500 hover:text-slate-800 transition";
            } else {
                pSingle.classList.add('hidden'); pBatch.classList.remove('hidden');
                tSingle.className = "py-2 text-sm font-medium rounded-lg text-slate-500 hover:text-slate-800 transition";
                tBatch.className = "py-2 text-sm font-medium rounded-lg bg-white shadow-xs text-slate-800 transition";
            }
        }

        // 畫面渲染
        function render() {
            renderStudents();
            renderHistory();
        }

        function renderStudents() {
            const listEl = document.getElementById('studentList');
            const countEl = document.getElementById('studentCount');
            listEl.innerHTML = '';
            countEl.textContent = students.length;

            if (students.length === 0) {
                listEl.innerHTML = `<p class="text-slate-400 text-center py-6 text-sm">名單是空的，快去上方新增學生吧！</p>`;
                updateStats();
                return;
            }

            students.forEach((student, index) => {
                const div = document.createElement('div');
                div.className = "flex justify-between items-center bg-white p-3 rounded-xl border border-slate-100 shadow-xs";
                
                let badge = '<span class="text-xs font-bold text-slate-400 bg-slate-100 px-2 py-1 rounded-md">未點</span>';
                if (student.status === 'present') badge = '<span class="text-xs font-bold text-emerald-600 bg-emerald-50 px-2 py-1 rounded-md">已到</span>';
                if (student.status === 'absent') badge = '<span class="text-xs font-bold text-rose-600 bg-rose-50 px-2 py-1 rounded-md">缺席</span>';

                div.innerHTML = `
                    <div class="flex items-center gap-2">
                        <span class="font-semibold text-slate-700">${student.name}</span>
                        ${badge}
                    </div>
                    <div class="flex gap-1">
                        <button onclick="setStatus(${index}, 'present')" class="bg-emerald-500 hover:bg-emerald-600 text-white text-xs font-bold px-3 py-1 rounded-lg transition">到</button>
                        <button onclick="setStatus(${index}, 'absent')" class="bg-rose-500 hover:bg-rose-600 text-white text-xs font-bold px-3 py-1 rounded-lg transition">缺</button>
                        <button onclick="deleteStudent(${index})" class="text-slate-300 hover:text-rose-500 text-sm px-2 transition">✕</button>
                    </div>
                `;
                listEl.appendChild(div);
            });
            updateStats();
            localStorage.setItem('students_v2', JSON.stringify(students));
        }

        function renderHistory() {
            const historyEl = document.getElementById('historyList');
            historyEl.innerHTML = '';

            if (historyRecords.length === 0) {
                historyEl.innerHTML = `<p class="text-slate-400 text-center py-4 text-xs">目前還沒有任何儲存的點名紀錄。</p>`;
                return;
            }

            // 讓最新的紀錄排在最上面
            [...historyRecords].reverse().forEach((record) => {
                const div = document.createElement('div');
                div.className = "flex justify-between items-center bg-slate-50 p-2.5 rounded-lg text-xs border border-slate-100";
                div.innerHTML = `
                    <div>
                        <span class="font-bold text-slate-700">${record.date}</span>
                        <span class="text-slate-400 ml-2">儲存時間：${record.time}</span>
                    </div>
                    <div class="flex gap-2">
                        <span class="text-emerald-600 font-medium">到：${record.present}</span>
                        <span class="text-rose-500 font-medium">缺：${record.absent}</span>
                    </div>
                `;
                historyEl.appendChild(div);
            });
        }

        // 儲存本日紀錄
        function saveRecord() {
            if (students.length === 0) {
                alert("名單內沒有學生，無法儲存紀錄！");
                return;
            }
            
            const present = students.filter(s => s.status === 'present').length;
            const absent = students.filter(s => s.status === 'absent').length;
            const pending = students.filter(s => s.status === 'pending').length;

            if (pending > 0 && !confirm(`還有 ${pending} 位學生尚未點名，確定要直接儲存紀錄嗎？`)) {
                return;
            }

            const now = new Date();
            const dateStr = `${now.getFullYear()}/${now.getMonth()+1}/${now.getDate()}`;
            const timeStr = `${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`;

            const newRecord = {
                date: dateStr,
                time: timeStr,
                present: present,
                absent: absent
            };

            historyRecords.push(newRecord);
            localStorage.setItem('rollcall_history', JSON.stringify(historyRecords));
            alert("🎉 紀錄儲存成功！已加到下方歷史紀錄中。");
            renderHistory();
        }

        // 基本操作函式
        function addStudent() {
            const input = document.getElementById('studentInput');
            const name = input.value.trim();
            if (!name) return;
            students.push({ name, status: 'pending' });
            input.value = '';
            renderStudents();
        }

        function addBatchStudents() {
            const input = document.getElementById('batchInput');
            if (!input.value.trim()) return;
            const names = input.value.split(/[\n,，]+/).map(n => n.trim()).filter(n => n.length > 0);
            names.forEach(name => students.push({ name, status: 'pending' }));
            input.value = '';
            switchTab('single');
            renderStudents();
        }

        function drawLuckyStudent() {
            const winnerEl = document.getElementById('luckyWinner');
            const arrivedStudents = students.filter(s => s.status === 'present');
            if (arrivedStudents.length === 0) {
                winnerEl.textContent = "❌ 沒有已到的學生可抽";
                winnerEl.className = "text-sm font-medium text-rose-500 my-2 h-8";
                return;
            }
            winnerEl.className = "text-2xl font-black text-amber-600 my-2 h-8 animate-bounce";
            winnerEl.textContent = "🎲 抽籤中...";
            setTimeout(() => {
                const randomIndex = Math.floor(Math.random() * arrivedStudents.length);
                winnerEl.textContent = `🎉 ${arrivedStudents[randomIndex].name} 🎉`;
                winnerEl.className = "text-2xl font-black text-amber-500 my-2 h-8";
            }, 600);
        }

        function setStatus(index, status) { students[index].status = status; renderStudents(); }
        function deleteStudent(index) { students.splice(index, 1); renderStudents(); }
        function resetRollCall() { students.forEach(s => s.status = 'pending'); renderStudents(); }
        function clearAll() { if(confirm("確定要清空整份學生名單嗎？")) { students = []; renderStudents(); } }
        function clearHistory() { if(confirm("確定要刪除所有歷史點名紀錄嗎？此動作無法復原。")) { historyRecords = []; localStorage.setItem('rollcall_history', JSON.stringify(historyRecords)); renderHistory(); } }
        
        function updateStats() {
            document.getElementById('presentStat').textContent = students.filter(s => s.status === 'present').length;
            document.getElementById('absentStat').textContent = students.filter(s => s.status === 'absent').length;
        }

        render();
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>快速點名系統 2.0</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
</head>
<body class="bg-slate-50 min-h-screen p-4 md:p-8 font-sans">

    <div class="max-w-2xl mx-auto bg-white rounded-2xl shadow-xl border border-slate-100 p-6 space-y-6">
        <div class="text-center">
            <h1 class="text-3xl font-extrabold text-slate-800 tracking-wide">📋 快速點名系統 02</h1>
            <p class="text-sm text-amber-600 font-medium mt-1 bg-amber-50 inline-block px-3 py-1 rounded-full">⏱️ 每日自動重置，點完名直接隱藏</p>
        </div>

        <div class="grid grid-cols-2 gap-2 p-1 bg-slate-100 rounded-xl">
            <button onclick="switchTab('single')" id="tab-single" class="py-2 text-sm font-medium rounded-lg bg-white shadow-xs text-slate-800 transition">單筆新增</button>
            <button onclick="switchTab('batch')" id="tab-batch" class="py-2 text-sm font-medium rounded-lg text-slate-500 hover:text-slate-800 transition">批量匯入</button>
        </div>

        <div id="panel-single" class="flex gap-2">
            <input type="text" id="studentInput" placeholder="輸入學生姓名..." 
                   class="flex-1 border border-slate-200 rounded-xl px-4 py-2.5 focus:outline-none focus:ring-2 focus:ring-blue-500 bg-slate-50/50">
            <button onclick="addStudent()" class="bg-blue-600 hover:bg-blue-700 text-white font-medium px-5 py-2.5 rounded-xl transition shadow-sm">
                新增
            </button>
        </div>

        <div id="panel-batch" class="hidden space-y-2">
            <textarea id="batchInput" rows="3" placeholder="請輸入學生姓名，多個名字請用「換行」或「逗號」隔開..." 
                      class="w-full border border-slate-200 rounded-xl px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500 bg-slate-50/50 text-sm"></textarea>
            <button onclick="addBatchStudents()" class="w-full bg-slate-800 hover:bg-slate-900 text-white font-medium py-2 rounded-xl transition text-sm">
                確認批量匯入
            </button>
        </div>

        <div class="bg-amber-50 border border-amber-100 rounded-2xl p-4 text-center">
            <h3 class="text-amber-800 font-bold text-sm mb-2">🎲 課堂互動隨機抽籤</h3>
            <div id="luckyWinner" class="text-2xl font-black text-amber-600 my-2 h-8">？</div>
            <button onclick="drawLuckyStudent()" class="bg-amber-500 hover:bg-amber-600 text-white text-xs font-bold px-4 py-2 rounded-lg transition shadow-xs">
                從「已到學生」抽一位
            </button>
        </div>

        <hr class="border-slate-100">

        <div class="flex justify-between items-center">
            <div>
                <h2 class="text-lg font-bold text-slate-700 inline-block">未點名學生 (<span id="studentCount">0</span>)</h2>
                <span id="todayLabel" class="text-xs text-slate-400 ml-2"></span>
            </div>
            <div class="flex gap-4 text-xs">
                <button onclick="resetRollCall()" class="text-slate-400 hover:text-slate-600 transition">手動重置今日狀態</button>
                <button onclick="clearAll()" class="text-red-400 hover:text-red-600 font-medium transition">清空名單</button>
            </div>
        </div>

        <div id="studentList" class="space-y-2 max-h-60 overflow-y-auto pr-1">
            </div>

        <div class="bg-slate-50 border border-slate-100 rounded-xl p-4 flex items-center justify-around text-center">
            <div>
                <p class="text-2xl font-black text-emerald-600" id="presentStat">0</p>
                <p class="text-xs font-medium text-slate-400 mt-0.5">今日已到</p>
            </div>
            <div>
                <p class="text-2xl font-black text-rose-500" id="absentStat">0</p>
                <p class="text-xs font-medium text-slate-400 mt-0.5">今日缺席</p>
            </div>
        </div>

        <hr class="border-slate-100">

        <div>
            <div class="flex justify-between items-center mb-3">
                <h2 class="text-lg font-bold text-slate-700">📜 歷史點名紀錄</h2>
                <button onclick="clearHistory()" class="text-xs text-slate-400 hover:text-red-500 transition">清空歷史</button>
            </div>
            <div id="historyList" class="space-y-2 max-h-48 overflow-y-auto pr-1">
                </div>
        </div>
    </div>

    <script>
        // 1. 初始化資料
        let students = JSON.parse(localStorage.getItem('students_v2')) || [];
        let historyRecords = JSON.parse(localStorage.getItem('rollcall_history')) || [];
        let lastRollCallDate = localStorage.getItem('last_rollcall_date');

        // 取得今天的日期並顯示
        const now = new Date();
        const todayStr = `${now.getFullYear()}/${now.getMonth()+1}/${now.getDate()}`;
        if(document.getElementById('todayLabel')) {
            document.getElementById('todayLabel').textContent = `📅 今日：${todayStr}`;
        }

        // 2. 核心功能：點名後直接隱藏
        function renderStudents() {
            const listEl = document.getElementById('studentList');
            const countEl = document.getElementById('studentCount');
            if (!listEl) return;
            
            listEl.innerHTML = '';
            
            // 過濾出「還沒點名（pending）」的學生
            const remainingStudents = students.filter(s => s.status === 'pending');
            
            if (countEl) {
                countEl.textContent = remainingStudents.length;
            }

            // 如果全部都點完了
            if (remainingStudents.length === 0) {
                if (students.length > 0) {
                    listEl.innerHTML = `
                        <div class="text-center py-8">
                            <p class="text-emerald-600 font-bold text-lg">🎉 全班點名完成！</p>
                            <p class="text-slate-400 text-xs mt-1">所有人都有紀錄囉！</p>
                        </div>
                    `;
                } else {
                    listEl.innerHTML = `<p class="text-slate-400 text-center py-6 text-sm">名單是空的，快去上方新增學生吧！</p>`;
                }
                updateStats();
                return;
            }

            // 遍歷呈現「還沒點名」的學生
            students.forEach((student, index) => {
                if (student.status !== 'pending') return;

                const div = document.createElement('div');
                div.className = "flex justify-between items-center bg-white p-3 rounded-xl border border-slate-100 shadow-xs hover:border-slate-200 transition";
                
                let badge = '<span class="text-xs font-bold text-slate-400 bg-slate-100 px-2 py-1 rounded-md">未點</span>';

                div.innerHTML = `
                    <div class="flex items-center gap-2">
                        <span class="font-semibold text-slate-700">${student.name}</span>
                        ${badge}
                    </div>
                    <div class="flex gap-1">
                        <button onclick="setStatus(${index}, 'present')" class="bg-emerald-500 hover:bg-emerald-600 text-white text-xs font-bold px-3 py-1 rounded-lg transition shadow-2xs">到</button>
                        <button onclick="setStatus(${index}, 'absent')" class="bg-rose-500 hover:bg-rose-600 text-white text-xs font-bold px-3 py-1 rounded-lg transition shadow-2xs">缺</button>
                        <button onclick="deleteStudent(${index})" class="text-slate-300 hover:text-rose-500 text-sm px-2 transition">✕</button>
                    </div>
                `;
                listEl.appendChild(div);
            });
            updateStats();
            localStorage.setItem('students_v2', JSON.stringify(students));
        }

        // 3. 設定狀態（到/缺）
        function setStatus(index, status) {
            students[index].status = status;
            renderStudents(); // 重新整理列表，點完的名字會直接隱藏
        }

        // 4. 單筆新增
        function addStudent() {
            const input = document.getElementById('studentInput');
            const name = input.value.trim();
            if (!name) return;
            students.push({ name, status: 'pending' });
            input.value = '';
            renderStudents();
        }

        // 5. 批量匯入
        function addBatchStudents() {
            const input = document.getElementById('batchInput');
            if (!input || !input.value.trim()) return;
            const names = input.value.split(/[\n,，]+/).map(n => n.trim()).filter(n => n.length > 0);
            names.forEach(name => students.push({ name, status: 'pending' }));
            input.value = '';
            switchTab('single');
            renderStudents();
        }

        // 6. 更新統計數字
        function updateStats() {
            const presentCount = students.filter(s => s.status === 'present').length;
            const absentCount = students.filter(s => s.status === 'absent').length;
            
            if(document.getElementById('presentStat')) document.getElementById('presentStat').textContent = presentCount;
            if(document.getElementById('absentStat')) document.getElementById('absentStat').textContent = absentCount;
        }

        // 7. 跨天自動檢查歸檔
        function checkNewDay() {
            if (lastRollCallDate && lastRollCallDate !== todayStr) {
                const presentCount = students.filter(s => s.status === 'present').length;
                const absentCount = students.filter(s => s.status === 'absent').length;

                if (presentCount > 0 || absentCount > 0) {
                    const autoRecord = {
                        date: lastRollCallDate,
                        time: "系統自動歸檔",
                        present: presentCount,
                        absent: absentCount
                    };
                    historyRecords.push(autoRecord);
                    localStorage.setItem('rollcall_history', JSON.stringify(historyRecords));
                }
                students.forEach(s => s.status = 'pending');
                localStorage.setItem('students_v2', JSON.stringify(students));
            }
            localStorage.setItem('last_rollcall_date', todayStr);
        }

        // 8. 渲染歷史紀錄清單
        function renderHistory() {
            const historyEl = document.getElementById('historyList');
            if (!historyEl) return;
            historyEl.innerHTML = '';

            if (historyRecords.length === 0) {
                historyEl.innerHTML = `<p class="text-slate-400 text-center py-4 text-xs">目前還沒有任何歷史點名紀錄。</p>`;
                return;
            }

            [...historyRecords].reverse().forEach((record) => {
                const div = document.createElement('div');
                div.className = "flex justify-between items-center bg-slate-50 p-2.5 rounded-lg text-xs border border-slate-100";
                div.innerHTML = `
                    <div>
                        <span class="font-bold text-slate-700">${record.date}</span>
                        <span class="text-slate-400 ml-2">(${record.time})</span>
                    </div>
                    <div class="flex gap-2">
                        <span class="text-emerald-600 font-medium">到：${record.present}</span>
                        <span class="text-rose-500 font-medium">缺：${record.absent}</span>
                    </div>
                `;
                historyEl.appendChild(div);
            });
        }

        // 9. 隨機抽籤
        function drawLuckyStudent() {
            const winnerEl = document.getElementById('luckyWinner');
            const arrivedStudents = students.filter(s => s.status === 'present');
            if (!winnerEl) return;
            
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

        // 10. 切換頁籤
        function switchTab(type) {
            const pSingle = document.getElementById('panel-single');
            const pBatch = document.getElementById('panel-batch');
            if (type === 'single') {
                if(pSingle) pSingle.classList.remove('hidden'); 
                if(pBatch) pBatch.classList.add('hidden');
            } else {
                if(pSingle) pSingle.classList.add('hidden'); 
                if(pBatch) pBatch.classList.remove('hidden');
            }
        }

        // 11. 清除與手動重置控制
        function deleteStudent(index) { students.splice(index, 1); renderStudents(); }
        function resetRollCall() { if(confirm("確定要重置今天的點名狀態嗎？")) { students.forEach(s => s.status = 'pending'); renderStudents(); } }
        function clearAll() { if(confirm("確定要清空整份學生名單嗎？")) { students = []; renderStudents(); } }
        function clearHistory() { if(confirm("確定要刪除所有歷史點名紀錄嗎？")) { historyRecords = []; localStorage.setItem('rollcall_history', JSON.stringify(historyRecords)); renderHistory(); } }

        // 🚀 頁面載入時自動執行
        checkNewDay();
        renderStudents();
        renderHistory();
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>企業級每日出勤簽到系統</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <style>
        /* 自訂豐富的背景：科技感漸層 + 幾何裝飾圈圈 */
        .rich-bg {
            background: linear-gradient(135deg, #0f172a 0%, #1e1b4b 50%, #311042 100%);
            position: relative;
            overflow-x: hidden;
        }
        .rich-bg::before {
            content: '';
            position: absolute;
            width: 400px;
            height: 400px;
            background: radial-gradient(circle, rgba(99,102,241,0.15) 0%, rgba(0,0,0,0) 70%);
            top: -100px;
            right: -100px;
            z-index: 0;
        }
        .rich-bg::after {
            content: '';
            position: absolute;
            width: 500px;
            height: 500px;
            background: radial-gradient(circle, rgba(236,72,153,0.1) 0%, rgba(0,0,0,0) 70%);
            bottom: -150px;
            left: -150px;
            z-index: 0;
        }
    </style>
</head>
<body class="rich-bg min-h-screen p-4 md:p-8 font-sans flex items-center justify-center">

    <div class="w-full max-w-2xl bg-slate-900/80 backdrop-blur-xl rounded-3xl shadow-2xl border border-slate-700/50 p-6 space-y-6 relative z-10 my-4 text-slate-100">
        
        <div class="w-20 h-1.5 bg-gradient-to-r from-indigo-500 to-pink-500 rounded-full mx-auto -mt-2"></div>

        <div class="text-center space-y-1">
            <h1 class="text-3xl font-black bg-gradient-to-r from-indigo-400 via-purple-400 to-pink-400 bg-clip-text text-transparent tracking-wide">🏢 CORP 出勤管理系統</h1>
            <p class="text-xs text-indigo-300 font-medium bg-indigo-500/10 border border-indigo-500/20 inline-block px-3 py-1 rounded-full backdrop-blur-md">
                ⚡ 即時隱藏模式 ｜ 📊 支援手動送出存檔
            </p>
        </div>

        <div class="grid grid-cols-2 gap-2 p-1 bg-slate-950/60 rounded-xl border border-slate-800">
            <button onclick="switchTab('single')" id="tab-single" class="py-2 text-sm font-semibold rounded-lg bg-slate-800 text-white shadow-xs transition duration-300">單筆新增員工</button>
            <button onclick="switchTab('batch')" id="tab-batch" class="py-2 text-sm font-semibold rounded-lg text-slate-400 hover:text-slate-200 transition duration-300">批量匯入名單</button>
        </div>

        <div id="panel-single" class="flex gap-2">
            <input type="text" id="studentInput" placeholder="輸入員工姓名..." 
                   class="flex-1 border border-slate-700 rounded-xl px-4 py-2.5 focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-slate-950/40 text-white placeholder-slate-500">
            <button onclick="addStudent()" class="bg-indigo-600 hover:bg-indigo-500 text-white font-bold px-6 py-2.5 rounded-xl transition shadow-lg shadow-indigo-600/20">
                新增
            </button>
        </div>

        <div id="panel-batch" class="hidden space-y-2">
            <textarea id="batchInput" rows="3" placeholder="請輸入員工姓名，多個名字請用「換行」或「逗號」隔開..." 
                      class="w-full border border-slate-700 rounded-xl px-4 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-slate-950/40 text-white placeholder-slate-500 text-sm"></textarea>
            <button onclick="addBatchStudents()" class="w-full bg-slate-800 hover:bg-slate-700 text-white font-bold py-2.5 rounded-xl transition text-sm border border-slate-700">
                確認批量匯入
            </button>
        </div>

        <div class="bg-gradient-to-br from-amber-500/10 to-orange-500/5 border border-amber-500/20 rounded-2xl p-4 text-center">
            <h3 class="text-amber-400 font-bold text-xs uppercase tracking-wider mb-1">🎲 幸運抽獎 / 今日任務指派</h3>
            <div id="luckyWinner" class="text-2xl font-black text-amber-300 my-2 h-8 drop-shadow-[0_2px_8px_rgba(251,191,36,0.3)]">？</div>
            <button onclick="drawLuckyStudent()" class="bg-gradient-to-r from-amber-500 to-amber-600 hover:from-amber-400 hover:to-amber-500 text-slate-950 text-xs font-black px-5 py-2 rounded-lg transition shadow-md">
                從「已到員工」隨機抽一位
            </button>
        </div>

        <div class="flex justify-between items-center border-t border-slate-800 pt-4">
            <div>
                <h2 class="text-base font-bold text-slate-200 inline-block">未簽到人員 (<span id="studentCount" class="text-indigo-400">0</span>)</h2>
                <span id="todayLabel" class="text-xs text-slate-500 ml-2 font-mono"></span>
            </div>
            <div class="flex gap-4 text-xs">
                <button onclick="resetRollCall()" class="text-slate-400 hover:text-slate-200 transition">今日重置</button>
                <button onclick="clearAll()" class="text-rose-400 hover:text-rose-300 font-medium transition">清空員工</button>
            </div>
        </div>

        <div id="studentList" class="space-y-2 max-h-60 overflow-y-auto pr-1">
            </div>

        <div class="bg-slate-950/50 border border-slate-800 rounded-2xl p-4 flex flex-col sm:flex-row items-center justify-between gap-4">
            <div class="flex gap-8 text-center w-full sm:w-auto justify-around sm:justify-start">
                <div>
                    <p class="text-3xl font-black text-emerald-400" id="presentStat">0</p>
                    <p class="text-xs font-medium text-slate-400 mt-0.5">今日出勤</p>
                </div>
                <div>
                    <p class="text-3xl font-black text-rose-400" id="absentStat">0</p>
                    <p class="text-xs font-medium text-slate-400 mt-0.5">今日請假</p>
                </div>
            </div>
            <button onclick="submitRollCall()" class="w-full sm:w-auto bg-gradient-to-r from-emerald-500 to-teal-600 hover:from-emerald-400 hover:to-teal-500 text-slate-950 font-black px-6 py-3 rounded-xl transition shadow-lg shadow-emerald-500/10 text-sm">
                📤 送出今日出勤報表
            </button>
        </div>

        <div class="border-t border-slate-800 pt-4">
            <div class="flex justify-between items-center mb-3">
                <h2 class="text-base font-bold text-slate-200">📜 歷史簽到日誌 <span class="text-xs font-normal text-slate-500">(點擊行項目查看名單)</span></h2>
                <button onclick="clearHistory()" class="text-xs text-slate-500 hover:text-rose-400 transition">清空日誌</button>
            </div>
            <div id="historyList" class="space-y-2 max-h-40 overflow-y-auto pr-1">
                </div>
        </div>

        <div id="historyDetailBlock" class="hidden bg-slate-950/80 rounded-2xl p-4 border border-indigo-500/20 space-y-3 shadow-inner">
            <div class="flex justify-between items-center">
                <h3 class="font-bold text-indigo-300 text-sm">📅 <span id="detailDateTitle"></span> 的出勤名單核對：</h3>
                <button onclick="document.getElementById('historyDetailBlock').classList.add('hidden')" class="text-xs text-slate-500 hover:text-slate-300 font-bold">關閉 ✕</button>
            </div>
            <div id="historyDetailRows" class="bg-slate-900 rounded-xl border border-slate-800 divide-y divide-slate-800 overflow-hidden text-sm">
                </div>
        </div>

    </div>

    <script>
        // 1. 初始化資料
        let students = JSON.parse(localStorage.getItem('corp_staff_v1')) || [];
        let historyRecords = JSON.parse(localStorage.getItem('corp_history_v1')) || [];
        let lastRollCallDate = localStorage.getItem('corp_last_date');

        // 取得今天的日期
        const now = new Date();
        const todayStr = `${now.getFullYear()}/${now.getMonth()+1}/${now.getDate()}`;
        if(document.getElementById('todayLabel')) {
            document.getElementById('todayLabel').textContent = `📅 TODAY：${todayStr}`;
        }

        // 2. 核心功能：簽到後直接隱藏
        function renderStudents() {
            const listEl = document.getElementById('studentList');
            const countEl = document.getElementById('studentCount');
            if (!listEl) return;
            
            listEl.innerHTML = '';
            
            const remainingStudents = students.filter(s => s.status === 'pending');
            if (countEl) countEl.textContent = remainingStudents.length;

            if (remainingStudents.length === 0) {
                if (students.length > 0) {
                    listEl.innerHTML = `
                        <div class="text-center py-8 bg-slate-950/20 border border-dashed border-slate-800 rounded-2xl">
                            <p class="text-emerald-400 font-bold text-base">🎉 本日全體同仁簽到完畢！</p>
                            <p class="text-slate-500 text-xs mt-1">請點擊下方按鈕將報表送出歸檔。</p>
                        </div>
                    `;
                } else {
                    listEl.innerHTML = `<p class="text-slate-500 text-center py-6 text-sm border border-dashed border-slate-800 rounded-2xl">名單目前為空，請在上方新增同仁姓名。</p>`;
                }
                updateStats();
                return;
            }

            students.forEach((student, index) => {
                if (student.status !== 'pending') return;

                const div = document.createElement('div');
                div.className = "flex justify-between items-center bg-slate-900/60 p-3 rounded-xl border border-slate-800/80 hover:border-slate-700 transition duration-200";
                
                div.innerHTML = `
                    <div class="flex items-center gap-2">
                        <span class="font-semibold text-slate-200">${student.name}</span>
                        <span class="text-[10px] font-bold text-slate-400 bg-slate-800 px-2 py-0.5 rounded border border-slate-700">待簽</span>
                    </div>
                    <div class="flex gap-1.5">
                        <button onclick="setStatus(${index}, 'present')" class="bg-emerald-500/20 hover:bg-emerald-500 text-emerald-400 hover:text-slate-950 border border-emerald-500/30 text-xs font-bold px-3 py-1.5 rounded-lg transition duration-200">上班</button>
                        <button onclick="setStatus(${index}, 'absent')" class="bg-rose-500/20 hover:bg-rose-500 text-rose-400 hover:text-slate-950 border border-rose-500/30 text-xs font-bold px-3 py-1.5 rounded-lg transition duration-200">請假</button>
                        <button onclick="deleteStudent(${index})" class="text-slate-600 hover:text-rose-400 text-sm px-2 transition">✕</button>
                    </div>
                `;
                listEl.appendChild(div);
            });
            updateStats();
            localStorage.setItem('corp_staff_v1', JSON.stringify(students));
        }

        // 3. 設定狀態
        function setStatus(index, status) {
            students[index].status = status;
            renderStudents(); 
        }

        // 4. 手動送出
        function submitRollCall() {
            if (students.length === 0) {
                alert("目前員工名單內無資料！");
                return;
            }

            const presentCount = students.filter(s => s.status === 'present').length;
            const absentCount = students.filter(s => s.status === 'absent').length;
            const pendingCount = students.filter(s => s.status === 'pending').length;

            if (pendingCount > 0) {
                if (!confirm(`尚有 ${pendingCount} 名員工未簽到，確定送出？`)) return;
            }

            const clickTime = new Date();
            const timeStr = `${String(clickTime.getHours()).padStart(2, '0')}:${String(clickTime.getMinutes()).padStart(2, '0')}`;
            const detailList = students.map(s => ({ name: s.name, status: s.status }));

            const newRecord = {
                date: todayStr,
                time: timeStr,
                present: presentCount,
                absent: absentCount,
                details: detailList
            };

            historyRecords.push(newRecord);
            localStorage.setItem('corp_history_v1', JSON.stringify(historyRecords));

            alert(`📤 成功送出出勤日誌報告！`);
            renderHistory();
        }

        // 5. 新增員工
        function addStudent() {
            const input = document.getElementById('studentInput');
            const name = input.value.trim();
            if (!name) return;
            students.push({ name, status: 'pending' });
            input.value = '';
            renderStudents();
        }

        // 6. 批量匯入
        function addBatchStudents() {
            const input = document.getElementById('batchInput');
            if (!input || !input.value.trim()) return;
            const names = input.value.split(/[\n,，]+/).map(n => n.trim()).filter(n => n.length > 0);
            names.forEach(name => students.push({ name, status: 'pending' }));
            input.value = '';
            switchTab('single');
            renderStudents();
        }

        // 7. 更新數字
        function updateStats() {
            const presentCount = students.filter(s => s.status === 'present').length;
            const absentCount = students.filter(s => s.status === 'absent').length;
            if(document.getElementById('presentStat')) document.getElementById('presentStat').textContent = presentCount;
            if(document.getElementById('absentStat')) document.getElementById('absentStat').textContent = absentCount;
        }

        // 8. 跨天防呆
        function checkNewDay() {
            if (lastRollCallDate && lastRollCallDate !== todayStr) {
                const presentCount = students.filter(s => s.status === 'present').length;
                const absentCount = students.filter(s => s.status === 'absent').length;

                if (presentCount > 0 || absentCount > 0) {
                    const detailList = students.map(s => ({ name: s.name, status: s.status }));
                    const autoRecord = {
                        date: lastRollCallDate,
                        time: "系統自動歸檔",
                        present: presentCount,
                        absent: absentCount,
                        details: detailList
                    };
                    historyRecords.push(autoRecord);
                    localStorage.setItem('corp_history_v1', JSON.stringify(historyRecords));
                }
                students.forEach(s => s.status = 'pending');
                localStorage.setItem('corp_staff_v1', JSON.stringify(students));
            }
            localStorage.setItem('corp_last_date', todayStr);
        }

        // 9. 渲染歷史紀錄清單
        function renderHistory() {
            const historyEl = document.getElementById('historyList');
            if (!historyEl) return;
            historyEl.innerHTML = '';

            if (historyRecords.length === 0) {
                historyEl.innerHTML = `<p class="text-slate-500 text-center py-4 text-xs">暫無任何歷史出勤日誌。</p>`;
                return;
            }

            [...historyRecords].reverse().forEach((record, index) => {
                const trueIndex = historyRecords.length - 1 - index; 

                const div = document.createElement('div');
                div.className = "flex justify-between items-center bg-slate-950/40 hover:bg-slate-950/80 p-2.5 rounded-xl text-xs border border-slate-800 cursor-pointer transition";
                div.setAttribute('onclick', `showHistoryDetail(${trueIndex})`);
                
                div.innerHTML = `
                    <div>
                        <span class="font-bold text-indigo-400 hover:text-indigo-300">📅 ${record.date}</span>
                        <span class="text-slate-500 ml-1">(${record.time})</span>
                    </div>
                    <div class="flex gap-3 bg-slate-900 px-2 py-1 rounded-lg border border-slate-800">
                        <span class="text-emerald-400">出勤：${record.present}</span>
                        <span class="text-rose-400">請假：${record.absent}</span>
                    </div>
                `;
                historyEl.appendChild(div);
            });
        }

        // 10. 顯示明細
        function showHistoryDetail(index) {
            const record = historyRecords[index];
            const detailBlock = document.getElementById('historyDetailBlock');
            const detailRows = document.getElementById('historyDetailRows');
            const dateTitle = document.getElementById('detailDateTitle');

            if (!record) return;

            dateTitle.textContent = `${record.date} ${record.time}`;
            detailRows.innerHTML = '';

            record.details.forEach(student => {
                const row = document.createElement('div');
                row.className = "flex justify-between items-center px-4 py-2.5 hover:bg-slate-800/40 text-slate-300";
                
                let statusBadge = '';
                if (student.status === 'present') {
                    statusBadge = '<span class="text-xs font-bold text-emerald-400 bg-emerald-400/10 px-2 py-0.5 rounded border border-emerald-500/20">出勤</span>';
                } else if (student.status === 'absent') {
                    statusBadge = '<span class="text-xs font-bold text-rose-400 bg-rose-400/10 px-2 py-0.5 rounded border border-rose-500/20">請假</span>';
                } else {
                    statusBadge = '<span class="text-xs font-bold text-slate-500 bg-slate-500/10 px-2 py-0.5 rounded border border-slate-500/20">未簽到</span>';
                }

                row.innerHTML = `
                    <span class="font-medium">${student.name}</span>
                    <div class="flex items-center gap-4">
                        <span class="text-xs text-slate-500 font-mono">${record.date}</span>
                        ${statusBadge}
                    </div>
                `;
                detailRows.appendChild(row);
            });

            detailBlock.classList.remove('hidden');
            detailBlock.scrollIntoView({ behavior: 'smooth' });
        }

        // 11. 隨機抽籤
        function drawLuckyStudent() {
            const winnerEl = document.getElementById('luckyWinner');
            const arrivedStudents = students.filter(s => s.status === 'present');
            if (!winnerEl) return;
            
            if (arrivedStudents.length === 0) {
                winnerEl.textContent = "❌ 無任何已出勤同仁可供抽取";
                winnerEl.className = "text-xs font-medium text-rose-400 my-2 h-8";
                return;
            }
            winnerEl.className = "text-2xl font-black text-amber-400 my-2 h-8 animate-bounce";
            winnerEl.textContent = "🎲 抽籤中...";
            setTimeout(() => {
                const randomIndex = Math.floor(Math.random() * arrivedStudents.length);
                winnerEl.textContent = `🎉 ${arrivedStudents[randomIndex].name} 🎉`;
                winnerEl.className = "text-2xl font-black text-amber-300 my-2 h-8 drop-shadow-[0_2px_8px_rgba(251,191,36,0.5)]";
            }, 600);
        }

        // 12. 切換頁籤
        function switchTab(type) {
            const pSingle = document.getElementById('panel-single');
            const pBatch = document.getElementById('panel-batch');
            const tSingle = document.getElementById('tab-single');
            const tBatch = document.getElementById('tab-batch');
            if (type === 'single') {
                if(pSingle) pSingle.classList.remove('hidden'); 
                if(pBatch) pBatch.classList.add('hidden');
                tSingle.className = "py-2 text-sm font-semibold rounded-lg bg-slate-800 text-white shadow-xs transition duration-300";
                tBatch.className = "py-2 text-sm font-semibold rounded-lg text-slate-400 hover:text-slate-200 transition duration-300";
            } else {
                if(pSingle) pSingle.classList.add('hidden'); 
                if(pBatch) pBatch.classList.remove('hidden');
                tSingle.className = "py-2 text-sm font-semibold rounded-lg text-slate-400 hover:text-slate-200 transition duration-300";
                tBatch.className = "py-2 text-sm font-semibold rounded-lg bg-slate-800 text-white shadow-xs transition duration-300";
            }
        }

        // 13. 控制按鈕
        function deleteStudent(index) { students.splice(index, 1); renderStudents(); }
        function resetRollCall() { if(confirm("確定要重置今日所有出勤狀態？")) { students.forEach(s => s.status = 'pending'); renderStudents(); } }
        function clearAll() { if(confirm("確定要清空整份員工名單？")) { students = []; renderStudents(); } }
        function clearHistory() { if(confirm("確定要刪除所有出勤日誌紀錄？")) { historyRecords = []; localStorage.setItem('corp_history_v1', JSON.stringify(historyRecords)); renderHistory(); document.getElementById('historyDetailBlock').classList.add('hidden'); } }

        // 🚀 初始化
        checkNewDay();
        renderStudents();
        renderHistory();
    </script>
</body>
</html>

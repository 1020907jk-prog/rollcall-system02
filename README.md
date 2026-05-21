<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>快速點名系統 2.0 - 手動送出名單版</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
</head>
<body class="bg-slate-50 min-h-screen p-4 md:p-8 font-sans">

    <div class="max-w-2xl mx-auto bg-white rounded-2xl shadow-xl border border-slate-100 p-6 space-y-6">
        <div class="text-center">
            <h1 class="text-3xl font-extrabold text-slate-800 tracking-wide">📋 快速點名系統 02</h1>
            <p class="text-sm text-emerald-600 font-medium mt-1 bg-emerald-50 inline-block px-3 py-1 rounded-full">✓ 點完名直接隱藏 ｜ 📤 點擊一鍵送出存檔</p>
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

        <div class="bg-slate-50 border border-slate-100 rounded-xl p-4 flex flex-col sm:flex-row items-center justify-between gap-4">
            <div class="flex gap-8 text-center w-full sm:w-auto justify-around sm:justify-start">
                <div>
                    <p class="text-2xl font-black text-emerald-600" id="presentStat">0</p>
                    <p class="text-xs font-medium text-slate-400 mt-0.5">今日已到</p>
                </div>
                <div>
                    <p class="text-2xl font-black text-rose-500" id="absentStat">0</p>
                    <p class="text-xs font-medium text-slate-400 mt-0.5">今日缺席</p>
                </div>
            </div>
            <button onclick="submitRollCall()" class="w-full sm:w-auto bg-emerald-600 hover:bg-emerald-700 text-white font-bold px-6 py-2.5 rounded-xl transition shadow-md text-sm">
                📤 確認送出今日點名
            </button>
        </div>

        <hr class="border-slate-100">

        <div>
            <div class="flex justify-between items-center mb-3">
                <h2 class="text-lg font-bold text-slate-700">📜 歷史點名紀錄 <span class="text-xs font-normal text-slate-400">(點擊日期可查看詳細名單)</span></h2>
                <button onclick="clearHistory()" class="text-xs text-slate-400 hover:text-red-500 transition">清空歷史</button>
            </div>
            <div id="historyList" class="space-y-2 max-h-48 overflow-y-auto pr-1">
                </div>
        </div>

        <div id="historyDetailBlock" class="hidden bg-slate-100 rounded-2xl p-4 border border-slate-200 space-y-3">
            <div class="flex justify-between items-center">
                <h3 class="font-bold text-slate-800 text-sm">📅 <span id="detailDateTitle"></span> 的詳細點名名單：</h3>
                <button onclick="document.getElementById('historyDetailBlock').classList.add('hidden')" class="text-xs text-slate-400 hover:text-slate-600 font-bold">關閉明細 ✕</button>
            </div>
            <div id="historyDetailRows" class="bg-white rounded-xl border border-slate-200 divide-y divide-slate-100 overflow-hidden text-sm">
                </div>
        </div>

    </div>

    <script>
        // 1. 初始化資料
        let students = JSON.parse(localStorage.getItem('students_v2')) || [];
        let historyRecords = JSON.parse(localStorage.getItem('rollcall_history_v3')) || [];
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
            
            const remainingStudents = students.filter(s => s.status === 'pending');
            if (countEl) countEl.textContent = remainingStudents.length;

            if (remainingStudents.length === 0) {
                if (students.length > 0) {
                    listEl.innerHTML = `
                        <div class="text-center py-8">
                            <p class="text-emerald-600 font-bold text-lg">🎉 全班點名完成！</p>
                            <p class="text-slate-400 text-xs mt-1">請點擊下方「確認送出今日點名」按鈕存檔。</p>
                        </div>
                    `;
                } else {
                    listEl.innerHTML = `<p class="text-slate-400 text-center py-6 text-sm">名單是空的，快去上方新增學生吧！</p>`;
                }
                updateStats();
                return;
            }

            students.forEach((student, index) => {
                if (student.status !== 'pending') return;

                const div = document.createElement('div');
                div.className = "flex justify-between items-center bg-white p-3 rounded-xl border border-slate-100 shadow-xs hover:border-slate-200 transition";
                
                div.innerHTML = `
                    <div class="flex items-center gap-2">
                        <span class="font-semibold text-slate-700">${student.name}</span>
                        <span class="text-xs font-bold text-slate-400 bg-slate-100 px-2 py-1 rounded-md">未點</span>
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
            renderStudents(); 
        }

        // 4. 【全新功能】手動按一格按鈕送出點名紀錄
        function submitRollCall() {
            if (students.length === 0) {
                alert("目前名單內沒有學生，無法送出紀錄！");
                return;
            }

            const presentCount = students.filter(s => s.status === 'present').length;
            const absentCount = students.filter(s => s.status === 'absent').length;
            const pendingCount = students.filter(s => s.status === 'pending').length;

            // 如果有人還沒點名，跳出警告
            if (pendingCount > 0) {
                if (!confirm(`還有 ${pendingCount} 位學生尚未點名，確定要直接送出紀錄嗎？（未點名的學生將不會計入到/缺中）`)) {
                    return;
                }
            }

            // 撈取點擊當下的精準時間
            const clickTime = new Date();
            const timeStr = `${String(clickTime.getHours()).padStart(2, '0')}:${String(clickTime.getMinutes()).padStart(2, '0')}`;
            
            // 複製一份目前所有學生的姓名與狀態
            const detailList = students.map(s => ({ name: s.name, status: s.status }));

            // 建立歷史物件
            const newRecord = {
                date: todayStr,
                time: timeStr,
                present: presentCount,
                absent: absentCount,
                details: detailList
            };

            // 塞進歷史紀錄並儲存
            historyRecords.push(newRecord);
            localStorage.setItem('rollcall_history_v3', JSON.stringify(historyRecords));

            alert(`🎉 點名紀錄送出成功！已同步儲存至下方歷史紀錄中。`);

            // 重新渲染歷史清單
            renderHistory();
        }

        // 5. 單筆新增
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

        // 7. 更新統計數字
        function updateStats() {
            const presentCount = students.filter(s => s.status === 'present').length;
            const absentCount = students.filter(s => s.status === 'absent').length;
            if(document.getElementById('presentStat')) document.getElementById('presentStat').textContent = presentCount;
            if(document.getElementById('absentStat')) document.getElementById('absentStat').textContent = absentCount;
        }

        // 8. 跨天檢查（防呆保留：若忘記按送出，隔天還是會幫忙墊底自動存一份）
        function checkNewDay() {
            if (lastRollCallDate && lastRollCallDate !== todayStr) {
                const presentCount = students.filter(s => s.status === 'present').length;
                const absentCount = students.filter(s => s.status === 'absent').length;

                if (presentCount > 0 || absentCount > 0) {
                    const detailList = students.map(s => ({ name: s.name, status: s.status }));
                    const autoRecord = {
                        date: lastRollCallDate,
                        time: "隔日自動歸檔",
                        present: presentCount,
                        absent: absentCount,
                        details: detailList
                    };
                    historyRecords.push(autoRecord);
                    localStorage.setItem('rollcall_history_v3', JSON.stringify(historyRecords));
                }
                students.forEach(s => s.status = 'pending');
                localStorage.setItem('students_v2', JSON.stringify(students));
            }
            localStorage.setItem('last_rollcall_date', todayStr);
        }

        // 9. 渲染歷史紀錄清單
        function renderHistory() {
            const historyEl = document.getElementById('historyList');
            if (!historyEl) return;
            historyEl.innerHTML = '';

            if (historyRecords.length === 0) {
                historyEl.innerHTML = `<p class="text-slate-400 text-center py-4 text-xs">目前還沒有任何歷史點名紀錄。</p>`;
                return;
            }

            [...historyRecords].reverse().forEach((record, index) => {
                const trueIndex = historyRecords.length - 1 - index; 

                const div = document.createElement('div');
                div.className = "flex justify-between items-center bg-slate-50 hover:bg-slate-100 p-2.5 rounded-lg text-xs border border-slate-200 cursor-pointer transition";
                div.setAttribute('onclick', `showHistoryDetail(${trueIndex})`);
                
                div.innerHTML = `
                    <div>
                        <span class="font-bold text-blue-600 hover:underline">📅 ${record.date}</span>
                        <span class="text-slate-400 ml-1">(${record.time})</span>
                    </div>
                    <div class="flex gap-2 bg-white px-2 py-1 rounded border border-slate-100">
                        <span class="text-emerald-600 font-medium">到：${record.present}</span>
                        <span class="text-rose-500 font-medium">缺：${record.absent}</span>
                    </div>
                `;
                historyEl.appendChild(div);
            });
        }

        // 10. 顯示某天的「一行一行姓名與狀態」
        function showHistoryDetail(index) {
            const record = historyRecords[index];
            const detailBlock = document.getElementById('historyDetailBlock');
            const detailRows = document.getElementById('historyDetailRows');
            const dateTitle = document.getElementById('detailDateTitle');

            if (!record) return;

            dateTitle.textContent = `${record.date} (${record.time})`;
            detailRows.innerHTML = '';

            if (!record.details || record.details.length === 0) {
                detailRows.innerHTML = `<p class="p-4 text-xs text-slate-400 text-center">此筆紀錄無細節名字資料。</p>`;
                detailBlock.classList.remove('hidden');
                return;
            }

            record.details.forEach(student => {
                const row = document.createElement('div');
                row.className = "flex justify-between items-center px-4 py-2.5 hover:bg-slate-50 transition";
                
                let statusBadge = '';
                if (student.status === 'present') {
                    statusBadge = '<span class="text-xs font-bold text-emerald-600 bg-emerald-50 px-2 py-0.5 rounded">已到</span>';
                } else if (student.status === 'absent') {
                    statusBadge = '<span class="text-xs font-bold text-rose-600 bg-rose-50 px-2 py-0.5 rounded">缺席</span>';
                } else {
                    statusBadge = '<span class="text-xs font-bold text-slate-400 bg-slate-50 px-2 py-0.5 rounded">未點名</span>';
                }

                row.innerHTML = `
                    <span class="font-medium text-slate-700">${student.name}</span>
                    <div class="flex items-center gap-4">
                        <span class="text-xs text-slate-400">${record.date}</span>
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

        // 12. 切換頁籤
        function switchTab(type) {
            const pSingle = document.getElementById('panel-single');
            const pBatch = document.getElementById('panel-batch');
            if (type === 'single') {
                if(pSingle) pSingle.classList.remove('hidden'); 
                if(pBatch) pBatch.classList.add('hidden');
            } else {
                if(pSingle) pSingle.classList.add('hidden'); 
                if(pBatch) pBatch.github.classList.remove('hidden'); // 防呆
                if(pBatch) pBatch.classList.remove('hidden');
            }
        }

        // 13. 控制按鈕
        function deleteStudent(index) { students.splice(index, 1); renderStudents(); }
        function resetRollCall() { if(confirm("確定要重置今天的點名狀態嗎？")) { students.forEach(s => s.status = 'pending'); renderStudents(); } }
        function clearAll() { if(confirm("確定要清空整份學生名單嗎？")) { students = []; renderStudents(); } }
        function clearHistory() { if(confirm("確定要刪除所有歷史點名紀錄嗎？")) { historyRecords = []; localStorage.setItem('rollcall_history_v3', JSON.stringify(historyRecords)); renderHistory(); document.getElementById('historyDetailBlock').classList.add('hidden'); } }

        // 🚀 初始化
        checkNewDay();
        renderStudents();
        renderHistory();
    </script>
</body>
</html>

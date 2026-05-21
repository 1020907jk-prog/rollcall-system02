// 1. 初始化資料（從瀏覽器讀取舊資料，如果沒有就給空陣列）
        let students = JSON.parse(localStorage.getItem('students_v2')) || [];
        let historyRecords = JSON.parse(localStorage.getItem('rollcall_history')) || [];
        let lastRollCallDate = localStorage.getItem('last_rollcall_date');

        // 取得今天的日期並顯示
        const now = new Date();
        const todayStr = `${now.getFullYear()}/${now.getMonth()+1}/${now.getDate()}`;
        if(document.getElementById('todayLabel')) {
            document.getElementById('todayLabel').textContent = `📅 今日：${todayStr}`;
        }

        // 2. 核心：你剛剛提供的點名渲染功能（已修正與優化）
        function renderStudents() {
            const listEl = document.getElementById('studentList');
            const countEl = document.getElementById('studentCount');
            if (!listEl) return; // 預防找不到 HTML 元素
            
            listEl.innerHTML = '';
            
            // 過濾出「還沒點名」的學生
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

            // 遍歷「還沒點名」的學生並呈現在網頁上
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

        // 3. 處理按鈕點擊：設定到/缺狀態
        function setStatus(index, status) {
            students[index].status = status;
            renderStudents(); // 重新渲染，被點到的學生就會消失
        }

        // 4. 單筆新增學生
        function addStudent() {
            const input = document.getElementById('studentInput');
            const name = input.value.trim();
            if (!name) return;
            students.push({ name, status: 'pending' });
            input.value = '';
            renderStudents();
        }

        // 5. 批量匯入學生
        function addBatchStudents() {
            const input = document.getElementById('batchInput');
            if (!input || !input.value.trim()) return;
            const names = input.value.split(/[\n,，]+/).map(n => n.trim()).filter(n => n.length > 0);
            names.forEach(name => students.push({ name, status: 'pending' }));
            input.value = '';
            switchTab('single');
            renderStudents();
        }

        // 6. 更新底部的統計數據數字
        function updateStats() {
            const presentCount = students.filter(s => s.status === 'present').length;
            const absentCount = students.filter(s => s.status === 'absent').length;
            
            if(document.getElementById('presentStat')) document.getElementById('presentStat').textContent = presentCount;
            if(document.getElementById('absentStat')) document.getElementById('absentStat').textContent = absentCount;
        }

        // 7. 跨天自動歸檔檢查
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

        // 8. 歷史紀錄渲染
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

        // 9. 隨機抽籤功能 (只抽已到的人)
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

        // 10. 切換單筆/批量頁籤
        function switchTab(type) {
            const pSingle = document.getElementById('panel-single');
            const pBatch = document.getElementById('panel-batch');
            const tSingle = document.getElementById('tab-single');
            const tBatch = document.getElementById('tab-batch');

            if (type === 'single') {
                if(pSingle) pSingle.classList.remove('hidden'); 
                if(pBatch) pBatch.classList.add('hidden');
                if(tSingle) tSingle.className = "py-2 text-sm font-medium rounded-lg bg-white shadow-xs text-slate-800 transition";
                if(tBatch) tBatch.className = "py-2 text-sm font-medium rounded-lg text-slate-500 hover:text-slate-800 transition";
            } else {
                if(pSingle) pSingle.classList.add('hidden'); 
                if(pBatch) pBatch.classList.remove('hidden');
                if(tSingle) tSingle.className = "py-2 text-sm font-medium rounded-lg text-slate-500 hover:text-slate-800 transition";
                if(tBatch) tBatch.className = "py-2 text-sm font-medium rounded-lg bg-white shadow-xs text-slate-800 transition";
            }
        }

        // 11. 其他控制按鈕
        function deleteStudent(index) { students.splice(index, 1); renderStudents(); }
        function resetRollCall() { if(confirm("確定要重置今天的點名狀態嗎？")) { students.forEach(s => s.status = 'pending'); renderStudents(); } }
        function clearAll() { if(confirm("確定要清空整份學生名單嗎？")) { students = []; renderStudents(); } }
        function clearHistory() { if(confirm("確定要刪除所有歷史點名紀錄嗎？")) { historyRecords = []; localStorage.setItem('rollcall_history', JSON.stringify(historyRecords)); renderHistory(); } }

        // 🚀 初始化執行
        checkNewDay();
        renderStudents();
        renderHistory();

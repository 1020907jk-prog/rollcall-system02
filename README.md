function renderStudents() {
            const listEl = document.getElementById('studentList');
            const countEl = document.getElementById('studentCount');
            listEl.innerHTML = '';
            
            // 💡 核心修改：過濾出「還沒點名」的學生
            const remainingStudents = students.filter(s => s.status === 'pending');
            
            // 這裡顯示的數量改成「還沒點名的人數」
            countEl.textContent = remainingStudents.length;

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

            // 遍歷「還沒點名」的學生
            students.forEach((student, index) => {
                // 💡 關鍵：如果這個學生已經點過了 (到或缺)，就跳過不渲染它
                if (student.status !== 'pending') return;

                const div = document.createElement('div');
                div.className = "flex justify-between items-center bg-white p-3 rounded-xl border border-slate-100 shadow-xs hover:border-slate-200 transition";
                
                // 因為留在畫面上的都是未點，所以 badge 固定是未點
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

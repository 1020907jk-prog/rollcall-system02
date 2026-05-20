<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的線上點名系統</title>
    <!-- 引入 Tailwind CSS 讓畫面變好看 -->
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
</head>
<body class="bg-gray-50 min-h-screen p-6 font-sans">

    <div class="max-w-2xl mx-auto bg-white rounded-xl shadow-md p-6">
        <h1 class="text-3xl font-bold text-gray-800 text-center mb-6">📋 快速點名系統</h1>

        <!-- 新增學生區塊 -->
        <div class="flex gap-2 mb-6">
            <input type="text" id="studentInput" placeholder="輸入學生姓名..." 
                   class="flex-1 border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500">
            <button onclick="addStudent()" 
                    class="bg-blue-600 hover:bg-blue-700 text-white font-medium px-4 py-2 rounded-lg transition">
                新增學生
            </button>
        </div>

        <hr class="border-gray-200 mb-6">

        <!-- 學生名單區塊 -->
        <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-semibold text-gray-700">學生名單 (<span id="studentCount">0</span>)</h2>
            <button onclick="resetRollCall()" class="text-sm text-red-500 hover:underline">重設所有狀態</button>
        </div>

        <div id="studentList" class="space-y-3">
            <!-- 學生項目會由 JavaScript 動態產生 -->
            <p class="text-gray-400 text-center py-4" id="emptyMessage">目前名單空空如也，請先新增學生！</p>
        </div>

        <!-- 底部統計 -->
        <div class="mt-8 pt-4 border-t border-gray-100 flex justify-around text-center">
            <div>
                <p class="text-2xl font-bold text-green-600" id="presentStat">0</p>
                <p class="text-xs text-gray-500">已到</p>
            </div>
            <div>
                <p class="text-2xl font-bold text-red-500" id="absentStat">0</p>
                <p class="text-xs text-gray-500">缺席</p>
            </div>
        </div>
    </div>

    <script>
        // 初始化學生資料，優先從瀏覽器快取讀取
        let students = JSON.parse(localStorage.getItem('students')) || [];

        // 渲染畫面
        function render() {
            const listEl = document.getElementById('studentList');
            const emptyMsg = document.getElementById('emptyMessage');
            const countEl = document.getElementById('studentCount');
            
            listEl.innerHTML = '';
            countEl.textContent = students.length;

            if (students.length === 0) {
                listEl.appendChild(emptyMsg);
                updateStats();
                return;
            }

            students.forEach((student, index) => {
                const div = document.createElement('div');
                div.className = "flex justify-between items-center bg-gray-50 p-3 rounded-lg border border-gray-100 shadow-xs";
                
                // 依據狀態給予不同顏色的標籤
                let statusBadge = '';
                if (student.status === 'present') {
                    statusBadge = '<span class="text-sm font-semibold text-green-600 bg-green-50 px-2 py-1 rounded">已到</span>';
                } else if (student.status === 'absent') {
                    statusBadge = '<span class="text-sm font-semibold text-red-600 bg-red-50 px-2 py-1 rounded">缺席</span>';
                } else {
                    statusBadge = '<span class="text-sm font-semibold text-gray-400 bg-gray-100 px-2 py-1 rounded">未點</span>';
                }

                div.innerHTML = `
                    <div class="flex items-center gap-3">
                        <span class="font-medium text-gray-800">${student.name}</span>
                        ${statusBadge}
                    </div>
                    <div class="flex gap-1">
                        <button onclick="setStatus(${index}, 'present')" class="bg-green-500 hover:bg-green-600 text-white text-xs font-medium px-3 py-1 rounded">到</button>
                        <button onclick="setStatus(${index}, 'absent')" class="bg-red-500 hover:bg-red-600 text-white text-xs font-medium px-3 py-1 rounded">缺</button>
                        <button onclick="deleteStudent(${index})" class="text-gray-400 hover:text-red-500 text-xs px-2 py-1">✕</button>
                    </div>
                `;
                listEl.appendChild(div);
            });

            updateStats();
            // 儲存到瀏覽器，重新整理也不會不見
            localStorage.setItem('students', JSON.stringify(students));
        }

        // 新增學生
        function addStudent() {
            const input = document.getElementById('studentInput');
            const name = input.value.trim();
            if (!name) return;

            students.push({ name: name, status: 'pending' });
            input.value = '';
            render();
        }

        // 設定出席狀態
        function setStatus(index, status) {
            students[index].status = status;
            render();
        }

        // 刪除學生
        function deleteStudent(index) {
            students.splice(index, 1);
            render();
        }

        // 重設點名
        function resetRollCall() {
            students.forEach(s => s.status = 'pending');
            render();
        }

        // 更新數據統計
        function updateStats() {
            const present = students.filter(s => s.status === 'present').length;
            const absent = students.filter(s => s.status === 'absent').length;
            
            document.getElementById('presentStat').textContent = present;
            document.getElementById('absentStat').textContent = absent;
        }

        // 初始化第一次渲染
        render();
    </script>
</body>
</html>

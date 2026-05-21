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

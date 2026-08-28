<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover" />
  <title>FlashSRS - 分散学習フラッシュカード</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Font Awesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css" />
  <!-- Google Fonts: Inter & Noto Sans JP -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=Noto+Sans+JP:wght@400;500;600;700;800&display=swap" rel="stylesheet">
  
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          fontFamily: {
            sans: ['Inter', 'Noto Sans JP', 'sans-serif'],
          },
          colors: {
            brand: {
              50: '#f0fdfa',
              100: '#ccfbf1',
              400: '#2dd4bf',
              500: '#14b8a6',
              600: '#0d9488',
              700: '#0f766e',
            }
          }
        }
      }
    }
  </script>

  <style>
    /* Mobile Touch Optimization & Safe Area */
    * {
      -webkit-tap-highlight-color: transparent;
      touch-action: manipulation;
    }

    body {
      padding-top: env(safe-area-inset-top, 0px);
      padding-bottom: calc(env(safe-area-inset-bottom, 0px) + 68px);
    }

    .safe-bottom {
      padding-bottom: env(safe-area-inset-bottom, 8px);
    }

    /* 3D Flashcard Flip */
    .perspective-1000 {
      perspective: 1000px;
    }
    .card-inner {
      transition: transform 0.35s cubic-bezier(0.2, 0.8, 0.2, 1);
      transform-style: preserve-3d;
    }
    .card-flipped {
      transform: rotateY(180deg);
    }
    .card-front, .card-back {
      backface-visibility: hidden;
      -webkit-backface-visibility: hidden;
    }
    .card-back {
      transform: rotateY(180deg);
    }

    /* Custom Scrollbar */
    ::-webkit-scrollbar {
      width: 4px;
      height: 4px;
    }
    ::-webkit-scrollbar-thumb {
      background: #cbd5e1;
      border-radius: 9999px;
    }
    .dark ::-webkit-scrollbar-thumb {
      background: #475569;
    }

    /* Bottomsheet / Modal Transitions */
    .bottom-sheet {
      transition: transform 0.22s cubic-bezier(0.16, 1, 0.3, 1), opacity 0.2s ease;
    }
    .bottom-sheet-hidden {
      transform: translateY(100%);
      opacity: 0;
      pointer-events: none;
    }
    .bottom-sheet-visible {
      transform: translateY(0);
      opacity: 1;
      pointer-events: auto;
    }
  </style>
</head>
<body class="bg-slate-100 dark:bg-slate-950 text-slate-800 dark:text-slate-100 min-h-screen flex flex-col font-sans select-none antialiased">

  <header class="bg-white/95 dark:bg-slate-900/95 backdrop-blur-md border-b border-slate-200/80 dark:border-slate-800 sticky top-0 z-30 px-4 py-2.5">
    <div class="max-w-2xl mx-auto flex items-center justify-between">
      <!-- Brand Logo -->
      <div class="flex items-center space-x-2.5 cursor-pointer" onclick="switchView('decks')">
        <div class="w-8 h-8 rounded-xl bg-gradient-to-tr from-teal-600 to-emerald-400 flex items-center justify-center text-white shadow-sm">
          <i class="fa-solid fa-brain text-sm"></i>
        </div>
        <div>
          <span class="text-base font-extrabold tracking-tight bg-gradient-to-r from-teal-600 to-emerald-500 bg-clip-text text-transparent">FlashSRS</span>
        </div>
      </div>

      <!-- Quick Action Buttons -->
      <div class="flex items-center space-x-2">
        <button onclick="toggleDarkMode()" class="w-8 h-8 rounded-xl flex items-center justify-center bg-slate-100 dark:bg-slate-800 text-slate-600 dark:text-slate-300 active:scale-95 transition">
          <i id="theme-icon" class="fa-solid fa-moon text-xs"></i>
        </button>
        <button onclick="openCardModal()" class="flex items-center space-x-1.5 bg-teal-600 active:bg-teal-700 text-white text-xs font-bold px-3 py-1.5 rounded-xl shadow-sm active:scale-95 transition">
          <i class="fa-solid fa-plus text-[10px]"></i>
          <span>単語追加</span>
        </button>
      </div>
    </div>
  </header>

  <main class="flex-1 max-w-2xl w-full mx-auto p-3 sm:p-4">

    <!-- 1. DECKS VIEW (Initial Screen) -->
    <section id="view-decks" class="view-section space-y-3">
      <div class="bg-white dark:bg-slate-900 rounded-2xl border border-slate-200/80 dark:border-slate-800 p-4 shadow-sm">
        <div class="flex items-center justify-between mb-3">
          <div>
            <h2 class="text-base font-bold text-slate-800 dark:text-white flex items-center space-x-1.5">
              <i class="fa-solid fa-folder-tree text-teal-500 text-sm"></i>
              <span>デッキ一覧</span>
            </h2>
            <p class="text-[11px] text-slate-400 mt-0.5">デッキをタップして単語学習を開始</p>
          </div>
          <div class="flex items-center space-x-1.5">
            <button onclick="openImportModal('__from_csv__')" class="bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 text-slate-700 dark:text-slate-300 text-xs font-bold px-2.5 py-1.5 rounded-xl transition flex items-center space-x-1 active:scale-95">
              <i class="fa-solid fa-file-import text-teal-500 text-xs"></i>
              <span>CSV読込</span>
            </button>
            <button onclick="openDeckModal()" class="bg-teal-600 active:bg-teal-700 text-white text-xs font-bold px-3 py-1.5 rounded-xl shadow-sm flex items-center space-x-1 active:scale-95">
              <i class="fa-solid fa-folder-plus text-xs"></i>
              <span>新規デッキ</span>
            </button>
          </div>
        </div>

        <!-- Hierarchical Tree -->
        <div id="deck-tree-container" class="divide-y divide-slate-100 dark:divide-slate-800/80">
          <!-- Dynamically populated -->
        </div>
      </div>
    </section>

    <!-- 2. STUDY VIEW -->
    <section id="view-study" class="view-section hidden">
      <!-- Deck Filter & Daily Limit Banner -->
      <div class="bg-white dark:bg-slate-900 p-3 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm mb-3">
        <div class="flex items-center justify-between gap-2 mb-2">
          <div class="flex items-center space-x-1.5 flex-1 min-w-0">
            <i class="fa-solid fa-folder-open text-teal-500 text-xs flex-shrink-0"></i>
            <select id="study-deck-filter" onchange="changeStudyDeck(this.value)" class="w-full text-xs font-bold bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-100 rounded-lg px-2.5 py-1.5 border-none truncate focus:ring-1 focus:ring-teal-500">
              <option value="all">すべてのデッキ（全体学習）</option>
            </select>
          </div>
          <button onclick="resetSession()" title="再読み込み" class="p-1.5 text-slate-400 hover:text-teal-600 text-xs rounded-lg active:scale-90">
            <i class="fa-solid fa-arrows-rotate"></i>
          </button>
        </div>

        <!-- SRS Status Counts -->
        <div class="grid grid-cols-3 gap-1.5 text-center text-[11px] font-bold">
          <div class="bg-blue-50 dark:bg-blue-950/40 text-blue-600 dark:text-blue-400 py-1 px-2 rounded-lg flex flex-col">
            <span>新規カード</span>
            <span class="text-xs mt-0.5"><span id="queue-new-count">0</span> / <span id="limit-new-text" class="text-[10px] font-normal opacity-80">20</span></span>
          </div>
          <div class="bg-amber-50 dark:bg-amber-950/40 text-amber-600 dark:text-amber-400 py-1 px-2 rounded-lg flex flex-col">
            <span>定着中</span>
            <span id="queue-learn-count" class="text-xs mt-0.5">0</span>
          </div>
          <div class="bg-emerald-50 dark:bg-emerald-950/40 text-emerald-600 dark:text-emerald-400 py-1 px-2 rounded-lg flex flex-col">
            <span>復習待ち</span>
            <span class="text-xs mt-0.5"><span id="queue-due-count">0</span> / <span id="limit-due-text" class="text-[10px] font-normal opacity-80">50</span></span>
          </div>
        </div>
      </div>

      <!-- Card Stage Container -->
      <div id="study-stage" class="flex flex-col">
        <!-- Progress Bar -->
        <div class="mb-2 flex items-center justify-between text-[11px] text-slate-400 font-medium px-1">
          <span id="session-progress-text">残り 0 枚</span>
          <span class="text-[10px] text-teal-600 dark:text-teal-400 font-semibold flex items-center space-x-1">
            <i class="fa-solid fa-hand-pointer text-[10px]"></i>
            <span>タップで反転</span>
          </span>
        </div>
        <div class="w-full bg-slate-200 dark:bg-slate-800 h-1.5 rounded-full overflow-hidden mb-3">
          <div id="session-progress-bar" class="bg-gradient-to-r from-teal-500 to-emerald-400 h-full w-0 transition-all duration-300"></div>
        </div>

        <!-- 3D Flashcard optimized for mobile height -->
        <div class="w-full h-[52vh] min-h-[300px] max-h-[450px] perspective-1000 cursor-pointer" onclick="flipCard()">
          <div id="flashcard-inner" class="card-inner relative w-full h-full rounded-3xl shadow-md border border-slate-200/90 dark:border-slate-800 bg-white dark:bg-slate-900 transition-all duration-300">
            
            <!-- FRONT SIDE -->
            <div class="card-front absolute inset-0 w-full h-full p-5 flex flex-col justify-between rounded-3xl bg-white dark:bg-slate-900">
              <div class="flex items-center justify-between">
                <span id="card-front-tag" class="px-2.5 py-0.5 rounded-full text-[11px] font-semibold bg-teal-50 dark:bg-teal-950/60 text-teal-600 dark:text-teal-400 border border-teal-100 dark:border-teal-900 truncate max-w-[200px]">
                  一般
                </span>
                <button onclick="event.stopPropagation(); speakCurrentWord();" class="w-9 h-9 rounded-full bg-slate-100 dark:bg-slate-800 active:bg-teal-100 dark:active:bg-teal-900 text-slate-600 dark:text-slate-300 text-sm flex items-center justify-center">
                  <i class="fa-solid fa-volume-high"></i>
                </button>
              </div>

              <!-- Main Front Text -->
              <div class="text-center my-auto py-2">
                <h2 id="card-front-text" class="text-3xl sm:text-4xl font-extrabold text-slate-900 dark:text-white tracking-tight break-words">
                  Word
                </h2>
                <p id="card-front-phonetic" class="text-xs font-mono text-slate-400 mt-1">/wɜːd/</p>
                <p id="card-front-hint" class="text-[11px] text-slate-400 dark:text-slate-500 mt-2 italic hidden"></p>
              </div>

              <div class="text-center text-[11px] text-slate-400 flex items-center justify-center space-x-1 py-1">
                <i class="fa-solid fa-arrow-rotate-left text-[10px]"></i>
                <span>タップして答えを確認</span>
              </div>
            </div>

            <!-- BACK SIDE -->
            <div class="card-back absolute inset-0 w-full h-full p-5 flex flex-col justify-between rounded-3xl bg-gradient-to-b from-slate-50 to-white dark:from-slate-900 dark:to-slate-850">
              <div class="flex items-center justify-between">
                <span id="card-back-tag" class="px-2.5 py-0.5 rounded-full text-[11px] font-semibold bg-indigo-50 dark:bg-indigo-950/60 text-indigo-600 dark:text-indigo-400 border border-indigo-100 dark:border-indigo-900 truncate max-w-[200px]">
                  日本語訳
                </span>
                <div class="flex items-center space-x-2">
                  <span id="card-interval-badge" class="text-[10px] text-slate-400 font-mono">1日後</span>
                  <button onclick="event.stopPropagation(); speakCurrentWord();" class="w-8 h-8 rounded-full bg-slate-100 dark:bg-slate-800 active:bg-indigo-100 text-slate-600 dark:text-slate-300 text-xs flex items-center justify-center">
                    <i class="fa-solid fa-volume-high"></i>
                  </button>
                </div>
              </div>

              <!-- Main Back Content -->
              <div class="text-center my-auto overflow-y-auto max-h-[36vh] px-1 py-1">
                <h3 id="card-back-text" class="text-2xl sm:text-3xl font-bold text-teal-600 dark:text-teal-400 mb-2 break-words">
                  意味
                </h3>
                <div id="card-example-box" class="bg-slate-100 dark:bg-slate-800/80 p-3 rounded-xl text-left text-xs text-slate-700 dark:text-slate-200 border-l-4 border-teal-500 my-2">
                  <p id="card-back-example" class="font-medium text-xs"></p>
                  <p id="card-back-example-tr" class="text-[11px] text-slate-400 mt-1"></p>
                </div>
                <p id="card-back-note" class="text-[11px] text-slate-500 dark:text-slate-400 mt-1.5 text-left"></p>
              </div>

              <div class="text-center text-[10px] text-slate-400">
                下のボタンで記憶度を選択
              </div>
            </div>

          </div>
        </div>

        <!-- SRS 4 Rating Buttons -->
        <div id="srs-action-bar" class="w-full mt-3 opacity-30 pointer-events-none transition-all duration-200">
          <div class="grid grid-cols-4 gap-1.5 sm:gap-2">
            <!-- 1: Again -->
            <button onclick="rateCard(1)" class="flex flex-col items-center justify-center py-2.5 px-1 rounded-2xl bg-red-50 active:bg-red-200 dark:bg-red-950/50 dark:active:bg-red-900 border border-red-200 dark:border-red-900/60 text-red-600 dark:text-red-400 active:scale-95 transition">
              <span class="text-xs font-bold leading-tight">もう一度</span>
              <span id="time-again" class="text-[9px] font-mono opacity-80 mt-0.5">&lt;10分</span>
            </button>

            <!-- 2: Hard -->
            <button onclick="rateCard(2)" class="flex flex-col items-center justify-center py-2.5 px-1 rounded-2xl bg-amber-50 active:bg-amber-200 dark:bg-amber-950/50 dark:active:bg-amber-900 border border-amber-200 dark:border-amber-900/60 text-amber-600 dark:text-amber-400 active:scale-95 transition">
              <span class="text-xs font-bold leading-tight">難しい</span>
              <span id="time-hard" class="text-[9px] font-mono opacity-80 mt-0.5">1日</span>
            </button>

            <!-- 3: Good -->
            <button onclick="rateCard(3)" class="flex flex-col items-center justify-center py-2.5 px-1 rounded-2xl bg-blue-50 active:bg-blue-200 dark:bg-blue-950/50 dark:active:bg-blue-900 border border-blue-200 dark:border-blue-900/60 text-blue-600 dark:text-blue-400 active:scale-95 transition">
              <span class="text-xs font-bold leading-tight">普通</span>
              <span id="time-good" class="text-[9px] font-mono opacity-80 mt-0.5">3日</span>
            </button>

            <!-- 4: Easy -->
            <button onclick="rateCard(4)" class="flex flex-col items-center justify-center py-2.5 px-1 rounded-2xl bg-emerald-50 active:bg-emerald-200 dark:bg-emerald-950/50 dark:active:bg-emerald-900 border border-emerald-200 dark:border-emerald-900/60 text-emerald-600 dark:text-emerald-400 active:scale-95 transition">
              <span class="text-xs font-bold leading-tight">簡単</span>
              <span id="time-easy" class="text-[9px] font-mono opacity-80 mt-0.5">7日</span>
            </button>
          </div>
        </div>

      </div>

      <!-- Completed Empty Screen -->
      <div id="study-empty-stage" class="text-center py-8 px-5 bg-white dark:bg-slate-900 rounded-3xl border border-slate-200/80 dark:border-slate-800 shadow-sm hidden my-4">
        <div class="w-14 h-14 mx-auto mb-3 rounded-2xl bg-teal-100 dark:bg-teal-950/60 flex items-center justify-center text-teal-600 dark:text-teal-400">
          <i class="fa-solid fa-circle-check text-2xl"></i>
        </div>
        <h3 id="study-complete-title" class="text-lg font-extrabold text-slate-800 dark:text-white mb-1">本日の学習が完了しました！</h3>
        <p id="study-complete-desc" class="text-xs text-slate-400 mb-4 max-w-xs mx-auto">
          設定された1日の学習上限に達したか、本日復習が必要なカードが完了しました。
        </p>

        <!-- Today Summary Card -->
        <div class="bg-slate-50 dark:bg-slate-800/70 p-3 rounded-2xl mb-4 max-w-xs mx-auto text-left border border-slate-100 dark:border-slate-700/60">
          <h4 class="text-[11px] font-bold text-slate-600 dark:text-slate-300 mb-2 flex items-center justify-between">
            <span><i class="fa-solid fa-calendar-check mr-1.5 text-teal-500"></i>本日の学習実績</span>
            <span class="text-[10px] text-slate-400" id="today-date-label"></span>
          </h4>
          <div class="grid grid-cols-2 gap-2 text-center text-xs">
            <div class="bg-white dark:bg-slate-900 p-2 rounded-xl border border-slate-200/60 dark:border-slate-800">
              <div class="text-[10px] text-slate-400">新規完了</div>
              <div id="today-finished-new" class="text-sm font-black text-blue-600 dark:text-blue-400">0</div>
            </div>
            <div class="bg-white dark:bg-slate-900 p-2 rounded-xl border border-slate-200/60 dark:border-slate-800">
              <div class="text-[10px] text-slate-400">復習完了</div>
              <div id="today-finished-due" class="text-sm font-black text-emerald-600 dark:text-emerald-400">0</div>
            </div>
          </div>
        </div>

        <div class="space-y-2 max-w-xs mx-auto">
          <button onclick="startCramMode()" class="w-full py-2.5 px-4 bg-teal-600 active:bg-teal-700 text-white font-bold rounded-xl text-xs shadow-sm active:scale-95 transition">
            <i class="fa-solid fa-bolt mr-1.5"></i>上限を無視して全単語復習
          </button>
          <button onclick="switchView('settings')" class="w-full py-2.5 px-4 bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-200 font-semibold rounded-xl text-xs active:scale-95 transition">
            <i class="fa-solid fa-sliders mr-1.5"></i>学習上限数を変更する
          </button>
        </div>
      </div>
    </section>

    <!-- 3. CARDS VIEW -->
    <section id="view-cards" class="view-section hidden space-y-3">
      <!-- Search & Filters Header -->
      <div class="bg-white dark:bg-slate-900 rounded-2xl border border-slate-200/80 dark:border-slate-800 p-3.5 shadow-sm space-y-2.5">
        <!-- Search bar -->
        <div class="relative">
          <i class="fa-solid fa-magnifying-glass absolute left-3 top-2.5 text-slate-400 text-xs"></i>
          <input type="text" id="card-search-input" oninput="renderCardsList()" placeholder="単語・意味・デッキで検索..." class="w-full pl-8 pr-3 py-1.5 text-xs bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-100 rounded-xl focus:ring-1 focus:ring-teal-500 focus:outline-none" />
        </div>

        <!-- Deck & Status Filters -->
        <div class="grid grid-cols-2 gap-2">
          <select id="card-deck-filter" onchange="renderCardsList()" class="text-xs bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-100 rounded-xl px-2.5 py-1.5 border-none truncate focus:ring-1 focus:ring-teal-500">
            <option value="all">すべてのデッキ</option>
          </select>
          <select id="card-status-filter" onchange="renderCardsList()" class="text-xs bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-100 rounded-xl px-2.5 py-1.5 border-none truncate focus:ring-1 focus:ring-teal-500">
            <option value="all">すべての状態</option>
            <option value="due">要復習</option>
            <option value="new">新規</option>
            <option value="learning">定着中</option>
            <option value="mastered">習得済み</option>
          </select>
        </div>

        <!-- Actions Bar -->
        <div class="flex items-center justify-between pt-1 border-t border-slate-100 dark:border-slate-800 text-xs">
          <div class="flex items-center space-x-1.5">
            <button onclick="openImportModal('__from_csv__')" class="bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 text-slate-700 dark:text-slate-300 text-[11px] font-semibold px-2.5 py-1 rounded-lg active:scale-95 transition flex items-center">
              <i class="fa-solid fa-file-import mr-1 text-teal-500"></i>CSV読込
            </button>
            <button onclick="downloadCSVTemplate()" class="bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 text-slate-700 dark:text-slate-300 text-[11px] font-semibold px-2.5 py-1 rounded-lg active:scale-95 transition">
              <i class="fa-solid fa-file-csv mr-1 text-teal-500"></i>雛形
            </button>
          </div>
          <button onclick="exportCardsJSON()" class="text-slate-500 text-[11px] px-2 py-1 rounded-lg hover:bg-slate-100 dark:hover:bg-slate-800">
            <i class="fa-solid fa-download mr-1"></i>バックアップ
          </button>
        </div>
      </div>

      <!-- Mobile Cards List Container -->
      <div id="cards-list-container" class="space-y-2">
        <!-- Dynamically populated cards -->
      </div>
      <div id="cards-empty-notice" class="p-8 text-center text-slate-400 text-xs hidden">
        該当する単語カードが見つかりません。
      </div>
    </section>

    <!-- 4. STATS VIEW -->
    <section id="view-stats" class="view-section hidden space-y-3">
      <div class="grid grid-cols-2 gap-2.5">
        <div class="bg-white dark:bg-slate-900 p-3.5 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm flex items-center space-x-3">
          <div class="w-10 h-10 rounded-xl bg-teal-50 dark:bg-teal-950/60 text-teal-600 flex items-center justify-center text-base flex-shrink-0">
            <i class="fa-solid fa-layer-group"></i>
          </div>
          <div>
            <p class="text-[10px] text-slate-400 font-bold uppercase">総単語数</p>
            <h4 id="stat-total" class="text-lg font-black text-slate-800 dark:text-white">0</h4>
          </div>
        </div>

        <div class="bg-white dark:bg-slate-900 p-3.5 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm flex items-center space-x-3">
          <div class="w-10 h-10 rounded-xl bg-emerald-50 dark:bg-emerald-950/60 text-emerald-600 flex items-center justify-center text-base flex-shrink-0">
            <i class="fa-solid fa-award"></i>
          </div>
          <div>
            <p class="text-[10px] text-slate-400 font-bold uppercase">習得済み</p>
            <h4 id="stat-mastered" class="text-lg font-black text-emerald-600 dark:text-emerald-400">0</h4>
          </div>
        </div>

        <div class="bg-white dark:bg-slate-900 p-3.5 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm flex items-center space-x-3">
          <div class="w-10 h-10 rounded-xl bg-amber-50 dark:bg-amber-950/60 text-amber-600 flex items-center justify-center text-base flex-shrink-0">
            <i class="fa-solid fa-fire"></i>
          </div>
          <div>
            <p class="text-[10px] text-slate-400 font-bold uppercase">復習待ち</p>
            <h4 id="stat-due" class="text-lg font-black text-amber-600 dark:text-amber-400">0</h4>
          </div>
        </div>

        <div class="bg-white dark:bg-slate-900 p-3.5 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm flex items-center space-x-3">
          <div class="w-10 h-10 rounded-xl bg-indigo-50 dark:bg-indigo-950/60 text-indigo-600 flex items-center justify-center text-base flex-shrink-0">
            <i class="fa-solid fa-retweet"></i>
          </div>
          <div>
            <p class="text-[10px] text-slate-400 font-bold uppercase">総復習回数</p>
            <h4 id="stat-reviews" class="text-lg font-black text-slate-800 dark:text-white">0</h4>
          </div>
        </div>
      </div>

      <!-- Retention Breakdown -->
      <div class="bg-white dark:bg-slate-900 p-4 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm">
        <h3 class="text-sm font-bold text-slate-800 dark:text-white mb-3">定着度の内訳</h3>
        
        <div class="space-y-3">
          <div>
            <div class="flex justify-between text-xs font-semibold mb-1">
              <span class="text-blue-600 dark:text-blue-400 flex items-center"><i class="fa-solid fa-seedling mr-1"></i>新規</span>
              <span id="bar-new-pct" class="text-slate-400 text-[11px]">0%</span>
            </div>
            <div class="w-full bg-slate-100 dark:bg-slate-800 h-2 rounded-full overflow-hidden">
              <div id="bar-new" class="bg-blue-500 h-full rounded-full transition-all duration-300" style="width: 0%"></div>
            </div>
          </div>

          <div>
            <div class="flex justify-between text-xs font-semibold mb-1">
              <span class="text-amber-600 dark:text-amber-400 flex items-center"><i class="fa-solid fa-arrows-rotate mr-1"></i>定着中 (1〜20日)</span>
              <span id="bar-learn-pct" class="text-slate-400 text-[11px]">0%</span>
            </div>
            <div class="w-full bg-slate-100 dark:bg-slate-800 h-2 rounded-full overflow-hidden">
              <div id="bar-learn" class="bg-amber-500 h-full rounded-full transition-all duration-300" style="width: 0%"></div>
            </div>
          </div>

          <div>
            <div class="flex justify-between text-xs font-semibold mb-1">
              <span class="text-emerald-600 dark:text-emerald-400 flex items-center"><i class="fa-solid fa-circle-check mr-1"></i>習得 (21日以上)</span>
              <span id="bar-master-pct" class="text-slate-400 text-[11px]">0%</span>
            </div>
            <div class="w-full bg-slate-100 dark:bg-slate-800 h-2 rounded-full overflow-hidden">
              <div id="bar-master" class="bg-emerald-500 h-full rounded-full transition-all duration-300" style="width: 0%"></div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- 5. SETTINGS VIEW -->
    <section id="view-settings" class="view-section hidden space-y-3">
      <!-- 1-day study limits card -->
      <div class="bg-white dark:bg-slate-900 p-4 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm space-y-4">
        <div class="flex items-center space-x-2 border-b border-slate-100 dark:border-slate-800 pb-2.5">
          <i class="fa-solid fa-sliders text-teal-500 text-sm"></i>
          <h3 class="text-sm font-bold text-slate-800 dark:text-white">1日の学習上限設定</h3>
        </div>

        <!-- Daily New Cards Limit -->
        <div class="space-y-1.5">
          <div class="flex items-center justify-between">
            <label class="text-xs font-bold text-slate-700 dark:text-slate-200">1日の新規単語カード上限</label>
            <span id="setting-new-limit-val" class="text-xs font-mono font-bold text-teal-600 dark:text-teal-400">20枚</span>
          </div>
          <p class="text-[11px] text-slate-400 leading-tight">1日に新しく学習する単語の上限枚数を設定します。</p>
          <div class="grid grid-cols-5 gap-1 pt-1">
            <button type="button" onclick="setDailyNewLimit(20)" class="limit-btn-new py-1.5 px-1 rounded-xl text-[11px] font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300" data-val="20">20枚</button>
            <button type="button" onclick="setDailyNewLimit(50)" class="limit-btn-new py-1.5 px-1 rounded-xl text-[11px] font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300" data-val="50">50枚</button>
            <button type="button" onclick="setDailyNewLimit(100)" class="limit-btn-new py-1.5 px-1 rounded-xl text-[11px] font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300" data-val="100">100枚</button>
            <button type="button" onclick="setDailyNewLimit(200)" class="limit-btn-new py-1.5 px-1 rounded-xl text-[11px] font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300" data-val="200">200枚</button>
            <button type="button" onclick="setDailyNewLimit(9999)" class="limit-btn-new py-1.5 px-1 rounded-xl text-[11px] font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300" data-val="9999">無制限</button>
          </div>
          <div class="flex items-center space-x-2 pt-1">
            <input type="number" id="custom-new-limit" min="1" max="1000" placeholder="カスタム数値 (例: 30)" onchange="setDailyNewLimit(this.value)" class="w-full text-xs px-3 py-1.5 bg-slate-50 dark:bg-slate-800/80 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500" />
          </div>
        </div>

        <div class="border-t border-slate-100 dark:border-slate-800 pt-3 space-y-1.5">
          <div class="flex items-center justify-between">
            <label class="text-xs font-bold text-slate-700 dark:text-slate-200">1日の最大復習カード上限</label>
            <span id="setting-due-limit-val" class="text-xs font-mono font-bold text-teal-600 dark:text-teal-400">50枚</span>
          </div>
          <p class="text-[11px] text-slate-400 leading-tight">1日に復習（過去に学んだ単語）する最大枚数を設定します。</p>
          <div class="grid grid-cols-5 gap-1 pt-1">
            <button type="button" onclick="setDailyDueLimit(50)" class="limit-btn-due py-1.5 px-1 rounded-xl text-[11px] font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300" data-val="50">50枚</button>
            <button type="button" onclick="setDailyDueLimit(100)" class="limit-btn-due py-1.5 px-1 rounded-xl text-[11px] font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300" data-val="100">100枚</button>
            <button type="button" onclick="setDailyDueLimit(200)" class="limit-btn-due py-1.5 px-1 rounded-xl text-[11px] font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300" data-val="200">200枚</button>
            <button type="button" onclick="setDailyDueLimit(500)" class="limit-btn-due py-1.5 px-1 rounded-xl text-[11px] font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300" data-val="500">500枚</button>
            <button type="button" onclick="setDailyDueLimit(9999)" class="limit-btn-due py-1.5 px-1 rounded-xl text-[11px] font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300" data-val="9999">無制限</button>
          </div>
          <div class="flex items-center space-x-2 pt-1">
            <input type="number" id="custom-due-limit" min="1" max="2000" placeholder="カスタム数値 (例: 150)" onchange="setDailyDueLimit(this.value)" class="w-full text-xs px-3 py-1.5 bg-slate-50 dark:bg-slate-800/80 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500" />
          </div>
        </div>
      </div>

      <!-- App Preference Options -->
      <div class="bg-white dark:bg-slate-900 p-4 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm space-y-3">
        <div class="flex items-center space-x-2 border-b border-slate-100 dark:border-slate-800 pb-2.5">
          <i class="fa-solid fa-gear text-teal-500 text-sm"></i>
          <h3 class="text-sm font-bold text-slate-800 dark:text-white">学習オプション</h3>
        </div>

        <div class="flex items-center justify-between py-1">
          <div>
            <div class="text-xs font-bold text-slate-800 dark:text-white">出題時に音声を自動再生</div>
            <div class="text-[11px] text-slate-400">カード表示時に自動で英語発音を再生</div>
          </div>
          <label class="relative inline-flex items-center cursor-pointer">
            <input type="checkbox" id="setting-auto-speak" onchange="toggleAutoSpeak(this.checked)" class="sr-only peer">
            <div class="w-9 h-5 bg-slate-200 peer-focus:outline-none rounded-full peer dark:bg-slate-700 peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-slate-300 after:border after:rounded-full after:h-4 after:w-4 after:transition-all peer-checked:bg-teal-600"></div>
          </label>
        </div>
      </div>

      <!-- Data Operations -->
      <div class="bg-white dark:bg-slate-900 p-4 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-sm space-y-2.5">
        <div class="flex items-center space-x-2 border-b border-slate-100 dark:border-slate-800 pb-2">
          <i class="fa-solid fa-database text-teal-500 text-sm"></i>
          <h3 class="text-sm font-bold text-slate-800 dark:text-white">データ管理</h3>
        </div>
        <div class="flex flex-col space-y-2 pt-1 text-xs">
          <button onclick="loadDefaultPresets()" class="w-full py-2.5 px-3 bg-teal-50 dark:bg-teal-950/40 text-teal-700 dark:text-teal-300 font-semibold rounded-xl text-left flex items-center justify-between active:scale-98 transition">
            <span><i class="fa-solid fa-rotate-left mr-2"></i>プリセット単語をロード</span>
            <i class="fa-solid fa-chevron-right text-[10px]"></i>
          </button>
          <button onclick="resetTodayReviewCount()" class="w-full py-2.5 px-3 bg-amber-50 dark:bg-amber-950/40 text-amber-700 dark:text-amber-400 font-semibold rounded-xl text-left flex items-center justify-between active:scale-98 transition">
            <span><i class="fa-solid fa-clock-rotate-left mr-2"></i>本日の学習カウントをリセット</span>
            <i class="fa-solid fa-chevron-right text-[10px]"></i>
          </button>
          <button onclick="clearAllDataModal()" class="w-full py-2.5 px-3 bg-red-50 dark:bg-red-950/40 text-red-600 dark:text-red-400 font-semibold rounded-xl text-left flex items-center justify-between active:scale-98 transition">
            <span><i class="fa-solid fa-trash-can mr-2"></i>すべての単語データを初期化</span>
            <i class="fa-solid fa-chevron-right text-[10px]"></i>
          </button>
        </div>
      </div>
    </section>

  </main>

  <nav class="fixed bottom-0 left-0 right-0 z-40 bg-white/95 dark:bg-slate-900/95 backdrop-blur-md border-t border-slate-200/80 dark:border-slate-800 safe-bottom">
    <div class="max-w-md mx-auto grid grid-cols-5 h-14">
      <button id="nav-decks" onclick="switchView('decks')" class="nav-btn flex flex-col items-center justify-center text-teal-600 dark:text-teal-400 font-bold transition-all">
        <i class="fa-solid fa-folder-tree text-base"></i>
        <span class="text-[10px] mt-0.5">デッキ</span>
      </button>

      <button id="nav-study" onclick="switchView('study')" class="nav-btn flex flex-col items-center justify-center text-slate-400 hover:text-slate-700 dark:hover:text-slate-200 transition-all relative">
        <div class="relative">
          <i class="fa-solid fa-graduation-cap text-lg"></i>
          <span id="nav-due-count" class="absolute -top-1 -right-2 px-1 text-[9px] rounded-full bg-teal-500 text-white font-bold leading-none min-w-[14px] h-[14px] flex items-center justify-center">0</span>
        </div>
        <span class="text-[10px] mt-0.5">学習</span>
      </button>

      <button id="nav-cards" onclick="switchView('cards')" class="nav-btn flex flex-col items-center justify-center text-slate-400 hover:text-slate-700 dark:hover:text-slate-200 transition-all relative">
        <div class="relative">
          <i class="fa-solid fa-layer-group text-base"></i>
          <span id="nav-total-count" class="absolute -top-1 -right-2.5 text-[9px] text-slate-400 font-semibold">0</span>
        </div>
        <span class="text-[10px] mt-0.5">単語帳</span>
      </button>

      <button id="nav-stats" onclick="switchView('stats')" class="nav-btn flex flex-col items-center justify-center text-slate-400 hover:text-slate-700 dark:hover:text-slate-200 transition-all">
        <i class="fa-solid fa-chart-pie text-base"></i>
        <span class="text-[10px] mt-0.5">統計</span>
      </button>

      <button id="nav-settings" onclick="switchView('settings')" class="nav-btn flex flex-col items-center justify-center text-slate-400 hover:text-slate-700 dark:hover:text-slate-200 transition-all">
        <i class="fa-solid fa-sliders text-base"></i>
        <span class="text-[10px] mt-0.5">設定</span>
      </button>
    </div>
  </nav>

  <!-- 1. Card Form Modal (Bottom Sheet style) -->
  <div id="card-modal" class="fixed inset-0 z-50 flex flex-col justify-end bg-slate-900/60 backdrop-blur-sm opacity-0 pointer-events-none transition-opacity duration-200">
    <div id="card-modal-box" class="bottom-sheet bottom-sheet-hidden bg-white dark:bg-slate-900 w-full max-w-lg mx-auto rounded-t-3xl border-t border-slate-200 dark:border-slate-800 shadow-2xl max-h-[90vh] flex flex-col">
      <div class="px-5 pt-3 pb-2 border-b border-slate-100 dark:border-slate-800 flex items-center justify-between relative">
        <div class="w-10 h-1 bg-slate-200 dark:bg-slate-700 rounded-full mx-auto absolute left-0 right-0 top-2.5"></div>
        <h3 id="modal-title" class="text-sm font-bold text-slate-800 dark:text-white mt-2">単語カードを追加</h3>
        <button onclick="closeCardModal()" class="w-8 h-8 rounded-full text-slate-400 hover:bg-slate-100 dark:hover:bg-slate-800 flex items-center justify-center">
          <i class="fa-solid fa-xmark text-sm"></i>
        </button>
      </div>

      <form id="card-form" onsubmit="saveCard(event)" class="p-4 space-y-3 overflow-y-auto flex-1">
        <input type="hidden" id="card-edit-id" value="" />
        
        <div>
          <label class="block text-[11px] font-bold text-slate-600 dark:text-slate-300 mb-1">表面 (単語 / 質問) <span class="text-red-500">*</span></label>
          <input type="text" id="form-front" required placeholder="例: ubiquitous" class="w-full px-3 py-2 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500 focus:outline-none" />
        </div>

        <div>
          <label class="block text-[11px] font-bold text-slate-600 dark:text-slate-300 mb-1">裏面 (意味 / 解答) <span class="text-red-500">*</span></label>
          <input type="text" id="form-back" required placeholder="例: どこにでもある、偏在する" class="w-full px-3 py-2 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500 focus:outline-none" />
        </div>

        <div>
          <label class="block text-[11px] font-bold text-slate-600 dark:text-slate-300 mb-1">所属デッキ</label>
          <select id="form-deck-select" onchange="onFormDeckChange(this.value)" class="w-full px-3 py-2 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500 mb-1">
          </select>
          <input type="text" id="form-deck-custom" placeholder="新規デッキ名 (例: 英語::TOEIC::Part5)" class="w-full px-3 py-1.5 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500 hidden" />
        </div>

        <div>
          <label class="block text-[11px] font-bold text-slate-600 dark:text-slate-300 mb-1">発音記号 (任意)</label>
          <input type="text" id="form-phonetic" placeholder="例: /juːˈbɪkwɪtəs/" class="w-full px-3 py-2 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500" />
        </div>

        <div>
          <label class="block text-[11px] font-bold text-slate-600 dark:text-slate-300 mb-1">例文と日本語訳 (任意)</label>
          <input type="text" id="form-example" placeholder="Smartphones have become ubiquitous." class="w-full px-3 py-2 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500 mb-1.5" />
          <input type="text" id="form-example-tr" placeholder="スマートフォンはどこにでもある存在になった。" class="w-full px-3 py-2 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500" />
        </div>

        <div>
          <label class="block text-[11px] font-bold text-slate-600 dark:text-slate-300 mb-1">メモ (任意)</label>
          <input type="text" id="form-note" placeholder="補足情報や重要ポイント" class="w-full px-3 py-2 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500" />
        </div>

        <div class="pt-2 flex items-center justify-end space-x-2">
          <button type="button" onclick="closeCardModal()" class="px-4 py-2 text-xs font-semibold text-slate-500 rounded-xl">
            キャンセル
          </button>
          <button type="submit" class="px-5 py-2 bg-teal-600 active:bg-teal-700 text-white text-xs font-bold rounded-xl shadow-md active:scale-95 transition">
            保存する
          </button>
        </div>
      </form>
    </div>
  </div>

  <!-- 2. Import Target Modal -->
  <div id="import-modal" class="fixed inset-0 z-50 flex flex-col justify-end bg-slate-900/60 backdrop-blur-sm opacity-0 pointer-events-none transition-opacity duration-200">
    <div id="import-modal-box" class="bottom-sheet bottom-sheet-hidden bg-white dark:bg-slate-900 w-full max-w-lg mx-auto rounded-t-3xl border-t border-slate-200 dark:border-slate-800 shadow-2xl p-5">
      <div class="w-10 h-1 bg-slate-200 dark:bg-slate-700 rounded-full mx-auto mb-3"></div>
      <div class="flex items-center justify-between mb-3">
        <h3 class="text-sm font-bold text-slate-800 dark:text-white flex items-center space-x-1.5">
          <i class="fa-solid fa-file-import text-teal-500"></i>
          <span>単語ファイル読み込み (CSV / JSON)</span>
        </h3>
        <button onclick="closeImportModal()" class="w-8 h-8 rounded-full text-slate-400 hover:bg-slate-100 dark:hover:bg-slate-800 flex items-center justify-center">
          <i class="fa-solid fa-xmark text-sm"></i>
        </button>
      </div>

      <div class="space-y-3.5">
        <div>
          <label class="block text-[11px] font-bold text-slate-600 dark:text-slate-300 mb-1">インポート先デッキの指定</label>
          <select id="import-deck-target-select" onchange="onImportDeckSelectChange(this.value)" class="w-full px-3 py-2 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500 mb-1.5">
            <option value="__from_csv__">CSV内のデッキ指定を優先（空欄時は一般）</option>
          </select>
          <input type="text" id="import-deck-custom" placeholder="新規デッキ名を入力 (例: 英語::TOEIC900)" class="w-full px-3 py-1.5 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500 hidden" />
          <p class="text-[10px] text-slate-400 mt-1">特定のデッキを選択すると、CSV内の単語をすべてそのデッキに登録します。</p>
        </div>

        <div class="pt-2">
          <label class="w-full py-3 px-4 bg-teal-600 active:bg-teal-700 text-white font-bold rounded-2xl text-xs shadow-md active:scale-95 transition flex items-center justify-center space-x-2 cursor-pointer">
            <i class="fa-solid fa-file-arrow-up text-sm"></i>
            <span>CSV / TSV / JSON ファイルを選択</span>
            <input type="file" id="modal-import-file" accept=".csv,.tsv,.txt,.json" onchange="handleModalFileImport(event)" class="hidden" />
          </label>
        </div>

        <div class="text-center pt-1">
          <button type="button" onclick="downloadCSVTemplate()" class="text-[11px] text-teal-600 dark:text-teal-400 font-semibold hover:underline">
            <i class="fa-solid fa-download mr-1"></i>インポート用CSV雛形をダウンロード
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- 3. Deck Form Modal -->
  <div id="deck-modal" class="fixed inset-0 z-50 flex flex-col justify-end bg-slate-900/60 backdrop-blur-sm opacity-0 pointer-events-none transition-opacity duration-200">
    <div id="deck-modal-box" class="bottom-sheet bottom-sheet-hidden bg-white dark:bg-slate-900 w-full max-w-lg mx-auto rounded-t-3xl border-t border-slate-200 dark:border-slate-800 shadow-2xl p-5">
      <div class="w-10 h-1 bg-slate-200 dark:bg-slate-700 rounded-full mx-auto mb-3"></div>
      <div class="flex items-center justify-between mb-3">
        <h3 id="deck-modal-title" class="text-sm font-bold text-slate-800 dark:text-white">デッキ作成</h3>
        <button onclick="closeDeckModal()" class="w-8 h-8 rounded-full text-slate-400 hover:bg-slate-100 dark:hover:bg-slate-800 flex items-center justify-center">
          <i class="fa-solid fa-xmark text-sm"></i>
        </button>
      </div>

      <form id="deck-form" onsubmit="saveDeck(event)" class="space-y-3">
        <input type="hidden" id="deck-edit-original" value="" />

        <div>
          <label class="block text-[11px] font-bold text-slate-600 dark:text-slate-300 mb-1">親デッキ (任意)</label>
          <select id="deck-parent-select" class="w-full px-3 py-2 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500">
            <option value="">(なし: トップレベルに作成)</option>
          </select>
        </div>

        <div>
          <label class="block text-[11px] font-bold text-slate-600 dark:text-slate-300 mb-1">デッキ名 <span class="text-red-500">*</span></label>
          <input type="text" id="deck-name-input" required placeholder="例: TOEIC 800点" class="w-full px-3 py-2 text-xs bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl focus:ring-1 focus:ring-teal-500" />
        </div>

        <div class="pt-2 flex items-center justify-end space-x-2">
          <button type="button" onclick="closeDeckModal()" class="px-4 py-2 text-xs font-semibold text-slate-500 rounded-xl">
            キャンセル
          </button>
          <button type="submit" class="px-5 py-2 bg-teal-600 active:bg-teal-700 text-white text-xs font-bold rounded-xl shadow-md active:scale-95 transition">
            保存する
          </button>
        </div>
      </form>
    </div>
  </div>

  <!-- 4. Confirm Modal -->
  <div id="confirm-modal" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm opacity-0 pointer-events-none transition-opacity duration-200">
    <div class="bg-white dark:bg-slate-900 w-full max-w-xs rounded-3xl shadow-2xl p-5 border border-slate-200 dark:border-slate-800 text-center">
      <div class="w-10 h-10 mx-auto mb-2 rounded-2xl bg-red-100 text-red-600 flex items-center justify-center text-lg">
        <i class="fa-solid fa-triangle-exclamation"></i>
      </div>
      <h4 id="confirm-title" class="text-sm font-bold text-slate-800 dark:text-white mb-1">確認</h4>
      <p id="confirm-msg" class="text-xs text-slate-400 mb-5">本当に実行しますか？</p>
      <div class="flex items-center space-x-2">
        <button id="confirm-btn-cancel" class="w-1/2 py-2 text-xs font-semibold text-slate-600 dark:text-slate-300 bg-slate-100 dark:bg-slate-800 rounded-xl">キャンセル</button>
        <button id="confirm-btn-ok" class="w-1/2 py-2 text-xs font-bold text-white bg-red-600 active:bg-red-700 rounded-xl">実行</button>
      </div>
    </div>
  </div>

  <!-- Toast Notification Container -->
  <div id="toast-container" class="fixed bottom-16 left-4 right-4 z-50 flex flex-col space-y-2 pointer-events-none items-center"></div>

  <script>
    const DEFAULT_PRESET_CARDS = [
      {
        id: "c1",
        front: "ubiquitous",
        back: "どこにでもある、偏在する",
        deck: "英語::TOEIC必修",
        phonetic: "/juːˈbɪkwɪtəs/",
        example: "Smartphones have become ubiquitous in modern society.",
        exampleTr: "スマートフォンは現代社会においてどこにでもある存在になった。",
        note: "",
        repetition: 0,
        interval: 0,
        easeFactor: 2.5,
        nextReview: new Date().toISOString(),
        lastReview: null,
        reviewCount: 0
      },
      {
        id: "c2",
        front: "meticulous",
        back: "細心の注意を払う、極めて几帳面な",
        deck: "英語::TOEIC必修",
        phonetic: "/məˈtɪkjələs/",
        example: "She did meticulous research before writing the report.",
        exampleTr: "彼女は報告書を書く前に綿密な調査を行った。",
        note: "",
        repetition: 0,
        interval: 0,
        easeFactor: 2.5,
        nextReview: new Date().toISOString(),
        lastReview: null,
        reviewCount: 0
      },
      {
        id: "c3",
        front: "pragmatic",
        back: "実践的な、実用主義の",
        deck: "英語::ビジネス::交渉",
        phonetic: "/præɡˈmætɪk/",
        example: "We need to take a pragmatic approach to solving this issue.",
        exampleTr: "私たちはこの問題を解決するために現実的なアプローチをとる必要がある。",
        note: "",
        repetition: 0,
        interval: 0,
        easeFactor: 2.5,
        nextReview: new Date().toISOString(),
        lastReview: null,
        reviewCount: 0
      },
      {
        id: "c4",
        front: "lucid",
        back: "明快な、分かりやすい",
        deck: "英語::TOEIC必修",
        phonetic: "/ˈluːsɪd/",
        example: "The professor gave a lucid explanation of the complex theory.",
        exampleTr: "教授はその複雑な理論について明快な説明をした。",
        note: "",
        repetition: 0,
        interval: 0,
        easeFactor: 2.5,
        nextReview: new Date().toISOString(),
        lastReview: null,
        reviewCount: 0
      },
      {
        id: "c5",
        front: "resilient",
        back: "回復力がある、立ち直りの早い",
        deck: "英語::ビジネス::メンタル",
        phonetic: "/rɪˈzɪliənt/",
        example: "Local businesses proved remarkably resilient during the crisis.",
        exampleTr: "地元企業は危機の最中に目覚ましい回復力を発揮した。",
        note: "",
        repetition: 0,
        interval: 0,
        easeFactor: 2.5,
        nextReview: new Date().toISOString(),
        lastReview: null,
        reviewCount: 0
      }
    ];

    let cards = [];
    let customDecks = [];
    let studyQueue = [];
    let currentCardIndex = 0;
    let isFlipped = false;
    let currentStudyDeck = 'all';
    let isCramMode = false;

    // Daily Learning Limits and App Settings
    let appSettings = {
      dailyNewLimit: 20,
      dailyDueLimit: 50,
      autoSpeak: false
    };

    let dailyStats = {
      date: new Date().toISOString().slice(0, 10),
      newCount: 0,
      dueCount: 0
    };

    function initApp() {
      // 1. Load Cards
      const savedCards = localStorage.getItem('flashsrs_cards');
      if (savedCards) {
        try { cards = JSON.parse(savedCards); } catch(e) { cards = [...DEFAULT_PRESET_CARDS]; }
      } else {
        cards = [...DEFAULT_PRESET_CARDS];
        saveToStorage();
      }

      // 2. Load Custom Decks
      const savedDecks = localStorage.getItem('flashsrs_decks');
      if (savedDecks) {
        try { customDecks = JSON.parse(savedDecks); } catch(e) { customDecks = []; }
      }

      // 3. Load Settings
      const savedSettings = localStorage.getItem('flashsrs_settings');
      if (savedSettings) {
        try { appSettings = { ...appSettings, ...JSON.parse(savedSettings) }; } catch(e) {}
      }

      // 4. Load Daily Stats (Reset if date changed)
      const todayStr = new Date().toISOString().slice(0, 10);
      const savedStats = localStorage.getItem('flashsrs_daily_stats');
      if (savedStats) {
        try {
          const parsed = JSON.parse(savedStats);
          if (parsed.date === todayStr) {
            dailyStats = parsed;
          } else {
            dailyStats = { date: todayStr, newCount: 0, dueCount: 0 };
            saveDailyStats();
          }
        } catch(e) {
          dailyStats = { date: todayStr, newCount: 0, dueCount: 0 };
        }
      } else {
        dailyStats = { date: todayStr, newCount: 0, dueCount: 0 };
        saveDailyStats();
      }

      // 5. Theme setup
      if (localStorage.getItem('flashsrs_theme') === 'dark' || 
          (!('flashsrs_theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
        document.documentElement.classList.add('dark');
        document.getElementById('theme-icon').className = 'fa-solid fa-sun text-xs';
      }

      setupTouchAndKeyboardShortcuts();
      updateDeckDropdowns();
      renderSettingsUI();
      updateGlobalCounters();

      // App initial entry screen: Deck list
      switchView('decks');
    }

    function saveToStorage() {
      localStorage.setItem('flashsrs_cards', JSON.stringify(cards));
      localStorage.setItem('flashsrs_decks', JSON.stringify(customDecks));
      updateGlobalCounters();
      updateDeckDropdowns();
    }

    function saveSettings() {
      localStorage.setItem('flashsrs_settings', JSON.stringify(appSettings));
      renderSettingsUI();
    }

    function saveDailyStats() {
      localStorage.setItem('flashsrs_daily_stats', JSON.stringify(dailyStats));
    }

    function isCardInDeckHierarchy(cardDeck, selectedDeck) {
      if (!selectedDeck || selectedDeck === 'all') return true;
      const cDeck = cardDeck || '一般';
      return cDeck === selectedDeck || cDeck.startsWith(selectedDeck + '::');
    }

    function getAllDeckPaths() {
      const set = new Set(customDecks);
      cards.forEach(c => {
        const deck = c.deck || '一般';
        set.add(deck);
        const parts = deck.split('::');
        let acc = '';
        for (let i = 0; i < parts.length; i++) {
          acc = acc ? `${acc}::${parts[i]}` : parts[i];
          set.add(acc);
        }
      });
      return Array.from(set).sort((a, b) => a.localeCompare(b, 'ja'));
    }

    function buildDeckTree() {
      const paths = getAllDeckPaths();
      const root = {};
      paths.forEach(fullPath => {
        const parts = fullPath.split('::');
        let current = root;
        let currPath = '';
        parts.forEach(part => {
          currPath = currPath ? `${currPath}::${part}` : part;
          if (!current[part]) {
            current[part] = { name: part, fullPath: currPath, children: {} };
          }
          current = current[part].children;
        });
      });
      return root;
    }

    function calculateSRS(card, rating) {
      let { repetition = 0, interval = 0, easeFactor = 2.5 } = card;
      const now = new Date();
      let nextIntervalDays = 1;

      if (rating === 1) {
        repetition = 0;
        nextIntervalDays = 0;
        easeFactor = Math.max(1.3, easeFactor - 0.2);
      } else if (rating === 2) {
        repetition = Math.max(1, repetition);
        nextIntervalDays = interval === 0 ? 1 : Math.round(interval * 1.2);
        easeFactor = Math.max(1.3, easeFactor - 0.15);
      } else if (rating === 3) {
        if (repetition === 0) nextIntervalDays = 1;
        else if (repetition === 1) nextIntervalDays = 3;
        else nextIntervalDays = Math.round(interval * easeFactor);
        repetition += 1;
      } else if (rating === 4) {
        if (repetition === 0) nextIntervalDays = 3;
        else if (repetition === 1) nextIntervalDays = 6;
        else nextIntervalDays = Math.round(interval * easeFactor * 1.35);
        repetition += 1;
        easeFactor += 0.15;
      }

      if (nextIntervalDays < 1 && rating > 1) nextIntervalDays = 1;

      let nextReviewDate = new Date();
      if (nextIntervalDays === 0) {
        nextReviewDate = new Date(now.getTime() + 10 * 60 * 1000);
      } else {
        nextReviewDate.setDate(now.getDate() + nextIntervalDays);
        nextReviewDate.setHours(4, 0, 0, 0);
      }

      return {
        repetition,
        interval: nextIntervalDays,
        easeFactor: Number(easeFactor.toFixed(2)),
        nextReview: nextReviewDate.toISOString(),
        lastReview: now.toISOString(),
        reviewCount: (card.reviewCount || 0) + 1
      };
    }

    function isCardDue(card) {
      if (!card.nextReview) return true;
      return new Date(card.nextReview) <= new Date();
    }

    function prepareStudySession() {
      // 1. Separate into New cards vs Due (Review) cards
      let allMatching = cards.filter(c => isCardInDeckHierarchy(c.deck, currentStudyDeck));

      let newCandidates = allMatching.filter(c => !c.reviewCount || c.reviewCount === 0);
      let dueCandidates = allMatching.filter(c => (c.reviewCount > 0) && isCardDue(c));

      // Calculate remaining daily allowance
      const newAllowed = Math.max(0, appSettings.dailyNewLimit - (dailyStats.newCount || 0));
      const dueAllowed = Math.max(0, appSettings.dailyDueLimit - (dailyStats.dueCount || 0));

      // Sort review cards (most overdue first)
      dueCandidates.sort((a, b) => new Date(a.nextReview || 0) - new Date(b.nextReview || 0));

      const selectedDue = dueCandidates.slice(0, dueAllowed);
      const selectedNew = newCandidates.slice(0, newAllowed);

      studyQueue = [...selectedDue, ...selectedNew];
      currentCardIndex = 0;
      isCramMode = false;

      // Update Limit Labels in Study Header
      const limitNewText = document.getElementById('limit-new-text');
      const limitDueText = document.getElementById('limit-due-text');
      if (limitNewText) limitNewText.innerText = appSettings.dailyNewLimit >= 9999 ? '∞' : appSettings.dailyNewLimit;
      if (limitDueText) limitDueText.innerText = appSettings.dailyDueLimit >= 9999 ? '∞' : appSettings.dailyDueLimit;

      renderCurrentCard();
      updateQueueBadges();
    }

    function startCramMode() {
      let filtered = cards.filter(c => isCardInDeckHierarchy(c.deck, currentStudyDeck));
      if (filtered.length === 0) {
        showToast('カードがありません', 'error');
        return;
      }
      studyQueue = [...filtered].sort(() => Math.random() - 0.5);
      currentCardIndex = 0;
      isCramMode = true;
      renderCurrentCard();
      showToast('上限を無視して全カード復習開始！', 'info');
    }

    function renderCurrentCard() {
      const stage = document.getElementById('study-stage');
      const emptyStage = document.getElementById('study-empty-stage');
      const cardInner = document.getElementById('flashcard-inner');
      const actionBar = document.getElementById('srs-action-bar');

      isFlipped = false;
      cardInner.classList.remove('card-flipped');
      actionBar.classList.add('opacity-30', 'pointer-events-none');

      if (studyQueue.length === 0 || currentCardIndex >= studyQueue.length) {
        stage.classList.add('hidden');
        emptyStage.classList.remove('hidden');
        document.getElementById('session-progress-text').innerText = '完了！';
        document.getElementById('session-progress-bar').style.width = '100%';

        // Display today's review counts
        const todayDateLabel = document.getElementById('today-date-label');
        const todayFinNew = document.getElementById('today-finished-new');
        const todayFinDue = document.getElementById('today-finished-due');
        if (todayDateLabel) todayDateLabel.innerText = dailyStats.date;
        if (todayFinNew) todayFinNew.innerText = `${dailyStats.newCount} 枚`;
        if (todayFinDue) todayFinDue.innerText = `${dailyStats.dueCount} 枚`;
        return;
      }

      stage.classList.remove('hidden');
      emptyStage.classList.add('hidden');

      const card = studyQueue[currentCardIndex];

      // Front
      document.getElementById('card-front-tag').innerText = card.deck || '一般';
      document.getElementById('card-front-text').innerText = card.front;
      document.getElementById('card-front-phonetic').innerText = card.phonetic || '';
      
      const hintElem = document.getElementById('card-front-hint');
      if (card.note) {
        hintElem.innerText = `💡 ${card.note}`;
        hintElem.classList.remove('hidden');
      } else {
        hintElem.classList.add('hidden');
      }

      // Back
      document.getElementById('card-back-tag').innerText = card.deck || '一般';
      document.getElementById('card-back-text').innerText = card.back;
      document.getElementById('card-interval-badge').innerText = `習熟: ${card.interval || 0}日`;

      const exampleBox = document.getElementById('card-example-box');
      if (card.example) {
        exampleBox.classList.remove('hidden');
        document.getElementById('card-back-example').innerText = card.example;
        document.getElementById('card-back-example-tr').innerText = card.exampleTr || '';
      } else {
        exampleBox.classList.add('hidden');
      }

      const noteElem = document.getElementById('card-back-note');
      if (card.note) {
        noteElem.innerText = `メモ: ${card.note}`;
        noteElem.classList.remove('hidden');
      } else {
        noteElem.classList.add('hidden');
      }

      // Button times
      const estHard = calculateSRS(card, 2);
      const estGood = calculateSRS(card, 3);
      const estEasy = calculateSRS(card, 4);

      document.getElementById('time-again').innerText = '<10分';
      document.getElementById('time-hard').innerText = `${estHard.interval}日後`;
      document.getElementById('time-good').innerText = `${estGood.interval}日後`;
      document.getElementById('time-easy').innerText = `${estEasy.interval}日後`;

      // Progress bar
      const remaining = studyQueue.length - currentCardIndex;
      document.getElementById('session-progress-text').innerText = `残り ${remaining} / ${studyQueue.length} 枚`;
      const pct = (currentCardIndex / studyQueue.length) * 100;
      document.getElementById('session-progress-bar').style.width = `${pct}%`;

      // Auto speak if enabled
      if (appSettings.autoSpeak) {
        setTimeout(speakCurrentWord, 200);
      }
    }

    function flipCard() {
      if (studyQueue.length === 0 || currentCardIndex >= studyQueue.length) return;
      isFlipped = !isFlipped;
      const cardInner = document.getElementById('flashcard-inner');
      const actionBar = document.getElementById('srs-action-bar');

      if (isFlipped) {
        cardInner.classList.add('card-flipped');
        actionBar.classList.remove('opacity-30', 'pointer-events-none');
      } else {
        cardInner.classList.remove('card-flipped');
        actionBar.classList.add('opacity-30', 'pointer-events-none');
      }
    }

    function rateCard(rating) {
      if (studyQueue.length === 0 || currentCardIndex >= studyQueue.length) return;

      const currentCard = studyQueue[currentCardIndex];
      const isInitialNew = !currentCard.reviewCount || currentCard.reviewCount === 0;

      const updatedStats = calculateSRS(currentCard, rating);

      const cardIdx = cards.findIndex(c => c.id === currentCard.id);
      if (cardIdx !== -1) {
        cards[cardIdx] = { ...cards[cardIdx], ...updatedStats };
        saveToStorage();
      }

      // Record daily limit stats (only on first pass in this session)
      if (!isCramMode) {
        if (isInitialNew) {
          dailyStats.newCount = (dailyStats.newCount || 0) + 1;
        } else {
          dailyStats.dueCount = (dailyStats.dueCount || 0) + 1;
        }
        saveDailyStats();
      }

      if (rating === 1 && !isCramMode && cardIdx !== -1) {
        studyQueue.push(cards[cardIdx]);
      }

      currentCardIndex++;
      renderCurrentCard();
      updateQueueBadges();
    }

    function resetSession() {
      prepareStudySession();
      showToast('セッションを更新しました', 'info');
    }

    function changeStudyDeck(deckName) {
      currentStudyDeck = deckName;
      prepareStudySession();
    }

    function speakCurrentWord() {
      if (studyQueue.length === 0 || currentCardIndex >= studyQueue.length) return;
      const card = studyQueue[currentCardIndex];
      if (!('speechSynthesis' in window)) {
        showToast('音声読み上げ非対応の環境です', 'error');
        return;
      }
      window.speechSynthesis.cancel();
      const utterance = new SpeechSynthesisUtterance(card.front);
      utterance.lang = 'en-US';
      utterance.rate = 0.9;
      window.speechSynthesis.speak(utterance);
    }

    function setupTouchAndKeyboardShortcuts() {
      window.addEventListener('keydown', (e) => {
        if (['INPUT', 'SELECT', 'TEXTAREA'].includes(e.target.tagName)) return;
        const studySection = document.getElementById('view-study');
        if (studySection.classList.contains('hidden')) return;

        if (e.code === 'Space') {
          e.preventDefault();
          flipCard();
        } else if (isFlipped) {
          if (e.key === '1') rateCard(1);
          if (e.key === '2') rateCard(2);
          if (e.key === '3') rateCard(3);
          if (e.key === '4') rateCard(4);
        }
      });
    }

    function switchView(viewName) {
      document.querySelectorAll('.view-section').forEach(el => el.classList.add('hidden'));
      document.querySelectorAll('.nav-btn').forEach(el => {
        el.classList.remove('text-teal-600', 'dark:text-teal-400');
        el.classList.add('text-slate-400');
      });

      const activeNav = document.getElementById(`nav-${viewName}`);
      if (activeNav) {
        activeNav.classList.remove('text-slate-400');
        activeNav.classList.add('text-teal-600', 'dark:text-teal-400');
      }

      if (viewName === 'study') {
        document.getElementById('view-study').classList.remove('hidden');
        prepareStudySession();
      } else if (viewName === 'decks') {
        document.getElementById('view-decks').classList.remove('hidden');
        renderDeckTree();
      } else if (viewName === 'cards') {
        document.getElementById('view-cards').classList.remove('hidden');
        renderCardsList();
      } else if (viewName === 'stats') {
        document.getElementById('view-stats').classList.remove('hidden');
        renderStats();
      } else if (viewName === 'settings') {
        document.getElementById('view-settings').classList.remove('hidden');
        renderSettingsUI();
      }
      window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    function renderSettingsUI() {
      // 1. New card limits
      const newLimitVal = document.getElementById('setting-new-limit-val');
      if (newLimitVal) {
        newLimitVal.innerText = appSettings.dailyNewLimit >= 9999 ? '無制限' : `${appSettings.dailyNewLimit}枚`;
      }
      document.querySelectorAll('.limit-btn-new').forEach(btn => {
        const val = parseInt(btn.dataset.val, 10);
        if (val === appSettings.dailyNewLimit) {
          btn.className = 'limit-btn-new py-1.5 px-2 rounded-xl text-xs font-bold bg-teal-600 text-white shadow-xs';
        } else {
          btn.className = 'limit-btn-new py-1.5 px-2 rounded-xl text-xs font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300';
        }
      });

      // 2. Due card limits
      const dueLimitVal = document.getElementById('setting-due-limit-val');
      if (dueLimitVal) {
        dueLimitVal.innerText = appSettings.dailyDueLimit >= 9999 ? '無制限' : `${appSettings.dailyDueLimit}枚`;
      }
      document.querySelectorAll('.limit-btn-due').forEach(btn => {
        const val = parseInt(btn.dataset.val, 10);
        if (val === appSettings.dailyDueLimit) {
          btn.className = 'limit-btn-due py-1.5 px-2 rounded-xl text-xs font-bold bg-teal-600 text-white shadow-xs';
        } else {
          btn.className = 'limit-btn-due py-1.5 px-2 rounded-xl text-xs font-semibold bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300';
        }
      });

      // 3. Audio toggle
      const autoSpeakCheckbox = document.getElementById('setting-auto-speak');
      if (autoSpeakCheckbox) {
        autoSpeakCheckbox.checked = !!appSettings.autoSpeak;
      }
    }

    function setDailyNewLimit(val) {
      const num = parseInt(val, 10);
      if (isNaN(num) || num <= 0) return;
      appSettings.dailyNewLimit = num;
      saveSettings();
      showToast(`新規カード上限を ${num >= 9999 ? '無制限' : num + '枚'} に設定しました`, 'success');
      prepareStudySession();
    }

    function setDailyDueLimit(val) {
      const num = parseInt(val, 10);
      if (isNaN(num) || num <= 0) return;
      appSettings.dailyDueLimit = num;
      saveSettings();
      showToast(`復習カード上限を ${num >= 9999 ? '無制限' : num + '枚'} に設定しました`, 'success');
      prepareStudySession();
    }

    function toggleAutoSpeak(checked) {
      appSettings.autoSpeak = !!checked;
      saveSettings();
      showToast(`音声自動再生を ${checked ? 'ON' : 'OFF'} にしました`, 'info');
    }

    function resetTodayReviewCount() {
      dailyStats = {
        date: new Date().toISOString().slice(0, 10),
        newCount: 0,
        dueCount: 0
      };
      saveDailyStats();
      prepareStudySession();
      showToast('本日の学習カウントをリセットしました', 'success');
    }

    function updateDeckDropdowns() {
      const deckPaths = getAllDeckPaths();

      const formatOption = (path) => {
        const depth = path.split('::').length - 1;
        const indent = '　'.repeat(depth) + (depth > 0 ? '↳ ' : '');
        const displayName = path.split('::').pop();
        return `<option value="${escapeHTML(path)}">${indent}${escapeHTML(displayName)}</option>`;
      };
      
      const studySelect = document.getElementById('study-deck-filter');
      if (studySelect) {
        studySelect.innerHTML = '<option value="all">すべてのデッキ（全体学習）</option>' + 
          deckPaths.map(d => `<option value="${escapeHTML(d)}" ${currentStudyDeck === d ? 'selected' : ''}>${escapeHTML(d)}</option>`).join('');
      }

      const cardDeckFilter = document.getElementById('card-deck-filter');
      if (cardDeckFilter) {
        const curVal = cardDeckFilter.value;
        cardDeckFilter.innerHTML = '<option value="all">すべてのデッキ</option>' + 
          deckPaths.map(d => `<option value="${escapeHTML(d)}" ${curVal === d ? 'selected' : ''}>${escapeHTML(d)}</option>`).join('');
      }

      const formDeckSelect = document.getElementById('form-deck-select');
      if (formDeckSelect) {
        formDeckSelect.innerHTML = deckPaths.map(d => formatOption(d)).join('') + '<option value="__custom__">＋ 新規デッキ名を入力...</option>';
      }

      const parentSelect = document.getElementById('deck-parent-select');
      if (parentSelect) {
        parentSelect.innerHTML = '<option value="">(なし: トップレベル)</option>' +
          deckPaths.map(d => `<option value="${escapeHTML(d)}">${escapeHTML(d)}</option>`).join('');
      }

      const importDeckSelect = document.getElementById('import-deck-target-select');
      if (importDeckSelect) {
        importDeckSelect.innerHTML = `
          <option value="__from_csv__">CSV内のデッキ指定を優先（空欄時は一般）</option>
          <optgroup label="既存デッキを選択">
            ${deckPaths.map(d => formatOption(d)).join('')}
          </optgroup>
          <option value="__custom__">＋ 新規デッキを作成してインポート...</option>
        `;
      }
    }

    function renderDeckTree() {
      const container = document.getElementById('deck-tree-container');
      const tree = buildDeckTree();
      container.innerHTML = '';

      const allDueCount = cards.filter(isCardDue).length;
      const allNewCount = cards.filter(c => !c.reviewCount || c.reviewCount === 0).length;
      const allTotal = cards.length;

      // "All Decks" Quick Play header card
      const allCardRow = document.createElement('div');
      allCardRow.className = 'flex items-center justify-between py-3 px-2 bg-teal-50/60 dark:bg-teal-950/30 rounded-xl mb-2 cursor-pointer border border-teal-100 dark:border-teal-900/60 active:scale-[0.99] transition';
      allCardRow.onclick = (e) => {
        if (!e.target.closest('button')) studySpecificDeck('all');
      };
      allCardRow.innerHTML = `
        <div class="flex items-center space-x-2.5 flex-1 min-w-0">
          <div class="w-8 h-8 rounded-lg bg-teal-600 text-white flex items-center justify-center text-xs flex-shrink-0">
            <i class="fa-solid fa-layer-group"></i>
          </div>
          <div class="truncate">
            <div class="flex items-center space-x-1.5">
              <span class="font-extrabold text-xs text-slate-900 dark:text-white">すべてのデッキ (全体学習)</span>
              <span class="text-[10px] text-teal-600 dark:text-teal-400 font-bold">(${allTotal})</span>
            </div>
            <div class="text-[10px] text-slate-500 dark:text-slate-400 mt-0.5">
              新:<span class="text-blue-500 font-bold">${allNewCount}</span> / 復:<span class="${allDueCount > 0 ? 'text-emerald-500 font-bold' : 'text-slate-400'}">${allDueCount}</span>
            </div>
          </div>
        </div>
        <div class="flex items-center space-x-1 flex-shrink-0">
          <button onclick="studySpecificDeck('all')" class="px-3 py-1.5 bg-teal-600 active:bg-teal-700 text-white text-[11px] font-bold rounded-xl active:scale-95 shadow-xs flex items-center space-x-1">
            <i class="fa-solid fa-play text-[9px]"></i>
            <span>学習</span>
          </button>
        </div>
      `;
      container.appendChild(allCardRow);

      const renderNode = (node, depth = 0) => {
        const fullPath = node.fullPath;
        const displayName = node.name;
        
        const matchingCards = cards.filter(c => isCardInDeckHierarchy(c.deck, fullPath));
        const newCount = matchingCards.filter(c => !c.reviewCount || c.reviewCount === 0).length;
        const dueCount = matchingCards.filter(isCardDue).length;
        const totalCount = matchingCards.length;

        const row = document.createElement('div');
        row.className = 'flex items-center justify-between py-2.5 px-2 active:bg-slate-50 dark:active:bg-slate-800/60 rounded-xl transition cursor-pointer';
        row.onclick = (e) => {
          if (!e.target.closest('button')) {
            studySpecificDeck(fullPath);
          }
        };
        
        const indentPadding = depth * 14;

        row.innerHTML = `
          <div class="flex items-center space-x-2 flex-1 min-w-0" style="padding-left: ${indentPadding}px">
            <i class="fa-solid ${depth === 0 ? 'fa-folder text-amber-500' : 'fa-folder-open text-teal-500'} text-xs flex-shrink-0"></i>
            <div class="truncate">
              <span class="font-bold text-xs text-slate-800 dark:text-white">${escapeHTML(displayName)}</span>
              <span class="text-[10px] text-slate-400 ml-1">(${totalCount})</span>
              <div class="text-[10px] text-slate-400">
                新:<span class="text-blue-500 font-bold">${newCount}</span> / 復:<span class="${dueCount > 0 ? 'text-emerald-500 font-bold' : 'text-slate-400'}">${dueCount}</span>
              </div>
            </div>
          </div>
          <div class="flex items-center space-x-1 flex-shrink-0">
            <button onclick="studySpecificDeck('${escapeHTML(fullPath)}')" class="px-2.5 py-1 bg-teal-600 active:bg-teal-700 text-white text-[11px] font-bold rounded-lg active:scale-95 shadow-xs flex items-center space-x-1">
              <i class="fa-solid fa-play text-[8px]"></i>
              <span>学習</span>
            </button>
            <button onclick="openImportModal('${escapeHTML(fullPath)}')" title="このデッキにCSV読込" class="p-1.5 text-slate-400 hover:text-teal-600 text-xs">
              <i class="fa-solid fa-file-import"></i>
            </button>
            <button onclick="openSubDeckModal('${escapeHTML(fullPath)}')" title="サブデッキ作成" class="p-1.5 text-slate-400 hover:text-teal-600 text-xs">
              <i class="fa-solid fa-folder-plus"></i>
            </button>
            <button onclick="openEditDeckModal('${escapeHTML(fullPath)}')" title="編集" class="p-1.5 text-slate-400 hover:text-indigo-600 text-xs">
              <i class="fa-solid fa-pen"></i>
            </button>
            <button onclick="deleteDeckHierarchy('${escapeHTML(fullPath)}')" title="削除" class="p-1.5 text-slate-400 hover:text-red-500 text-xs">
              <i class="fa-solid fa-trash-can"></i>
            </button>
          </div>
        `;
        container.appendChild(row);

        const childKeys = Object.keys(node.children).sort((a, b) => a.localeCompare(b, 'ja'));
        childKeys.forEach(k => renderNode(node.children[k], depth + 1));
      };

      const topKeys = Object.keys(tree).sort((a, b) => a.localeCompare(b, 'ja'));
      topKeys.forEach(k => renderNode(tree[k], 0));
    }

    function studySpecificDeck(deckPath) {
      currentStudyDeck = deckPath;
      switchView('study');
      updateDeckDropdowns();
    }

    function renderCardsList() {
      const searchInput = document.getElementById('card-search-input');
      const deckFilter = document.getElementById('card-deck-filter');
      const statusFilter = document.getElementById('card-status-filter');
      const container = document.getElementById('cards-list-container');
      const emptyNotice = document.getElementById('cards-empty-notice');

      const query = (searchInput?.value || '').toLowerCase();
      const selectedDeck = deckFilter?.value || 'all';
      const selectedStatus = statusFilter?.value || 'all';

      let filtered = cards.filter(card => {
        if (!isCardInDeckHierarchy(card.deck, selectedDeck)) return false;

        const isDue = isCardDue(card);
        const isNew = !card.reviewCount || card.reviewCount === 0;
        const isMastered = card.interval >= 21;
        const isLearning = card.reviewCount > 0 && card.interval < 21;

        if (selectedStatus === 'due' && !isDue) return false;
        if (selectedStatus === 'new' && !isNew) return false;
        if (selectedStatus === 'learning' && !isLearning) return false;
        if (selectedStatus === 'mastered' && !isMastered) return false;

        if (query) {
          const text = `${card.front} ${card.back} ${card.deck || ''} ${card.note || ''}`.toLowerCase();
          if (!text.includes(query)) return false;
        }
        return true;
      });

      container.innerHTML = '';
      if (filtered.length === 0) {
        emptyNotice.classList.remove('hidden');
        return;
      }
      emptyNotice.classList.add('hidden');

      filtered.forEach(card => {
        const isDue = isCardDue(card);
        let badge = '<span class="px-2 py-0.5 rounded-md text-[10px] font-bold bg-blue-50 text-blue-600 dark:bg-blue-950/50 dark:text-blue-400">新規</span>';
        if (card.interval >= 21) {
          badge = '<span class="px-2 py-0.5 rounded-md text-[10px] font-bold bg-emerald-50 text-emerald-600 dark:bg-emerald-950/50 dark:text-emerald-400">習得</span>';
        } else if (card.reviewCount > 0) {
          badge = '<span class="px-2 py-0.5 rounded-md text-[10px] font-bold bg-amber-50 text-amber-600 dark:bg-amber-950/50 dark:text-amber-400">定着中</span>';
        }

        const cardItem = document.createElement('div');
        cardItem.className = 'bg-white dark:bg-slate-900 p-3 rounded-2xl border border-slate-200/80 dark:border-slate-800 shadow-xs flex items-center justify-between gap-2';
        cardItem.innerHTML = `
          <div class="flex-1 min-w-0">
            <div class="flex items-center space-x-2">
              <span class="font-bold text-xs text-slate-900 dark:text-white truncate">${escapeHTML(card.front)}</span>
              ${badge}
              <span class="text-[10px] text-slate-400 font-mono">間隔: ${card.interval || 0}日</span>
            </div>
            <div class="text-xs text-teal-600 dark:text-teal-400 mt-0.5 truncate">${escapeHTML(card.back)}</div>
            <div class="text-[10px] text-slate-400 mt-1 flex items-center space-x-2">
              <span class="bg-slate-100 dark:bg-slate-800 px-1.5 py-0.5 rounded">${escapeHTML(card.deck || '一般')}</span>
              ${isDue ? '<span class="text-red-500 font-bold">要復習</span>' : ''}
            </div>
          </div>
          <div class="flex items-center space-x-1 flex-shrink-0">
            <button onclick="editCard('${card.id}')" title="編集" class="w-8 h-8 rounded-xl bg-slate-100 dark:bg-slate-800 text-slate-600 dark:text-slate-300 active:scale-95 flex items-center justify-center text-xs">
              <i class="fa-solid fa-pen"></i>
            </button>
            <button onclick="deleteCard('${card.id}')" title="削除" class="w-8 h-8 rounded-xl bg-slate-100 dark:bg-slate-800 text-slate-400 hover:text-red-500 active:scale-95 flex items-center justify-center text-xs">
              <i class="fa-solid fa-trash-can"></i>
            </button>
          </div>
        `;
        container.appendChild(cardItem);
      });
    }

    function renderStats() {
      const total = cards.length;
      const due = cards.filter(isCardDue).length;
      const mastered = cards.filter(c => c.interval >= 21).length;
      const learning = cards.filter(c => c.reviewCount > 0 && c.interval < 21).length;
      const newCards = cards.filter(c => !c.reviewCount || c.reviewCount === 0).length;
      const totalReviews = cards.reduce((acc, c) => acc + (c.reviewCount || 0), 0);

      document.getElementById('stat-total').innerText = total;
      document.getElementById('stat-due').innerText = due;
      document.getElementById('stat-mastered').innerText = mastered;
      document.getElementById('stat-reviews').innerText = totalReviews;

      if (total > 0) {
        const newPct = Math.round((newCards / total) * 100);
        const learnPct = Math.round((learning / total) * 100);
        const masterPct = Math.round((mastered / total) * 100);

        document.getElementById('bar-new').style.width = `${newPct}%`;
        document.getElementById('bar-new-pct').innerText = `${newCards}枚 (${newPct}%)`;

        document.getElementById('bar-learn').style.width = `${learnPct}%`;
        document.getElementById('bar-learn-pct').innerText = `${learning}枚 (${learnPct}%)`;

        document.getElementById('bar-master').style.width = `${masterPct}%`;
        document.getElementById('bar-master-pct').innerText = `${mastered}枚 (${masterPct}%)`;
      } else {
        document.getElementById('bar-new').style.width = '0%';
        document.getElementById('bar-learn').style.width = '0%';
        document.getElementById('bar-master').style.width = '0%';
      }
    }

    function updateGlobalCounters() {
      const dueCount = cards.filter(isCardDue).length;
      const dueEl = document.getElementById('nav-due-count');
      const totalEl = document.getElementById('nav-total-count');
      if (dueEl) dueEl.innerText = dueCount;
      if (totalEl) totalEl.innerText = cards.length;
    }

    function updateQueueBadges() {
      const newCount = studyQueue.filter(c => !c.reviewCount || c.reviewCount === 0).length;
      const learnCount = studyQueue.filter(c => c.reviewCount > 0 && c.interval < 21).length;
      const dueCount = studyQueue.filter(c => c.interval >= 21).length;

      const qNew = document.getElementById('queue-new-count');
      const qLearn = document.getElementById('queue-learn-count');
      const qDue = document.getElementById('queue-due-count');

      if (qNew) qNew.innerText = newCount;
      if (qLearn) qLearn.innerText = learnCount;
      if (qDue) qDue.innerText = dueCount;
      updateGlobalCounters();
    }

    function openCardModal(cardId = null) {
      const modal = document.getElementById('card-modal');
      const box = document.getElementById('card-modal-box');
      const title = document.getElementById('modal-title');
      const form = document.getElementById('card-form');

      form.reset();
      document.getElementById('card-edit-id').value = '';
      updateDeckDropdowns();

      const customInput = document.getElementById('form-deck-custom');
      customInput.classList.add('hidden');
      customInput.value = '';

      if (cardId) {
        const card = cards.find(c => c.id === cardId);
        if (card) {
          title.innerText = '単語カードを編集';
          document.getElementById('card-edit-id').value = card.id;
          document.getElementById('form-front').value = card.front;
          document.getElementById('form-back').value = card.back;
          
          const select = document.getElementById('form-deck-select');
          if (card.deck && Array.from(select.options).some(o => o.value === card.deck)) {
            select.value = card.deck;
          } else {
            select.value = '__custom__';
            customInput.classList.remove('hidden');
            customInput.value = card.deck || '';
          }

          document.getElementById('form-phonetic').value = card.phonetic || '';
          document.getElementById('form-example').value = card.example || '';
          document.getElementById('form-example-tr').value = card.exampleTr || '';
          document.getElementById('form-note').value = card.note || '';
        }
      } else {
        title.innerText = '単語カードを追加';
        const select = document.getElementById('form-deck-select');
        if (currentStudyDeck && currentStudyDeck !== 'all') {
          select.value = currentStudyDeck;
        }
      }

      modal.classList.remove('opacity-0', 'pointer-events-none');
      box.classList.remove('bottom-sheet-hidden');
      box.classList.add('bottom-sheet-visible');
    }

    function closeCardModal() {
      const modal = document.getElementById('card-modal');
      const box = document.getElementById('card-modal-box');
      box.classList.remove('bottom-sheet-visible');
      box.classList.add('bottom-sheet-hidden');
      setTimeout(() => {
        modal.classList.add('opacity-0', 'pointer-events-none');
      }, 200);
    }

    function onFormDeckChange(val) {
      const customInput = document.getElementById('form-deck-custom');
      if (val === '__custom__') {
        customInput.classList.remove('hidden');
        customInput.focus();
      } else {
        customInput.classList.add('hidden');
      }
    }

    function saveCard(e) {
      e.preventDefault();
      const editId = document.getElementById('card-edit-id').value;
      const front = document.getElementById('form-front').value.trim();
      const back = document.getElementById('form-back').value.trim();
      
      let deck = document.getElementById('form-deck-select').value;
      if (deck === '__custom__') {
        deck = document.getElementById('form-deck-custom').value.trim() || '一般';
        if (!customDecks.includes(deck)) customDecks.push(deck);
      }

      const phonetic = document.getElementById('form-phonetic').value.trim();
      const example = document.getElementById('form-example').value.trim();
      const exampleTr = document.getElementById('form-example-tr').value.trim();
      const note = document.getElementById('form-note').value.trim();

      if (!front || !back) {
        showToast('表面と裏面を入力してください', 'error');
        return;
      }

      if (editId) {
        const idx = cards.findIndex(c => c.id === editId);
        if (idx !== -1) {
          cards[idx] = { ...cards[idx], front, back, deck, phonetic, example, exampleTr, note };
          showToast('単語を更新しました', 'success');
        }
      } else {
        const newCard = {
          id: 'card_' + Date.now() + '_' + Math.random().toString(36).substr(2, 5),
          front, back, deck, phonetic, example, exampleTr, note,
          repetition: 0, interval: 0, easeFactor: 2.5,
          nextReview: new Date().toISOString(), lastReview: null, reviewCount: 0
        };
        cards.unshift(newCard);
        showToast('単語を追加しました', 'success');
      }

      saveToStorage();
      closeCardModal();
      renderCardsList();
      renderDeckTree();
      prepareStudySession();
    }

    function editCard(id) {
      openCardModal(id);
    }

    function deleteCard(id) {
      showConfirmModal('単語の削除', 'この単語カードを削除しますか？', () => {
        cards = cards.filter(c => c.id !== id);
        saveToStorage();
        renderCardsList();
        renderDeckTree();
        prepareStudySession();
        showToast('カードを削除しました', 'info');
      });
    }

    function openDeckModal() {
      const modal = document.getElementById('deck-modal');
      const box = document.getElementById('deck-modal-box');
      document.getElementById('deck-modal-title').innerText = '新規デッキ作成';
      document.getElementById('deck-form').reset();
      document.getElementById('deck-edit-original').value = '';
      updateDeckDropdowns();

      modal.classList.remove('opacity-0', 'pointer-events-none');
      box.classList.remove('bottom-sheet-hidden');
      box.classList.add('bottom-sheet-visible');
    }

    function openSubDeckModal(parentPath) {
      openDeckModal();
      document.getElementById('deck-parent-select').value = parentPath;
      document.getElementById('deck-name-input').focus();
    }

    function openEditDeckModal(deckPath) {
      openDeckModal();
      document.getElementById('deck-modal-title').innerText = 'デッキ名変更';
      document.getElementById('deck-edit-original').value = deckPath;

      const parts = deckPath.split('::');
      const name = parts.pop();
      const parent = parts.join('::');

      document.getElementById('deck-parent-select').value = parent;
      document.getElementById('deck-name-input').value = name;
    }

    function closeDeckModal() {
      const modal = document.getElementById('deck-modal');
      const box = document.getElementById('deck-modal-box');
      box.classList.remove('bottom-sheet-visible');
      box.classList.add('bottom-sheet-hidden');
      setTimeout(() => {
        modal.classList.add('opacity-0', 'pointer-events-none');
      }, 200);
    }

    function saveDeck(e) {
      e.preventDefault();
      const originalPath = document.getElementById('deck-edit-original').value;
      const parent = document.getElementById('deck-parent-select').value.trim();
      const name = document.getElementById('deck-name-input').value.trim().replace(/::/g, '-');

      if (!name) return;
      const newPath = parent ? `${parent}::${name}` : name;

      if (originalPath) {
        cards.forEach(c => {
          if (c.deck === originalPath) c.deck = newPath;
          else if (c.deck && c.deck.startsWith(originalPath + '::')) {
            c.deck = newPath + c.deck.substring(originalPath.length);
          }
        });
        customDecks = customDecks.map(d => {
          if (d === originalPath) return newPath;
          if (d.startsWith(originalPath + '::')) return newPath + d.substring(originalPath.length);
          return d;
        });
        showToast('デッキ名を更新しました', 'success');
      } else {
        if (!customDecks.includes(newPath)) customDecks.push(newPath);
        showToast('デッキを作成しました', 'success');
      }

      saveToStorage();
      closeDeckModal();
      renderDeckTree();
    }

    function deleteDeckHierarchy(deckPath) {
      showConfirmModal('デッキの削除', `「${deckPath}」および配下の単語をすべて削除しますか？`, () => {
        cards = cards.filter(c => !isCardInDeckHierarchy(c.deck, deckPath));
        customDecks = customDecks.filter(d => !isCardInDeckHierarchy(d, deckPath));
        saveToStorage();
        renderDeckTree();
        renderCardsList();
        prepareStudySession();
        showToast('デッキを削除しました', 'info');
      });
    }

    function openImportModal(targetDeck = '__from_csv__') {
      const modal = document.getElementById('import-modal');
      const box = document.getElementById('import-modal-box');
      const select = document.getElementById('import-deck-target-select');
      const customInput = document.getElementById('import-deck-custom');

      updateDeckDropdowns();

      if (targetDeck && targetDeck !== '__from_csv__') {
        select.value = targetDeck;
      } else {
        select.value = '__from_csv__';
      }

      customInput.classList.add('hidden');
      customInput.value = '';

      modal.classList.remove('opacity-0', 'pointer-events-none');
      box.classList.remove('bottom-sheet-hidden');
      box.classList.add('bottom-sheet-visible');
    }

    function closeImportModal() {
      const modal = document.getElementById('import-modal');
      const box = document.getElementById('import-modal-box');
      box.classList.remove('bottom-sheet-visible');
      box.classList.add('bottom-sheet-hidden');
      setTimeout(() => {
        modal.classList.add('opacity-0', 'pointer-events-none');
      }, 200);
    }

    function onImportDeckSelectChange(val) {
      const customInput = document.getElementById('import-deck-custom');
      if (val === '__custom__') {
        customInput.classList.remove('hidden');
        customInput.focus();
      } else {
        customInput.classList.add('hidden');
      }
    }

    function handleModalFileImport(event) {
      const file = event.target.files[0];
      if (!file) return;

      const selectVal = document.getElementById('import-deck-target-select').value;
      let targetDeckOverride = null;

      if (selectVal === '__custom__') {
        targetDeckOverride = document.getElementById('import-deck-custom').value.trim();
        if (targetDeckOverride && !customDecks.includes(targetDeckOverride)) {
          customDecks.push(targetDeckOverride);
        }
      } else if (selectVal !== '__from_csv__') {
        targetDeckOverride = selectVal;
      }

      const fileName = file.name.toLowerCase();
      const reader = new FileReader();

      reader.onload = function(e) {
        const content = e.target.result;
        try {
          if (fileName.endsWith('.json')) {
            importCardsJSONContent(content, targetDeckOverride);
          } else {
            importCardsCSVContent(content, targetDeckOverride);
          }
          closeImportModal();
        } catch (err) {
          showToast('読込エラー: ' + (err.message || ''), 'error');
        }
      };

      reader.readAsText(file, 'utf-8');
      event.target.value = '';
    }

    function parseCSV(text) {
      const rows = [];
      let currentRow = [];
      let currentVal = '';
      let insideQuote = false;

      const firstLine = text.split(/\r\n|\r|\n/)[0] || '';
      const delimiter = firstLine.includes('\t') && !firstLine.includes(',') ? '\t' : ',';

      for (let i = 0; i < text.length; i++) {
        const char = text[i];
        const nextChar = text[i + 1];

        if (char === '"') {
          if (insideQuote && nextChar === '"') {
            currentVal += '"';
            i++;
          } else {
            insideQuote = !insideQuote;
          }
        } else if (char === delimiter && !insideQuote) {
          currentRow.push(currentVal.trim());
          currentVal = '';
        } else if ((char === '\r' || char === '\n') && !insideQuote) {
          if (char === '\r' && nextChar === '\n') i++;
          currentRow.push(currentVal.trim());
          if (currentRow.some(val => val.length > 0)) rows.push(currentRow);
          currentRow = [];
          currentVal = '';
        } else {
          currentVal += char;
        }
      }

      if (currentVal || currentRow.length > 0) {
        currentRow.push(currentVal.trim());
        if (currentRow.some(val => val.length > 0)) rows.push(currentRow);
      }
      return rows;
    }

    function importCardsCSVContent(content, targetDeckOverride = null) {
      const rows = parseCSV(content);
      if (!rows || rows.length === 0) {
        showToast('CSVデータが見つかりません', 'error');
        return;
      }

      let startIndex = 0;
      const headerKeywords = ['表面', 'front', '単語', 'word', 'term', 'question', '裏面', 'meaning', 'back'];
      const firstRowLower = rows[0].map(c => c.toLowerCase());
      if (firstRowLower.some(cell => headerKeywords.some(k => cell.includes(k)))) {
        startIndex = 1;
      }

      let addedCount = 0;
      const nowISO = new Date().toISOString();

      for (let i = startIndex; i < rows.length; i++) {
        const row = rows[i];
        if (row.length < 2) continue;

        const front = row[0] || '';
        const back = row[1] || '';
        if (!front || !back) continue;

        let deck = targetDeckOverride ? targetDeckOverride : (row[2] || '英語::一般');
        const phonetic = row[3] || '';
        const example = row[4] || '';
        const exampleTr = row[5] || '';
        const note = row[6] || '';

        const existingIdx = cards.findIndex(c => c.front.trim().toLowerCase() === front.trim().toLowerCase() && c.deck === deck);
        
        if (existingIdx !== -1) {
          cards[existingIdx] = {
            ...cards[existingIdx],
            back,
            phonetic: phonetic || cards[existingIdx].phonetic,
            example: example || cards[existingIdx].example,
            exampleTr: exampleTr || cards[existingIdx].exampleTr,
            note: note || cards[existingIdx].note
          };
        } else {
          cards.push({
            id: 'csv_' + Date.now() + '_' + Math.random().toString(36).substr(2, 6),
            front, back, deck, phonetic, example, exampleTr, note,
            repetition: 0, interval: 0, easeFactor: 2.5,
            nextReview: nowISO, lastReview: null, reviewCount: 0
          });
        }
        addedCount++;
      }

      if (addedCount > 0) {
        saveToStorage();
        renderCardsList();
        renderDeckTree();
        prepareStudySession();
        showToast(`${addedCount}件の単語をインポートしました`, 'success');
      } else {
        showToast('有効な単語データがありませんでした', 'error');
      }
    }

    function importCardsJSONContent(content, targetDeckOverride = null) {
      const imported = JSON.parse(content);
      if (Array.isArray(imported)) {
        if (targetDeckOverride) {
          imported.forEach(c => c.deck = targetDeckOverride);
        }
        cards = imported;
        saveToStorage();
        renderCardsList();
        renderDeckTree();
        prepareStudySession();
        showToast(`${imported.length}件の単語を復元しました`, 'success');
      } else {
        showToast('無効なJSONです', 'error');
      }
    }

    function exportCardsJSON() {
      const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(cards, null, 2));
      const downloadAnchor = document.createElement('a');
      downloadAnchor.setAttribute("href", dataStr);
      downloadAnchor.setAttribute("download", `FlashSRS_backup_${new Date().toISOString().slice(0,10)}.json`);
      document.body.appendChild(downloadAnchor);
      downloadAnchor.click();
      downloadAnchor.remove();
      showToast('単語データをエクスポートしました', 'success');
    }

    function downloadCSVTemplate() {
      const sampleCSV = `表面(必須),裏面(必須),デッキ名(階層は::),発音記号,例文,例文日本語訳,メモ
ubiquitous,どこにでもある・偏在する,英語::TOEIC必修,/juːˈbɪkwɪtəs/,Smartphones are ubiquitous.,スマホはどこにでもある。,
pragmatic,実践的な・実用主義の,英語::ビジネス::交渉,/præɡˈmætɪk/,We need a pragmatic solution.,現実的な解決策が必要だ。,`;

      const bom = new Uint8Array([0xEF, 0xBB, 0xBF]);
      const blob = new Blob([bom, sampleCSV], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'FlashSRS_単語インポート雛形.csv';
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
      showToast('CSV雛形をダウンロードしました', 'info');
    }

    function loadDefaultPresets() {
      DEFAULT_PRESET_CARDS.forEach(p => {
        if (!cards.some(c => c.front === p.front && c.deck === p.deck)) {
          cards.push({ ...p, id: 'preset_' + Date.now() + '_' + Math.random().toString(36).substr(2, 4) });
        }
      });
      saveToStorage();
      prepareStudySession();
      renderCardsList();
      renderDeckTree();
      showToast('プリセット単語をロードしました', 'success');
    }

    function clearAllDataModal() {
      showConfirmModal('全データ初期化', '単語と学習履歴が消去されます。よろしいですか？', () => {
        cards = [];
        customDecks = [];
        dailyStats = {
          date: new Date().toISOString().slice(0, 10),
          newCount: 0,
          dueCount: 0
        };
        saveToStorage();
        saveDailyStats();
        prepareStudySession();
        renderCardsList();
        renderDeckTree();
        renderStats();
        showToast('全データをリセットしました', 'info');
      });
    }

    function showConfirmModal(title, message, onConfirm) {
      const modal = document.getElementById('confirm-modal');
      document.getElementById('confirm-title').innerText = title;
      document.getElementById('confirm-msg').innerText = message;
      
      const btnCancel = document.getElementById('confirm-btn-cancel');
      const btnOk = document.getElementById('confirm-btn-ok');

      const close = () => {
        modal.classList.add('opacity-0', 'pointer-events-none');
      };

      btnCancel.onclick = close;
      btnOk.onclick = () => {
        close();
        if (onConfirm) onConfirm();
      };

      modal.classList.remove('opacity-0', 'pointer-events-none');
    }

    function showToast(msg, type = 'info') {
      const container = document.getElementById('toast-container');
      const toast = document.createElement('div');
      
      let bg = 'bg-slate-900 text-white';
      let icon = 'fa-circle-info text-teal-400';
      if (type === 'success') {
        bg = 'bg-emerald-600 text-white';
        icon = 'fa-circle-check';
      } else if (type === 'error') {
        bg = 'bg-red-600 text-white';
        icon = 'fa-circle-exclamation';
      }

      toast.className = `${bg} pointer-events-auto px-4 py-2 rounded-2xl shadow-xl flex items-center space-x-2 text-xs font-semibold transform transition-all duration-200 translate-y-2 opacity-0 max-w-sm`;
      toast.innerHTML = `<i class="fa-solid ${icon}"></i><span>${escapeHTML(msg)}</span>`;

      container.appendChild(toast);
      
      requestAnimationFrame(() => {
        toast.classList.remove('translate-y-2', 'opacity-0');
      });

      setTimeout(() => {
        toast.classList.add('opacity-0', 'translate-y-2');
        setTimeout(() => toast.remove(), 250);
      }, 2300);
    }

    function toggleDarkMode() {
      const html = document.documentElement;
      const isDark = html.classList.toggle('dark');
      localStorage.setItem('flashsrs_theme', isDark ? 'dark' : 'light');
      document.getElementById('theme-icon').className = isDark ? 'fa-solid fa-sun text-xs' : 'fa-solid fa-moon text-xs';
    }

    function escapeHTML(str) {
      if (!str) return '';
      return String(str).replace(/[&<>'"]/g, 
        tag => ({
          '&': '&amp;',
          '<': '&lt;',
          '>': '&gt;',
          "'": '&#39;',
          '"': '&quot;'
        }[tag] || tag)
      );
    }

    window.onload = initApp;
  </script>
</body>
</html>

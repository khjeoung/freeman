<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>작가용 모바일/아이패드 웹앱 (Novel Studio)</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- FontAwesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    /* 아이패드 및 모바일 스크롤 스무스 설정 */
    body { -webkit-tap-highlight-color: transparent; }
    ::-webkit-scrollbar { width: 4px; }
    ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
  </style>
</head>
<body class="bg-slate-100 text-slate-800 h-screen flex flex-col overflow-hidden font-sans">

  <!-- 상단 헤더 -->
  <header class="bg-white border-b border-slate-200 px-4 py-2.5 flex items-center justify-between shadow-sm">
    <div class="flex items-center space-x-3">
      <button onclick="toggleSidebar()" class="p-2 text-slate-600 hover:bg-slate-100 rounded-lg">
        <i class="fa-solid fa-bars text-lg"></i>
      </button>
      <h1 class="font-bold text-lg text-slate-800 flex items-center gap-2">
        <i class="fa-solid fa-feather-pointed text-indigo-600"></i> Novel Studio
      </h1>
    </div>
    
    <div class="flex items-center space-x-2">
      <span id="saveStatus" class="text-xs text-slate-400 font-medium mr-2">저장됨</span>
      <button onclick="toggleCharacterPanel()" class="px-3 py-1.5 text-xs font-semibold bg-indigo-50 text-indigo-600 rounded-lg hover:bg-indigo-100 flex items-center gap-1.5">
        <i class="fa-solid fa-user-gear"></i> 캐릭터 프로필
      </button>
    </div>
  </header>

  <!-- 메인 콘텐트 영역 -->
  <div class="flex-1 flex overflow-hidden relative">

    <!-- 1. 사이드바: 챕터 & 장면 목록 (스크리브너 방식) -->
    <aside id="sidebar" class="w-64 bg-white border-r border-slate-200 flex flex-col transition-all duration-300 z-20 absolute md:relative h-full shadow-lg md:shadow-none">
      <div class="p-3 border-b border-slate-100 flex justify-between items-center bg-slate-50">
        <span class="text-xs font-bold text-slate-500 uppercase tracking-wider">목차 (Outline)</span>
        <button onclick="addScene()" class="p-1 text-indigo-600 hover:bg-indigo-50 rounded">
          <i class="fa-solid fa-plus"></i> 장면 추가
        </button>
      </div>
      <div id="sceneList" class="flex-1 overflow-y-auto p-2 space-y-1">
        <!-- 동적으로 추가되는 장면 목록 -->
      </div>
    </aside>

    <!-- 2. 중앙 집필 에디터 -->
    <main class="flex-1 bg-white flex flex-col overflow-hidden relative">
      <!-- 에디터 상단 툴바 -->
      <div class="border-b border-slate-100 px-6 py-3 flex justify-between items-center bg-white">
        <input type="text" id="documentTitle" value="1화. 첫 번째 문장" class="text-xl font-bold text-slate-800 outline-none w-full bg-transparent" oninput="saveData()">
        <span id="charCount" class="text-xs font-medium text-slate-400 whitespace-nowrap ml-4">공백 포함 0자</span>
      </div>
      
      <!-- 본문 에디터 (웹소설/장편 몰입용) -->
      <textarea id="editor" 
        class="flex-1 p-6 text-slate-700 leading-relaxed text-base md:text-lg outline-none resize-none overflow-y-auto bg-slate-50/30"
        placeholder="여기에 이야기를 시작하세요..."
        oninput="handleEditorInput()"></textarea>
    </main>

    <!-- 3. 우측 패널: 캐릭터 설정 노트 (의상, 헤어, 가슴 컵 등) -->
    <aside id="characterPanel" class="w-80 bg-white border-l border-slate-200 flex flex-col transition-all duration-300 z-20 absolute right-0 h-full shadow-xl md:shadow-none translate-x-full md:translate-x-0 hidden md:flex">
      <div class="p-3 border-b border-slate-100 flex justify-between items-center bg-slate-50">
        <span class="text-xs font-bold text-slate-500 uppercase tracking-wider">캐릭터 상세 설정</span>
        <button onclick="toggleCharacterPanel()" class="md:hidden p-1 text-slate-400 hover:text-slate-600">
          <i class="fa-solid fa-xmark"></i>
        </button>
      </div>

      <div class="flex-1 overflow-y-auto p-4 space-y-4 text-sm">
        <!-- 인물 기본적인 정보 -->
        <div>
          <label class="block text-xs font-bold text-slate-500 mb-1">이름 / 성별</label>
          <div class="grid grid-cols-2 gap-2">
            <input type="text" id="charName" placeholder="이름" class="w-full border rounded p-1.5 text-xs outline-none focus:border-indigo-500" onchange="saveData()">
            <select id="charGender" class="w-full border rounded p-1.5 text-xs outline-none focus:border-indigo-500" onchange="saveData()">
              <option value="여성">여성</option>
              <option value="남성">남성</option>
              <option value="기타">기타</option>
            </select>
          </div>
        </div>

        <!-- 헤어 스타일 & 길이 -->
        <div>
          <label class="block text-xs font-bold text-slate-500 mb-1">헤어 스타일 & 길이</label>
          <input type="text" id="charHair" placeholder="예: 검은색 긴 생머리, 숏컷 등" class="w-full border rounded p-1.5 text-xs outline-none focus:border-indigo-500" onchange="saveData()">
        </div>

        <!-- 얼굴 특징 -->
        <div>
          <label class="block text-xs font-bold text-slate-500 mb-1">얼굴 특징 / 인상</label>
          <input type="text" id="charFace" placeholder="예: 고양이상, 날카로운 눈매" class="w-full border rounded p-1.5 text-xs outline-none focus:border-indigo-500" onchange="saveData()">
        </div>

        <!-- 체형 & 신체 스펙 -->
        <div>
          <label class="block text-xs font-bold text-slate-500 mb-1">체형 및 비율</label>
          <input type="text" id="charBody" placeholder="예: 슬림한 체형, 175cm" class="w-full border rounded p-1.5 text-xs outline-none focus:border-indigo-500" onchange="saveData()">
        </div>

        <!-- 세부 스펙 (가슴 컵 사이즈 등) -->
        <div>
          <label class="block text-xs font-bold text-slate-500 mb-1">세부 신체 레퍼런스 (컵 사이즈 등)</label>
          <input type="text" id="charBust" placeholder="예: C컵 / 넓은 어깨 등" class="w-full border rounded p-1.5 text-xs outline-none focus:border-indigo-500" onchange="saveData()">
        </div>

        <!-- 의상 스타일 -->
        <div>
          <label class="block text-xs font-bold text-slate-500 mb-1">주요 의상 레퍼런스</label>
          <textarea id="charOutfit" rows="3" placeholder="예: 제복, 캐주얼 셔츠, 디테일한 액세서리 등" class="w-full border rounded p-1.5 text-xs outline-none focus:border-indigo-500 resize-none" onchange="saveData()"></textarea>
        </div>
      </div>
    </aside>

  </div>

  <script>
    // 앱 내부 상태 관리
    let saveTimeout = null;

    // 사이드바 및 프로필 토글 기능 (아이패드/모바일 터치용)
    function toggleSidebar() {
      const sidebar = document.getElementById('sidebar');
      sidebar.classList.toggle('-translate-x-full');
    }

    function toggleCharacterPanel() {
      const panel = document.getElementById('characterPanel');
      panel.classList.toggle('hidden');
      panel.classList.toggle('flex');
    }

    // 글자 수 실시간 계산 및 자동 저장 트리거
    function handleEditorInput() {
      const editor = document.getElementById('editor');
      const charCount = document.getElementById('charCount');
      
      const length = editor.value.length;
      charCount.innerText = `공백 포함 ${length.toLocaleString()}자`;

      // 입력을 멈춘 후 1초 뒤 자동 저장
      document.getElementById('saveStatus').innerText = '저장 중...';
      clearTimeout(saveTimeout);
      saveTimeout = setTimeout(saveData, 1000);
    }

    // 로컬 스토리지에 데이터 저장
    function saveData() {
      const data = {
        title: document.getElementById('documentTitle').value,
        content: document.getElementById('editor').value,
        character: {
          name: document.getElementById('charName').value,
          gender: document.getElementById('charGender').value,
          hair: document.getElementById('charHair').value,
          face: document.getElementById('charFace').value,
          body: document.getElementById('charBody').value,
          bust: document.getElementById('charBust').value,
          outfit: document.getElementById('charOutfit').value
        }
      };

      localStorage.setItem('novel_studio_data', JSON.stringify(data));
      document.getElementById('saveStatus').innerText = '저장됨';
    }

    // 앱 로드시 저장된 데이터 불러오기
    function loadData() {
      const saved = localStorage.getItem('novel_studio_data');
      if (saved) {
        const data = JSON.parse(saved);
        document.getElementById('documentTitle').value = data.title || '';
        document.getElementById('editor').value = data.content || '';
        
        if (data.character) {
          document.getElementById('charName').value = data.character.name || '';
          document.getElementById('charGender').value = data.character.gender || '여성';
          document.getElementById('charHair').value = data.character.hair || '';
          document.getElementById('charFace').value = data.character.face || '';
          document.getElementById('charBody').value = data.character.body || '';
          document.getElementById('charBust').value = data.character.bust || '';
          document.getElementById('charOutfit').value = data.character.outfit || '';
        }

        handleEditorInput();
      }
    }

    window.onload = loadData;
  </script>
</body>
</html>

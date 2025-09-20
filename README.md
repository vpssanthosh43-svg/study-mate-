# study-mate-
 Here i submit frontend source code 

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>StudyMate — PDF Q&A</title>

  <!-- Tailwind CDN for fast attractive UI (good for hackathons) -->
  <script src="https://cdn.tailwindcss.com"></script>

  <!-- Heroicons (optional) -->
  <script src="https://unpkg.com/feather-icons"></script>

  <style>
    /* small custom tweaks */
    body { background: linear-gradient(180deg,#f8fafc,#eef2ff); }
    .glass { backdrop-filter: blur(6px); -webkit-backdrop-filter: blur(6px); background: rgba(255,255,255,0.65); }
    .file-dashed { border-style: dashed; border-width: 2px; border-color: rgba(99,102,241,0.35); }
    .scroll-smooth { scroll-behavior: smooth; }
  </style>
</head>
<body class="min-h-screen flex items-center justify-center p-6">

  <div class="max-w-5xl w-full grid grid-cols-1 md:grid-cols-3 gap-6">
    <!-- Left: Upload / PDF info -->
    <div class="md:col-span-1 glass rounded-2xl p-6 shadow-xl">
      <div class="flex items-center justify-between">
        <h2 class="text-2xl font-semibold text-indigo-700">📘 StudyMate</h2>
        <div class="text-sm text-gray-500">PDF Q&A • Hackathon</div>
      </div>

      <p class="mt-3 text-sm text-gray-600">Upload your textbook / notes PDF and ask questions. The backend should run at <code class="text-xs font-mono">http://127.0.0.1:5000</code>.</p>

      <!-- Drag & Drop / File Input -->
      <div id="dropZone" class="file-dashed mt-5 p-4 rounded-xl flex flex-col items-center justify-center text-center cursor-pointer hover:bg-indigo-50 transition">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 text-indigo-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M3 15a4 4 0 004 4h10a4 4 0 004-4V7a4 4 0 00-4-4H7a4 4 0 00-4 4v8z" />
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M8 11l3-3 3 3m0 6V8" />
        </svg>
        <div class="mt-3">
          <div id="fileName" class="text-sm font-medium text-gray-800">Drag & drop PDF here or click to select</div>
          <div class="text-xs text-gray-500 mt-1">Supports single PDF. Max file size depends on your backend.</div>
        </div>

        <input id="pdfInput" type="file" accept="application/pdf" class="sr-only" />
      </div>

      <!-- Upload buttons -->
      <div class="mt-4 flex gap-2">
        <button id="btnUpload" class="flex-1 py-2 px-4 bg-indigo-600 text-white rounded-lg font-medium shadow hover:bg-indigo-700">Upload PDF</button>
        <button id="btnClear" class="py-2 px-4 border rounded-lg text-indigo-600 font-medium hover:bg-indigo-50">Clear</button>
      </div>

      <!-- Upload progress and status -->
      <div class="mt-4">
        <div id="uploadStatus" class="text-sm text-gray-600">No file uploaded yet.</div>
        <div id="progressWrap" class="w-full bg-gray-200 rounded-full h-2 mt-3 hidden">
          <div id="progressBar" class="h-2 rounded-full bg-indigo-500 w-0"></div>
        </div>
      </div>

      <!-- PDF meta -->
      <div class="mt-6 text-sm text-gray-700 border-t pt-4">
        <div><strong>PDF ID:</strong> <span id="pdfId" class="font-mono text-indigo-600">—</span></div>
        <div class="mt-2"><strong>Pages (preview):</strong> <span id="pageCount">—</span></div>
        <div class="mt-2"><strong>Backend status:</strong> <span id="backendStatus" class="text-green-600">Unknown</span></div>
      </div>
    </div>

    <!-- Middle: Chat / Q&A -->
    <div class="glass md:col-span-2 rounded-2xl p-6 shadow-xl flex flex-col">
      <!-- Q&A header -->
      <div class="flex items-center justify-between">
        <div>
          <h3 class="text-xl font-semibold text-gray-800">Ask Questions</h3>
          <p class="text-sm text-gray-500">Type any question about the uploaded PDF. The answer will be fetched from your backend AI engine.</p>
        </div>
        <div class="text-xs text-gray-400">Demo UI</div>
      </div>

      <!-- Chat area -->
      <div id="chatArea" class="mt-6 flex-1 overflow-auto pr-2 space-y-4">
        <!-- messages will be appended here -->
        <div id="welcomeMsg" class="text-sm text-gray-500">No conversation yet — upload a PDF to begin.</div>
      </div>

      <!-- Input area -->
      <div class="mt-4 pt-4 border-t">
        <div class="flex gap-3">
          <input id="questionInput" type="text" placeholder="E.g. What is polymorphism in OOP? (Hint: upload a PDF first)" class="flex-1 p-3 rounded-lg border focus:outline-none"/>
          <button id="btnAsk" class="px-6 py-3 bg-emerald-500 text-white rounded-lg font-medium hover:bg-emerald-600">Ask</button>
        </div>
        <div class="flex items-center justify-between mt-3 text-xs text-gray-400">
          <div id="lastAction">Ready</div>
          <div class="flex gap-2">
            <button id="btnSummarize" class="px-3 py-1 rounded-lg border text-gray-600 hover:bg-gray-50">Summarize PDF</button>
            <button id="btnFlash" class="px-3 py-1 rounded-lg border text-gray-600 hover:bg-gray-50">Generate Flashcards</button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- small footer with tips -->
  <div class="fixed bottom-6 left-6 text-xs text-gray-500 bg-white/70 glass px-3 py-2 rounded-lg shadow">
    Tip: If upload fails, ensure your backend is running and has CORS enabled (pip install flask-cors; CORS(app)).
  </div>

  <script>
    // ---------- config ----------
    const API_BASE = "http://127.0.0.1:5000"; // change if your backend is elsewhere

    // UI elements
    const dropZone = document.getElementById('dropZone');
    const pdfInput = document.getElementById('pdfInput');
    const fileName = document.getElementById('fileName');
    const btnUpload = document.getElementById('btnUpload');
    const btnClear = document.getElementById('btnClear');
    const uploadStatus = document.getElementById('uploadStatus');
    const progressWrap = document.getElementById('progressWrap');
    const progressBar = document.getElementById('progressBar');
    const pdfIdEl = document.getElementById('pdfId');
    const pageCountEl = document.getElementById('pageCount');
    const backendStatusEl = document.getElementById('backendStatus');

    const chatArea = document.getElementById('chatArea');
    const questionInput = document.getElementById('questionInput');
    const btnAsk = document.getElementById('btnAsk');
    const lastAction = document.getElementById('lastAction');
    const btnSummarize = document.getElementById('btnSummarize');
    const btnFlash = document.getElementById('btnFlash');

    let selectedFile = null;
    let currentPdfId = null;

    // ping backend to show status
    async function pingBackend() {
      try {
        const res = await fetch(API_BASE + '/', { method: 'GET' });
        backendStatusEl.textContent = "Online";
        backendStatusEl.className = "text-green-600";
      } catch (e) {
        backendStatusEl.textContent = "Offline";
        backendStatusEl.className = "text-red-600";
      }
    }
    pingBackend();

    // drag/drop and click to open file picker
    dropZone.addEventListener('click', () => pdfInput.click());
    dropZone.addEventListener('dragover', e => { e.preventDefault(); dropZone.classList.add('bg-indigo-50'); });
    dropZone.addEventListener('dragleave', e => { e.preventDefault(); dropZone.classList.remove('bg-indigo-50'); });
    dropZone.addEventListener('drop', e => {
      e.preventDefault(); dropZone.classList.remove('bg-indigo-50');
      if (e.dataTransfer.files && e.dataTransfer.files.length) {
        handleSelectedFile(e.dataTransfer.files[0]);
      }
    });

    pdfInput.addEventListener('change', (e) => {
      if (e.target.files && e.target.files[0]) handleSelectedFile(e.target.files[0]);
    });

    function handleSelectedFile(file) {
      if (file.type !== 'application/pdf') {
        alert('Please select a PDF file.');
        return;
      }
      selectedFile = file;
      fileName.textContent = ${file.name} • ${(file.size/1024/1024).toFixed(2)} MB;
      uploadStatus.textContent = "Ready to upload.";
    }

    // clear
    btnClear.addEventListener('click', () => {
      selectedFile = null;
      currentPdfId = null;
      pdfInput.value = "";
      fileName.textContent = "Drag & drop PDF here or click to select";
      uploadStatus.textContent = "No file uploaded yet.";
      pdfIdEl.textContent = "—";
      pageCountEl.textContent = "—";
      chatArea.innerHTML = '<div id="welcomeMsg" class="text-sm text-gray-500">No conversation yet — upload a PDF to begin.</div>';
    });

    // upload function with progress and better error reporting
    btnUpload.addEventListener('click', async () => {
      if (!selectedFile) { alert('Select a PDF first'); return; }

      const fd = new FormData();
      fd.append('file', selectedFile);

      uploadStatus.textContent = "Uploading...";
      progressWrap.classList.remove('hidden');
      progressBar.style.width = '0%';

      try {
        // Use XMLHttpRequest to show progress
        await new Promise((resolve, reject) => {
          const xhr = new XMLHttpRequest();
          xhr.open('POST', API_BASE + '/upload', true);

          xhr.upload.onprogress = (e) => {
            if (e.lengthComputable) {
              const pct = Math.round((e.loaded / e.total) * 100);
              progressBar.style.width = pct + '%';
            }
          };

          xhr.onload = () => {
            if (xhr.status >= 200 && xhr.status < 300) {
              try {
                const data = JSON.parse(xhr.responseText);
                currentPdfId = data.pdf_id || data.pdfId || null;
                pdfIdEl.textContent = currentPdfId || '—';
                uploadStatus.textContent = 'Upload complete';
                lastAction.textContent = 'PDF uploaded';
                appendSystemMessage('PDF uploaded successfully ✅');
                // if backend returned pages count, use it
                if (data.page_count) pageCountEl.textContent = data.page_count;
                resolve();
              } catch (err) {
                reject('Invalid JSON response from server');
              }
            } else {
              // try to parse server error message
              let msg = Upload failed (status ${xhr.status});
              try { const data = JSON.parse(xhr.responseText); if (data.error) msg = data.error; } catch(e){}
              reject(msg);
            }
          };

          xhr.onerror = () => reject('Network error during upload');
          xhr.send(fd);
        });
      } catch (errMsg) {
        uploadStatus.textContent = 'Upload error';
        appendSystemMessage('❌ Upload failed: ' + errMsg);
        alert('Upload error: ' + errMsg);
      } finally {
        progressWrap.classList.add('hidden');
        progressBar.style.width = '0%';
      }
    });

    // helper: append messages to chat area
    function appendMessage(role, text) {
      const wrapper = document.createElement('div');
      wrapper.className = (role === 'user') ? 'flex justify-end' : 'flex justify-start';
      const bubble = document.createElement('div');
      bubble.className = max-w-[80%] p-3 rounded-xl shadow ${role === 'user' ? 'bg-indigo-600 text-white' : 'bg-white text-gray-800'};
      bubble.innerText = text;
      wrapper.appendChild(bubble);
      chatArea.appendChild(wrapper);
      chatArea.scrollTop = chatArea.scrollHeight;
    }
    function appendSystemMessage(text) {
      const el = document.createElement('div');
      el.className = 'text-xs text-gray-500 italic';
      el.innerText = text;
      chatArea.appendChild(el);
      chatArea.scrollTop = chatArea.scrollHeight;
    }

    // Ask question handler
    btnAsk.addEventListener('click', async () => {
      const q = questionInput.value.trim();
      if (!q) return alert('Type a question first!');
      if (!currentPdfId) return alert('Upload a PDF first.');

      appendMessage('user', q);
      questionInput.value = '';
      lastAction.textContent = 'Waiting for answer...';

      try {
        const res = await fetch(API_BASE + '/ask', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ pdf_id: currentPdfId, question: q })
        });

        const data = await (res.headers.get('content-type') || '').includes('application/json') ? res.json() : { answer: await res.text() };

        if (!res.ok) {
          const errMsg = (data && (data.error || data.message || data.answer)) || Server returned ${res.status};
          appendMessage('assistant', "Error: " + errMsg);
          lastAction.textContent = 'Error';
          return;
        }

        const answerText = data.answer || data.result || 'No answer returned.';
        appendMessage('assistant', answerText);
        lastAction.textContent = 'Answered';
      } catch (err) {
        appendMessage('assistant', 'Network error: Could not contact backend.');
        lastAction.textContent = 'Network error';
      }
    });

    // quick actions (these call /ask with a special question; backend should handle or you can adjust)
    btnSummarize.addEventListener('click', () => {
      questionInput.value = 'Please summarize the uploaded PDF in 5–7 bullet points.';
      btnAsk.click();
    });
    btnFlash.addEventListener('click', () => {
      questionInput.value = 'Generate 5 flashcards (Q: / A:) from the main topics in this PDF.';
      btnAsk.click();
    });

    // small UX: press Enter to ask
    questionInput.addEventListener('keydown', e => {
      if (e.key === 'Enter') {
        e.preventDefault();
        btnAsk.click();
      }
    });

    // display server root if exists (attempt once)
    (async () => {
      try {
        const r = await fetch(API_BASE + '/', { method: 'GET' });
        if (r.ok) backendStatusEl.textContent = 'Online';
      } catch(e) {
        backendStatusEl.textContent = 'Offline';
        backendStatusEl.className = "text-red-600";
      }
    })();

    // init small welcome message
    appendSystemMessage('Welcome — upload a PDF and start asking questions. Good luck with the hackathon!');

    // enable feather icons
    try { feather.replace(); } catch(e){}
  </script>
</body>
</html>

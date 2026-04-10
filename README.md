# watermark-999
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Watermark Remover</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .drop-zone {
            border: 2px dashed #cbd5e1;
            transition: all 0.3s ease;
        }
        .drop-zone.drag-over {
            border-color: #3b82f6;
            background-color: #eff6ff;
        }
        .image-preview {
            max-width: 150px;
            max-height: 150px;
            object-fit: cover;
        }
        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
        }
        ::-webkit-scrollbar-thumb {
            background: #888;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #555;
        }
    </style>
</head>
<body class="bg-slate-50 min-h-screen font-sans">
    <div class="max-w-6xl mx-auto p-4 md:p-8">
        <header class="mb-8 text-center">
            <h1 class="text-4xl font-extrabold text-slate-800 mb-2">AI Watermark Remover</h1>
            <p class="text-slate-600">High-quality batch processing with intelligent AI detection.</p>
        </header>

        <!-- Main Workspace -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            
            <!-- Left Panel: Controls & Upload -->
            <div class="lg:col-span-1 space-y-6">
                <div id="dropZone" class="drop-zone bg-white rounded-2xl p-8 text-center cursor-pointer shadow-sm">
                    <div class="flex flex-col items-center">
                        <svg class="w-12 h-12 text-blue-500 mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"></path>
                        </svg>
                        <p class="text-lg font-medium text-slate-700">Drag & Drop Images</p>
                        <p class="text-sm text-slate-500 mt-1">or click to browse files</p>
                        <input type="file" id="fileInput" multiple accept="image/*" class="hidden">
                    </div>
                </div>

                <div class="bg-white rounded-2xl p-6 shadow-sm">
                    <h2 class="text-lg font-semibold mb-4 text-slate-800">Processing Mode</h2>
                    <div class="space-y-3">
                        <label class="flex items-center space-x-3 p-3 rounded-lg border border-slate-200 cursor-pointer hover:bg-slate-50 transition">
                            <input type="radio" name="mode" value="auto" checked class="w-4 h-4 text-blue-600">
                            <div>
                                <p class="font-medium text-slate-700">Auto-Detect</p>
                                <p class="text-xs text-slate-500">AI finds the watermark automatically</p>
                            </div>
                        </label>
                        <label class="flex items-center space-x-3 p-3 rounded-lg border border-slate-200 cursor-pointer hover:bg-slate-50 transition">
                            <input type="radio" name="mode" value="grid" class="w-4 h-4 text-blue-600">
                            <div>
                                <p class="font-medium text-slate-700">Pattern Focus</p>
                                <p class="text-xs text-slate-500">Optimized for repeating grid patterns</p>
                            </div>
                        </label>
                    </div>

                    <div class="mt-6 pt-6 border-t border-slate-100">
                        <button id="processAllBtn" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-xl transition shadow-lg shadow-blue-200 disabled:opacity-50 disabled:cursor-not-allowed">
                            Process All Files
                        </button>
                    </div>
                </div>

                <div id="statusMessage" class="hidden p-4 rounded-xl text-sm font-medium"></div>
            </div>

            <!-- Right Panel: File Queue & Results -->
            <div class="lg:col-span-2 space-y-6">
                <div class="bg-white rounded-2xl p-6 shadow-sm min-h-[400px]">
                    <div class="flex items-center justify-between mb-6">
                        <h2 class="text-xl font-bold text-slate-800">Processing Queue <span id="fileCount" class="text-sm font-normal text-slate-400">(0)</span></h2>
                        <button id="clearBtn" class="text-sm text-red-500 hover:underline">Clear All</button>
                    </div>

                    <div id="queueList" class="space-y-4">
                        <!-- Empty State -->
                        <div id="emptyState" class="flex flex-col items-center justify-center py-20 text-slate-400">
                            <svg class="w-16 h-16 mb-4 opacity-20" fill="currentColor" viewBox="0 0 20 20">
                                <path d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z"></path>
                            </svg>
                            <p>No images added yet</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Processing Modal / Indicator -->
    <div id="processingOverlay" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 flex items-center justify-center hidden">
        <div class="bg-white p-8 rounded-3xl shadow-2xl text-center max-w-xs w-full">
            <div class="relative w-24 h-24 mx-auto mb-6">
                <div class="absolute inset-0 border-4 border-blue-100 rounded-full"></div>
                <div class="absolute inset-0 border-4 border-blue-600 rounded-full border-t-transparent animate-spin"></div>
            </div>
            <h3 class="text-xl font-bold text-slate-800 mb-2">AI is Thinking...</h3>
            <p id="overlayText" class="text-slate-500">Detecting watermarks and rebuilding pixels...</p>
        </div>
    </div>

    <script type="module">
        const apiKey = ""; // Environment provides this automatically
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'watermark-remover';

        const fileInput = document.getElementById('fileInput');
        const dropZone = document.getElementById('dropZone');
        const queueList = document.getElementById('queueList');
        const emptyState = document.getElementById('emptyState');
        const fileCountLabel = document.getElementById('fileCount');
        const clearBtn = document.getElementById('clearBtn');
        const processAllBtn = document.getElementById('processAllBtn');
        const processingOverlay = document.getElementById('processingOverlay');
        const overlayText = document.getElementById('overlayText');
        const statusMessage = document.getElementById('statusMessage');

        let queue = [];

        // --- Event Listeners ---

        dropZone.onclick = () => fileInput.click();
        
        fileInput.onchange = (e) => {
            handleFiles(e.target.files);
        };

        dropZone.ondragover = (e) => {
            e.preventDefault();
            dropZone.classList.add('drag-over');
        };

        dropZone.ondragleave = () => {
            dropZone.classList.remove('drag-over');
        };

        dropZone.ondrop = (e) => {
            e.preventDefault();
            dropZone.classList.remove('drag-over');
            handleFiles(e.dataTransfer.files);
        };

        clearBtn.onclick = () => {
            queue = [];
            renderQueue();
        };

        processAllBtn.onclick = async () => {
            if (queue.length === 0) return;
            
            processAllBtn.disabled = true;
            processingOverlay.classList.remove('hidden');
            
            for (let i = 0; i < queue.length; i++) {
                if (queue[i].status === 'completed') continue;
                
                overlayText.innerText = `Processing image ${i + 1} of ${queue.length}...`;
                await removeWatermarkAI(i);
            }
            
            processingOverlay.classList.add('hidden');
            processAllBtn.disabled = false;
        };

        // --- Core Functions ---

        function handleFiles(files) {
            const newFiles = Array.from(files).filter(f => f.type.startsWith('image/'));
            
            newFiles.forEach(file => {
                const reader = new FileReader();
                reader.onload = (e) => {
                    queue.push({
                        id: Math.random().toString(36).substr(2, 9),
                        file: file,
                        dataUrl: e.target.result,
                        status: 'pending', // pending, processing, completed, error
                        resultUrl: null
                    });
                    renderQueue();
                };
                reader.readAsDataURL(file);
            });
        }

        function renderQueue() {
            if (queue.length === 0) {
                emptyState.classList.remove('hidden');
                queueList.querySelectorAll('.queue-item').forEach(el => el.remove());
                fileCountLabel.innerText = `(0)`;
                processAllBtn.disabled = true;
                return;
            }

            emptyState.classList.add('hidden');
            processAllBtn.disabled = false;
            fileCountLabel.innerText = `(${queue.length})`;

            // Clear list except empty state
            queueList.querySelectorAll('.queue-item').forEach(el => el.remove());

            queue.forEach((item, index) => {
                const itemEl = document.createElement('div');
                itemEl.className = 'queue-item bg-slate-50 rounded-xl p-4 flex items-center justify-between group';
                
                const statusBadge = getStatusBadge(item.status);

                itemEl.innerHTML = `
                    <div class="flex items-center space-x-4">
                        <img src="${item.dataUrl}" class="image-preview rounded-lg shadow-sm border border-slate-200">
                        <div>
                            <p class="font-medium text-slate-800 truncate max-w-[200px]">${item.file.name}</p>
                            <p class="text-xs text-slate-400">${(item.file.size / 1024).toFixed(1)} KB</p>
                            <div class="mt-1">${statusBadge}</div>
                        </div>
                    </div>
                    <div class="flex items-center space-x-2">
                        ${item.status === 'completed' ? `
                            <button onclick="downloadImage('${item.resultUrl}', '${item.file.name}')" class="bg-emerald-500 hover:bg-emerald-600 text-white p-2 rounded-lg transition">
                                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a2 2 0 002 2h12a2 2 0 002-2v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
                            </button>
                        ` : ''}
                        <button onclick="removeFromQueue(${index})" class="text-slate-300 hover:text-red-500 transition group-hover:opacity-100 opacity-50 p-2">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                        </button>
                    </div>
                `;
                queueList.appendChild(itemEl);
            });
        }

        function getStatusBadge(status) {
            switch(status) {
                case 'pending': return '<span class="text-[10px] uppercase tracking-wider font-bold text-slate-400">Ready</span>';
                case 'processing': return '<span class="text-[10px] uppercase tracking-wider font-bold text-blue-500 animate-pulse">Processing...</span>';
                case 'completed': return '<span class="text-[10px] uppercase tracking-wider font-bold text-emerald-500">Cleaned</span>';
                case 'error': return '<span class="text-[10px] uppercase tracking-wider font-bold text-red-500">Failed</span>';
                default: return '';
            }
        }

        window.removeFromQueue = (index) => {
            queue.splice(index, 1);
            renderQueue();
        };

        window.downloadImage = (url, filename) => {
            const link = document.createElement('a');
            link.href = url;
            link.download = `cleaned_${filename}`;
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        };

        // --- AI Processing ---

        async function removeWatermarkAI(index) {
            queue[index].status = 'processing';
            renderQueue();

            const item = queue[index];
            const base64Data = item.dataUrl.split(',')[1];
            
            try {
                // We use gemini-2.5-flash-image-preview for image-to-image processing
                // This model takes an image + text and returns the modified image
                const response = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{
                            parts: [
                                { text: "Precisely identify and remove all watermarks, including the subtle star patterns, grid lines, and text overlays. Reconstruct the background textures perfectly while keeping the original resolution and color quality. Do not change the aspect ratio or image size." },
                                { inlineData: { mimeType: item.file.type, data: base64Data } }
                            ]
                        }],
                        generationConfig: {
                            responseModalities: ['TEXT', 'IMAGE']
                        }
                    })
                });

                const result = await response.json();
                const imagePart = result.candidates?.[0]?.content?.parts?.find(p => p.inlineData);

                if (imagePart) {
                    item.resultUrl = `data:image/png;base64,${imagePart.inlineData.data}`;
                    item.status = 'completed';
                } else {
                    console.error("AI did not return an image", result);
                    item.status = 'error';
                }
            } catch (err) {
                console.error(err);
                item.status = 'error';
                showError("AI Service unavailable or took too long. Please try again.");
            }

            renderQueue();
        }

        async function fetchWithRetry(url, options, retries = 5) {
            let delay = 1000;
            for (let i = 0; i < retries; i++) {
                try {
                    const res = await fetch(url, options);
                    if (res.ok) return res;
                    if (res.status === 429 || res.status >= 500) {
                        await new Promise(r => setTimeout(r, delay));
                        delay *= 2;
                        continue;
                    }
                    throw new Error(`HTTP ${res.status}`);
                } catch (e) {
                    if (i === retries - 1) throw e;
                    await new Promise(r => setTimeout(r, delay));
                    delay *= 2;
                }
            }
        }

        function showError(msg) {
            statusMessage.innerText = msg;
            statusMessage.className = "p-4 rounded-xl text-sm font-medium bg-red-50 text-red-600";
            statusMessage.classList.remove('hidden');
            setTimeout(() => statusMessage.classList.add('hidden'), 5000);
        }
    </script>
</body>
</html>

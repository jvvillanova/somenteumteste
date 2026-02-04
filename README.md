<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Transcritor Grátis (Local)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
</head>
<body class="bg-slate-900 text-slate-100 min-h-screen font-sans">

    <div class="max-w-5xl mx-auto py-12 px-6">
        <header class="flex flex-col items-center mb-12">
            <div class="bg-indigo-500 p-4 rounded-3xl mb-4 shadow-2xl shadow-indigo-500/20">
                <i class="fas fa-brain text-3xl"></i>
            </div>
            <h1 class="text-4xl font-black tracking-tighter">AI Local Transcriber</h1>
            <p class="text-slate-400 mt-2">Processamento 100% gratuito e local.</p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <div class="lg:col-span-1 space-y-4">
                <div class="bg-slate-800 border border-slate-700 p-6 rounded-3xl">
                    <h2 class="font-bold text-sm uppercase tracking-widest text-indigo-400 mb-4">Status da IA</h2>
                    <div id="modelStatus" class="flex items-center gap-3 text-sm">
                        <span class="w-3 h-3 bg-yellow-500 rounded-full animate-pulse"></span>
                        Aguardando inicialização...
                    </div>
                    <div id="progressContainer" class="hidden mt-4">
                        <div class="text-[10px] uppercase mb-1">Baixando cérebro da IA (uma única vez):</div>
                        <div class="w-full bg-slate-700 h-2 rounded-full overflow-hidden">
                            <div id="progressBar" class="bg-indigo-500 h-full w-0 transition-all"></div>
                        </div>
                    </div>
                </div>

                <div class="bg-slate-800 border border-slate-700 p-6 rounded-3xl">
                    <input type="file" id="audioUpload" class="hidden" accept="audio/*">
                    <button onclick="document.getElementById('audioUpload').click()" class="w-full py-4 bg-indigo-600 hover:bg-indigo-500 rounded-2xl font-bold transition mb-3">
                        <i class="fas fa-file-import mr-2"></i> Carregar Áudio
                    </button>
                    <button id="recordBtn" class="w-full py-4 bg-slate-700 hover:bg-slate-600 rounded-2xl font-bold transition">
                        <i class="fas fa-microphone mr-2"></i> Gravar Agora
                    </button>
                </div>
            </div>

            <div class="lg:col-span-2">
                <div class="bg-slate-800 border border-slate-700 rounded-3xl min-h-[500px] flex flex-col overflow-hidden">
                    <div class="p-4 border-b border-slate-700 flex justify-between items-center bg-slate-800/50">
                        <span class="text-xs font-bold text-slate-500 uppercase">Transcrição em Tempo Real</span>
                        <button onclick="copyText()" class="text-indigo-400 text-sm hover:text-indigo-300 transition">Copiar Tudo</button>
                    </div>
                    <textarea id="output" class="flex-1 p-8 bg-transparent outline-none resize-none leading-relaxed text-slate-200" placeholder="O resultado aparecerá aqui após o processamento..."></textarea>
                </div>
            </div>
        </div>
    </div>

    <script type="module">
        import { pipeline } from 'https://cdn.jsdelivr.net/npm/@xenova/transformers@2.17.1';

        let transcriber;
        const modelStatus = document.getElementById('modelStatus');
        const progressBar = document.getElementById('progressBar');
        const progressContainer = document.getElementById('progressContainer');
        const output = document.getElementById('output');

        // Inicializa o modelo Whisper localmente
        async function init() {
            try {
                progressContainer.classList.remove('hidden');
                modelStatus.innerHTML = '<span class="w-3 h-3 bg-blue-500 rounded-full animate-pulse"></span> Carregando modelo...';
                
                transcriber = await pipeline('automatic-speech-recognition', 'Xenova/whisper-tiny', {
                    progress_callback: (p) => {
                        if (p.status === 'progress') {
                            progressBar.style.width = p.progress + '%';
                        }
                    }
                });

                modelStatus.innerHTML = '<span class="w-3 h-3 bg-green-500 rounded-full"></span> IA Pronta (Offline)';
                progressContainer.classList.add('hidden');
            } catch (e) {
                modelStatus.innerHTML = '<span class="w-3 h-3 bg-red-500 rounded-full"></span> Erro ao carregar IA.';
            }
        }

        init();

        // Processamento de Áudio
        async function processAudio(audioData) {
            if (!transcriber) return alert("IA ainda está carregando...");
            
            output.value = "Processando... Por favor, aguarde.";
            const start = performance.now();
            
            const result = await transcriber(audioData, { 
                language: 'portuguese',
                task: 'transcribe',
                chunk_length_s: 30,
                stride_length_s: 5
            });

            const end = performance.now();
            output.value = result.text;
            console.log(`Concluído em ${((end - start) / 1000).toFixed(2)}s`);
        }

        // Evento de Upload
        document.getElementById('audioUpload').onchange = async (e) => {
            const file = e.target.files[0];
            if (!file) return;

            const audioCtx = new (window.AudioContext || window.webkitAudioContext)({ sampleRate: 16000 });
            const arrayBuffer = await file.arrayBuffer();
            const decodedData = await audioCtx.decodeAudioData(arrayBuffer);
            const audioData = decodedData.getChannelData(0);
            
            processAudio(audioData);
        };

        window.copyText = () => {
            navigator.clipboard.writeText(output.value);
            alert("Copiado!");
        }
    </script>
</body>
</html>

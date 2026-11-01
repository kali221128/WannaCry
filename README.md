
## Player HTML Source
Save this code as `wcry-player.html`, open it with browser.
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Security Demo | Wcry Video Player</title>
    <style>
        :root {
            --bg-primary: #0c0c0e;
            --bg-card: #16161a;
            --border: #2a2a30;
            --text-main: #e6e6e8;
            --text-sub: #94949b;
            --accent-red: #d63031;
            --accent-glow: rgba(214, 48, 49, 0.25);
            --control-bg: rgba(12,12,14,0.72);
            --transition: 0.22s ease;
        }
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: "Segoe UI", Roboto, Consolas, monospace, system-ui;
        }
        body {
            background: var(--bg-primary);
            color: var(--text-main);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 24px;
        }
        .page-wrap {
            width: 100%;
            max-width: 1080px;
        }
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 16px;
        }
        .header-title {
            font-size: 18px;
            font-weight: 500;
            letter-spacing: 0.5px;
        }
        .header-warning {
            color: var(--accent-red);
            font-size: 13px;
        }
        .video-shell {
            position: relative;
            background: #000;
            border: 1px solid var(--border);
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 10px 40px rgba(0,0,0,0.45), 0 0 0 1px rgba(214,48,49,0.08);
        }
        video {
            width: 100%;
            display: block;
        }
        .overlay-text {
            position: absolute;
            inset: 0;
            pointer-events: none;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        .overlay-text > div {
            color: #ff3838;
            font-family: Consolas, monospace;
            font-size: clamp(16px, 2.2vw, 26px);
            text-align: center;
            line-height: 1.7;
            text-shadow: 0 0 12px var(--accent-glow);
            white-space: pre-line;
        }
        .controls {
            position: absolute;
            left: 0;
            right: 0;
            bottom: 0;
            padding: 14px 18px;
            background: linear-gradient(transparent, var(--control-bg));
            display: flex;
            gap:14px;
            align-items: center;
            transition: var(--transition);
        }
        .video-shell:not(:hover):not(.paused) .controls {
            opacity: 0;
        }
        .ctrl-btn {
            background: transparent;
            border: none;
            color: var(--text-main);
            cursor: pointer;
            padding: 4px 8px;
            font-size:15px;
            transition: var(--transition);
        }
        .ctrl-btn:hover {
            color: var(--accent-red);
        }
        .progress-wrap {
            flex:1;
        }
        #progress {
            width: 100%;
            accent-color: var(--accent-red);
        }
        .time-label {
            font-size: 12px;
            color: var(--text-sub);
            min-width: 80px;
            text-align: right;
        }
        .meta-footer {
            margin-top: 18px;
            padding: 14px 18px;
            background: var(--bg-card);
            border:1px solid var(--border);
            border-radius: 8px;
            color: var(--text-sub);
            font-size:13px;
            line-height:1.6;
        }
        .loading, .error-tip {
            position: absolute;
            inset:0;
            background: rgba(0,0,0,0.75);
            display: grid;
            place-items:center;
            color:#fff;
            text-align:center;
        }
        .error-tip {
            color:var(--accent-red);
        }
        .spinner {
            width:40px;height:40px;
            border:3px solid #222;
            border-top-color: var(--accent-red);
            border-radius:50%;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { to { transform: rotate(360deg); } }
        .hidden { display: none; }
    </style>
</head>
<body>
    <div class="page-wrap">
        <div class="header">
            <div class="header-title">Malware Demo Player | Wcry</div>
            <div class="header-warning">FOR SECURITY RESEARCH ONLY</div>
        </div>
        <div class="video-shell" id="videoShell">
            <div class="loading" id="loading">
                <div class="spinner"></div>
                <div style="margin-top:12px; font-size:14px;">Loading video...</div>
            </div>
            <div class="error-tip hidden" id="errorTip">
                Video load failed<br/>Raw Github source may be blocked by network
            </div>

            <video id="vid" muted>
                <source src="https://raw.githubusercontent.com/kali221128/WannaCry/main/wcry.mp4" type="video/mp4">
                Browser does not support MP4 playback
            </video>

            <div class="overlay-text">
                <div>Your file has been encrypted.Please contact us soon as possible to decrypt it.
if delayed for too long,the file will be permanently damaged.</div>
            </div>

            <div class="controls">
                <button class="ctrl-btn" id="playBtn">▶ Play</button>
                <div class="progress-wrap">
                    <input type="range" id="progress" min="0" max="100" value="0">
                </div>
                <span class="time-label" id="timeText">00:00 / 00:00</span>
                <button class="ctrl-btn" id="fullBtn">⛶ Fullscreen</button>
                <button class="ctrl-btn" id="loopBtn">⟳ Loop:OFF</button>
            </div>
        </div>

        <div class="meta-footer">
            <strong>⚠️ Legal Notice：</strong> This page is only used for information security teaching and threat intelligence research.
            Dissemination, deployment and use of ransomware are illegal and criminal acts.
            The raw video resource is hosted on GitHub, and access may fail in mainland network environment.
        </div>
    </div>

    <script>
        const vid = document.getElementById('vid');
        const shell = document.getElementById('videoShell');
        const playBtn = document.getElementById('playBtn');
        const progress = document.getElementById('progress');
        const timeText = document.getElementById('timeText');
        const fullBtn = document.getElementById('fullBtn');
        const loopBtn = document.getElementById('loopBtn');
        const loading = document.getElementById('loading');
        const errorTip = document.getElementById('errorTip');

        let loopMode = false;

        function fmtTime(s) {
            if(isNaN(s)) return "00:00";
            const m = Math.floor(s / 60).toString().padStart(2,'0');
            const sec = Math.floor(s % 60).toString().padStart(2,'0');
            return `${m}:${sec}`;
        }

        vid.addEventListener('loadeddata', ()=>{
            loading.classList.add('hidden');
        })
        vid.addEventListener('error', ()=>{
            loading.classList.add('hidden');
            errorTip.classList.remove('hidden');
        })

        playBtn.onclick = ()=>{
            if(vid.paused) {
                vid.play();
                playBtn.textContent = '⏸ Pause';
                shell.classList.remove('paused');
            }else{
                vid.pause();
                playBtn.textContent = '▶ Play';
                shell.classList.add('paused');
            }
        }
        shell.classList.add('paused');

        vid.addEventListener('timeupdate', ()=>{
            const pct = (vid.currentTime / vid.duration) * 100;
            progress.value = pct || 0;
            timeText.textContent = `${fmtTime(vid.currentTime)} / ${fmtTime(vid.duration)}`;
        })
        progress.oninput = ()=>{
            vid.currentTime = (progress.value / 100) * vid.duration;
        }

        fullBtn.onclick = ()=>{
            if(document.fullscreenElement) document.exitFullscreen();
            else shell.requestFullscreen();
        }

        loopBtn.onclick = ()=>{
            loopMode = !loopMode;
            vid.loop = loopMode;
            loopBtn.textContent = loopMode ? '⟳ Loop:ON' : '⟳ Loop:OFF';
        }
    </script>
</body>
</html>


# # WannaCry-Experiment
This resource mainly reproduces the process of the WannaCry ransomware, including the corresponding resources, articles, and the ransomware itself. Hope it is helpful to you.
I firmly oppose any criminal acts carried out using teaching methods. All criminal acts will be severely punished. We need to work together to maintain a green internet. I also recommend everyone learn about the principles behind them so as to better protect against them.

<div align="center">
  <img src="https://img-blog.csdnimg.cn/20200413232834492.png" height="60%" width="60%" />
</div>

# The execution flow of WannaCry
<div align="center">
  <img src="https://img-blog.csdnimg.cn/20200414014519105.png" height="60%" width="60%" />
</div>

# Download and Usage Agreement:
1. This is a WannaCry simulator. Based on the original version, its propagation and encryption capabilities have been removed.
2. It will not actually encrypt files; it only changes file extensions. The file extensions can be restored using the included file recovery tool.
3. It will release all processed original components, but they will not be easily activated.
4. This WannaCry simulator may be used freely by anyone (except malicious actors). However, the author assumes no responsibility for any harmful consequences.
5. This WannaCry simulator is open source. 
6. For security drills and research only.
7. This WannaCry simulator has significant value for security research. It is suitable for people who do not dare to run the real virus to study, and can also be used for security drill research.
By downloading this project (WannaCry), you voluntarily agree to all the above terms!

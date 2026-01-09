<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PIXEL BALATRO CLONE</title>
    <link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
    <style>
        :root { 
            --bg: #0b141a; --card: #eff2f1; --accent: #4ade80; --danger: #f87171; 
            --joker: #a855f7; --gold: #fbbf24; --voucher: #22d3ee; --planet: #f97316;
            --border: #334155;
        }
        
        body { 
            background: var(--bg); color: white; font-family: 'VT323', monospace; 
            display: flex; flex-direction: column; align-items: center; margin: 0;
            image-rendering: pixelated; letter-spacing: 1px; overflow: hidden;
        }

        #game-container { width: 900px; padding: 20px; text-transform: uppercase; transition: transform 0.1s; }

        /* Start Screen */
        #start-screen {
            position: fixed; inset: 0; background: var(--bg);
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            z-index: 100; text-align: center;
        }
        .title-text { font-size: 8rem; color: var(--danger); text-shadow: 8px 8px 0px #000; margin: 0; }
        .subtitle { font-size: 2rem; color: var(--gold); margin-bottom: 40px; }
        .flicker { animation: flicker 1.5s infinite; cursor: pointer; }
        @keyframes flicker { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }

        /* UI Elements */
        .stats-bar { 
            display: flex; justify-content: space-around; background: #1e293b; 
            padding: 15px; border: 4px solid var(--border); border-radius: 4px;
            margin-bottom: 20px; font-size: 1.8rem; box-shadow: 6px 6px 0px #000;
        }
        .stats-bar b { color: var(--gold); }

        .area { 
            background: rgba(15, 23, 42, 0.9); border: 4px solid var(--border);
            padding: 20px; margin-bottom: 20px; position: relative;
            box-shadow: 6px 6px 0px #000;
        }
        .area-label { 
            position: absolute; top: -14px; left: 20px; background: #1e293b; 
            padding: 2px 10px; font-size: 1.2rem; border: 2px solid var(--border);
        }

        .grid { display: flex; gap: 12px; justify-content: center; flex-wrap: wrap; min-height: 120px; }

        /* Pixel Cards */
        .card { 
            width: 75px; height: 110px; background: var(--card); color: #1e293b; 
            border-radius: 4px; display: flex; flex-direction: column; align-items: center; 
            justify-content: center; cursor: pointer; border: 4px solid #94a3b8; 
            transition: 0.1s; position: relative; font-size: 2rem;
            box-shadow: 4px 4px 0px rgba(0,0,0,0.4);
        }
        .card.selected { border-color: var(--gold); transform: translateY(-15px); box-shadow: 0 0 20px var(--gold); }
        .card.red { color: #b91c1c; }
        
        .joker { 
            width: 90px; height: 125px; background: #312e81; color: white; 
            border-radius: 4px; padding: 8px; font-size: 1.1rem; text-align: center; 
            border: 4px solid var(--joker); animation: float 3s ease-in-out infinite;
            box-shadow: 4px 4px 0px #000;
        }
        @keyframes float { 0%, 100% { transform: translateY(0px); } 50% { transform: translateY(-8px); } }

        .btn { 
            padding: 12px 24px; border: 4px solid #000; cursor: pointer; 
            font-family: 'VT323', monospace; font-size: 1.8rem; color: white;
            box-shadow: 4px 4px 0px #000; margin: 0 10px;
        }
        .btn-play { background: var(--accent); color: #064e3b; }
        .btn-discard { background: var(--danger); color: #7f1d1d; }

        @keyframes shake { 0%, 100% { transform: translate(0, 0); } 25% { transform: translate(-5px, 3px); } 75% { transform: translate(5px, -3px); } }
        .shake { animation: shake 0.15s ease-in-out 2; }
        .hidden { display: none !important; }
        #music-container { position: absolute; left: -9999px; }
    </style>
</head>
<body>

<div id="start-screen">
    <h1 class="title-text">BALATRO</h1>
    <div class="subtitle">WEB PIXEL EDITION</div>
    <div class="flicker subtitle" onclick="startGame()">[ PRESS START ]</div>
</div>

<div id="music-container"><div id="player"></div></div>

<div id="game-container" class="hidden">
    <div class="stats-bar">
        <div>ANTE: <b id="ui-ante">1</b>/8</div>
        <div>GOAL: <b id="target">300</b></div>
        <div>SCORE: <b id="score">0</b></div>
        <div>$: <b id="money">10</b></div>
        <div>HANDS: <b id="hands">4</b></div>
    </div>

    <div class="area"><div class="area-label">JOKERS</div><div id="joker-display" class="grid"></div></div>

    <div id="play-screen">
        <div class="area">
            <div class="area-label" id="round-label">SMALL BLIND</div>
            <div id="hand-display" class="grid"></div>
        </div>
        <div style="text-align: center;">
            <button class="btn btn-play" onclick="playHand()">PLAY HAND</button>
            <button class="btn btn-discard" onclick="discardSelected()">DISCARD (<span id="discards">3</span>)</button>
        </div>
    </div>

    <div id="shop-screen" class="area hidden">
        <div class="area-label">THE SHOP</div>
        <h1 id="win-msg" style="text-align: center; color: var(--gold);">ROUND WON!</h1>
        <div id="shop-items" class="grid" style="margin-bottom: 20px;"></div>
        <div style="text-align: center;"><button class="btn btn-play" onclick="startRound()">NEXT ROUND</button></div>
    </div>
</div>

<script src="https://www.youtube.com/iframe_api"></script>

<script>
    let player;
    function onYouTubeIframeAPIReady() {
        player = new YT.Player('player', {
            height: '0', width: '0', videoId: '7GutRIhmTwQ',
            playerVars: { 'autoplay': 0, 'loop': 1, 'playlist': '7GutRIhmTwQ', 'controls': 0 }
        });
    }

    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playSound(freq, type, duration, vol = 0.1) {
        if (audioCtx.state === 'suspended') audioCtx.resume();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = type;
        osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
        gain.gain.setValueAtTime(vol, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + duration);
        osc.connect(gain); gain.connect(audioCtx.destination);
        osc.start(); osc.stop(audioCtx.currentTime + duration);
    }

    const sfx = {
        select: () => playSound(500, 'sine', 0.05),
        play: () => { playSound(150, 'square', 0.2); setTimeout(() => playSound(400, 'square', 0.1), 50); },
        buy: () => { playSound(600, 'triangle', 0.1); setTimeout(() => playSound(800, 'triangle', 0.1), 80); }
    };

    const RANKS = ['2', '3', '4', '5', '6', '7', '8', '9', 'T', 'J', 'Q', 'K', 'A'];
    const SUITS = ['♠', '♥', '♦', '♣'];
    let state = { score: 0, target: 300, money: 10, hands: 4, discards: 3, handSize: 8, deck: [], hand: [], selected: [], jokers: [], ante: 1, round: 1 };
    
    let handLevels = { "High Card": [5, 1], "Pair": [10, 2], "Two Pair": [20, 2], "Three of a Kind": [30, 3], "Straight": [30, 4], "Flush": [35, 4], "Full House": [40, 4], "Four of a Kind": [60, 7] };

    function startGame() {
        document.getElementById('start-screen').classList.add('hidden');
        document.getElementById('game-container').classList.remove('hidden');
        if (player && player.playVideo) player.playVideo();
        startRound();
    }

    function initDeck() {
        state.deck = [];
        SUITS.forEach(s => RANKS.forEach(r => state.deck.push({ rank: r, suit: s, val: RANKS.indexOf(r), isRed: (s==='♥'||s==='♦') })));
        state.deck.sort(() => Math.random() - 0.5);
    }

    function startRound() {
        state.score = 0; 
        state.target = Math.floor(300 * Math.pow(2, state.ante - 1));
        state.hands = 4; state.discards = 3;
        initDeck(); state.hand = []; drawToFull();
        document.getElementById('play-screen').classList.remove('hidden');
        document.getElementById('shop-screen').classList.add('hidden');
        render();
    }

    function evaluateHand(cards) {
        cards.sort((a,b) => a.val - b.val);
        const counts = {}; cards.forEach(c => counts[c.rank] = (counts[c.rank] || 0) + 1);
        const vals = Object.values(counts).sort((a,b) => b-a);
        const flush = cards.length === 5 && cards.every(c => c.suit === cards[0].suit);
        const straight = cards.length === 5 && cards.every((c, i) => i === 0 || c.val === cards[i-1].val + 1);
        
        if (flush && straight) return "Straight Flush";
        if (vals[0] === 4) return "Four of a Kind";
        if (vals[0] === 3 && vals[1] === 2) return "Full House";
        if (flush) return "Flush";
        if (straight) return "Straight";
        if (vals[0] === 3) return "Three of a Kind";
        if (vals[0] === 2 && vals[1] === 2) return "Two Pair";
        if (vals[0] === 2) return "Pair";
        return "High Card";
    }

    function playHand() {
        if (state.selected.length === 0) return;
        sfx.play();
        document.getElementById('game-container').classList.add('shake');
        setTimeout(() => document.getElementById('game-container').classList.remove('shake'), 200);

        const played = state.selected.map(i => state.hand[i]);
        const handName = evaluateHand(played);
        const base = handLevels[handName] || [5, 1];
        
        state.score += (base[0] * base[1]);
        state.hands--;
        state.hand = state.hand.filter((_, i) => !state.selected.includes(i));
        state.selected = [];
        
        if (state.score >= state.target) showShop();
        else if (state.hands <= 0) { alert("GAME OVER"); location.reload(); }
        drawToFull();
    }

    function showShop() {
        sfx.buy();
        document.getElementById('play-screen').classList.add('hidden');
        document.getElementById('shop-screen').classList.remove('hidden');
        state.round++; if(state.round > 3) { state.round = 1; state.ante++; }
    }

    function drawToFull() {
        while(state.hand.length < state.handSize && state.deck.length > 0) state.hand.push(state.deck.pop());
        render();
    }

    function discardSelected() {
        if (state.selected.length === 0 || state.discards <= 0) return;
        state.discards--;
        state.hand = state.hand.filter((_, i) => !state.selected.includes(i));
        state.selected = [];
        drawToFull();
    }

    function render() {
        document.getElementById('score').innerText = state.score;
        document.getElementById('target').innerText = state.target;
        document.getElementById('ui-ante').innerText = state.ante;
        document.getElementById('hands').innerText = state.hands;
        document.getElementById('discards').innerText = state.discards;
        document.getElementById('money').innerText = state.money;

        const handDiv = document.getElementById('hand-display');
        handDiv.innerHTML = '';
        state.hand.forEach((c, i) => {
            const d = document.createElement('div');
            d.className = `card ${c.isRed?'red':''} ${state.selected.includes(i)?'selected':''}`;
            d.innerHTML = `<div>${c.rank}</div><div>${c.suit}</div>`;
            d.onclick = () => {
                if(state.selected.includes(i)) state.selected = state.selected.filter(x => x!==i);
                else if(state.selected.length < 5) { state.selected.push(i); sfx.select(); }
                render();
            };
            handDiv.appendChild(d);
        });
    }
</script>
</body>
</html>

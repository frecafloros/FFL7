---
layout: page
title: "Root Universe"
parent: "附録"
nav_order: 1
---

# Root Universe (語根占有状況)

プリャンナの語根 (Slot 2: C1-V-C2) の占有状況を視覚化したものです。
スライダーを動かして、現在データが存在する第1子音 (C1) の組み合わせを確認できます。

<div id="controls" style="margin-bottom: 20px; padding: 15px; background: #2d333b; border-radius: 6px;">
    <label for="c1-slider" style="display: block; margin-bottom: 10px;">
        第1子音 (C1) 選択: <span id="c1-label" style="font-weight: bold; color: #00ffcc;">読み込み中...</span>
    </label>
    <input type="range" id="c1-slider" min="0" max="0" value="0" style="width: 100%; cursor: pointer;">
    <small style="color: #8b949e; display: block; margin-top: 5px;">※データが存在する子音のみを表示しています（空のプレーンはスキップされます）</small>
</div>

<div id="heatmap-container" style="width: 100%; height: 600px; background: #1c2128; border-radius: 6px; position: relative;">
    <div id="heatmap" style="width: 100%; height: 100%;"></div>
    <div id="loading-overlay" style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(28, 33, 40, 0.9); display: flex; align-items: center; justify-content: center; z-index: 10; color: #adbac7; font-family: sans-serif;">
        <div style="text-align: center;">
            <div id="status-message">データを読み込み中...</div>
            <div id="error-details" style="color: #ff6b6b; margin-top: 10px; font-size: 0.9em; display: none;"></div>
        </div>
    </div>
</div>

<script src="https://cdn.plot.ly/plotly-2.24.1.min.js"></script>
<script>
document.addEventListener('DOMContentLoaded', function() {
    const slider = document.getElementById('c1-slider');
    const label = document.getElementById('c1-label');
    const heatmapDiv = document.getElementById('heatmap');
    const loadingOverlay = document.getElementById('loading-overlay');
    const statusMsg = document.getElementById('status-message');
    const errorDetails = document.getElementById('error-details');

    const MAX_C = 234;
    const MAX_V = 46;

    let occupancyData = {
        phonemes: { C: {}, V: {} },
        roots: [],
        available_c1: []
    };

    // データの読み込み (GitHub Pages のパス解決を考慮)
    const jsonUrl = '{{ "/assets/data/root_occupancy.json" | relative_url }}';
    
    fetch(jsonUrl)
        .then(response => {
            if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
            return response.json();
        })
        .then(data => {
            console.log("Data loaded successfully", data);
            occupancyData = data;
            
            if (!data.available_c1 || data.available_c1.length === 0) {
                statusMsg.textContent = "表示可能なデータがありません。";
                return;
            }

            // スライダーの設定
            slider.max = data.available_c1.length - 1;
            slider.value = 0;
            
            loadingOverlay.style.display = 'none';
            updateHeatmap(0); // 最初の有効なインデックスを表示
        })
        .catch(err => {
            console.error('Failed to load data:', err);
            statusMsg.textContent = "データの読み込みに失敗しました。";
            errorDetails.textContent = err.message;
            errorDetails.style.display = 'block';
        });

    slider.addEventListener('input', function() {
        updateHeatmap(parseInt(this.value));
    });

    function updateHeatmap(sliderVal) {
        const c1Idx = occupancyData.available_c1[sliderVal];
        const c1Char = occupancyData.phonemes.C[c1Idx] || `C${String(c1Idx).padStart(3, '0')}`;
        label.textContent = `${c1Char} (C${String(c1Idx).padStart(3, '0')}) [${sliderVal + 1}/${occupancyData.available_c1.length}]`;

        // ヒートマップ用行列の初期化 (V x C2)
        const matrix = Array.from({ length: MAX_V }, () => Array(MAX_C).fill(0));
        const hoverText = Array.from({ length: MAX_V }, () => Array(MAX_C).fill('Empty'));

        // 該当するC1のデータを抽出
        occupancyData.roots.forEach(root => {
            const [rC1, rV, rC2, phonetic, gloss] = root;
            if (rC1 === c1Idx) {
                matrix[rV][rC2] = 1;
                hoverText[rV][rC2] = `${phonetic}<br>Gloss: ${gloss || '-'}`;
            }
        });

        const xLabels = Array.from({ length: MAX_C }, (_, i) => occupancyData.phonemes.C[i] || `C${String(i).padStart(3, '0')}`);
        const yLabels = Array.from({ length: MAX_V }, (_, i) => occupancyData.phonemes.V[i] || `V${String(i).padStart(2, '0')}`);

        const data = [{
            z: matrix,
            x: xLabels,
            y: yLabels,
            type: 'heatmap',
            colorscale: [
                [0, '#1e1e1e'], // 空き
                [1, '#00ffcc']  // 占有
            ],
            showscale: false,
            text: hoverText,
            hovertemplate: '<b>%{text}</b><br>V: %{y}<br>C2: %{x}<extra></extra>',
        }];

        const layout = {
            paper_bgcolor: 'rgba(0,0,0,0)',
            plot_bgcolor: 'rgba(0,0,0,0)',
            xaxis: {
                title: '第2子音 (C2)',
                tickmode: 'auto',
                nticks: 10,
                gridcolor: '#2d333b',
                color: '#adbac7'
            },
            yaxis: {
                title: '母音 (V)',
                tickmode: 'auto',
                nticks: 10,
                gridcolor: '#2d333b',
                color: '#adbac7'
            },
            margin: { l: 50, r: 20, b: 50, t: 20 },
            font: { color: '#adbac7' }
        };

        const config = {
            responsive: true,
            displayModeBar: false
        };

        Plotly.newPlot(heatmapDiv, data, layout, config);
    }
});
</script>

<style>
/* スライダーのスタイル調整 */
#c1-slider {
    -webkit-appearance: none;
    height: 8px;
    background: #444c56;
    outline: none;
    border-radius: 4px;
}
#c1-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 20px;
    height: 20px;
    background: #00ffcc;
    cursor: pointer;
    border-radius: 50%;
}
</style>

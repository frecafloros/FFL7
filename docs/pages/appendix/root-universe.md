---
layout: page
title: "Root Universe"
parent: "附録"
nav_order: 1
---

# Root Universe (語根占有状況)

プリャンナの語根 (Slot 2: C1-V-C2) の占有状況を視覚化したものです。
スライダーを動かして、第1子音 (C1) ごとの母音 (V) と第2子音 (C2) の組み合わせを確認できます。

<div id="controls" style="margin-bottom: 20px; padding: 15px; background: #2d333b; border-radius: 6px;">
    <label for="c1-slider" style="display: block; margin-bottom: 10px;">
        第1子音 (C1): <span id="c1-label" style="font-weight: bold; color: #00ffcc;">C000 (???)</span>
    </label>
    <input type="range" id="c1-slider" min="0" max="233" value="0" style="width: 100%; cursor: pointer;">
</div>

<div id="heatmap-container" style="width: 100%; height: 600px; background: #1c2128; border-radius: 6px; position: relative;">
    <div id="heatmap" style="width: 100%; height: 100%;"></div>
    <div id="loading-overlay" style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(28, 33, 40, 0.8); display: flex; align-items: center; justify-content: center; z-index: 10;">
        <span>データ読み込み中...</span>
    </div>
</div>

<script src="https://cdn.plot.ly/plotly-2.24.1.min.js"></script>
<script>
document.addEventListener('DOMContentLoaded', function() {
    const slider = document.getElementById('c1-slider');
    const label = document.getElementById('c1-label');
    const heatmapDiv = document.getElementById('heatmap');
    const loadingOverlay = document.getElementById('loading-overlay');

    const MAX_C = 234;
    const MAX_V = 46;

    let occupancyData = {
        phonemes: { C: {}, V: {} },
        roots: []
    };

    // データの読み込み
    fetch('{{ site.baseurl }}/assets/data/root_occupancy.json')
        .then(response => response.json())
        .then(data => {
            occupancyData = data;
            loadingOverlay.style.display = 'none';
            updateHeatmap(0);
        })
        .catch(err => {
            console.error('Failed to load data:', err);
            loadingOverlay.innerHTML = '<span style="color: #ff6b6b;">データの読み込みに失敗しました</span>';
        });

    slider.addEventListener('input', function() {
        updateHeatmap(parseInt(this.value));
    });

    function updateHeatmap(c1Idx) {
        const c1Char = occupancyData.phonemes.C[c1Idx] || `C${String(c1Idx).padStart(3, '0')}`;
        label.textContent = `${c1Char} (C${String(c1Idx).padStart(3, '0')})`;

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
                nticks: 20,
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

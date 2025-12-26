---
title: "OGP画像作るくん"
---

<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.15.3/dist/cdn.min.js"></script>

<div x-data="ogpGenerator()" x-init="generate()">
  <div>
    <label for="filename">ファイル名</label><br>
    <input type="text" id="filename" x-model="filename">
  </div>
  <div>
    <label for="title">記事タイトル</label><br>
    <textarea id="title" x-model="title" @input="generate()" rows="3"></textarea>
  </div>
  <div>
    <button @click="download()">ダウンロード</button>
  </div>  
  <div>
    <canvas x-ref="canvas" width="1200" height="630" style="width:100%;height:auto;"></canvas>
  </div>
</div>

<script>
function ogpGenerator() {
  return {
    filename: 'ogp-image',
    title: '',
    theme: { bg1: '#fafafa', bg2: '#f0f0f0', accent: '#333333', text: '#1a1a1a', subtext: '#666666' },

    get canvas() { return this.$refs.canvas },
    get ctx() { return this.canvas.getContext('2d') },

    generate() {
      const W = 1200, H = 630, t = this.theme, ctx = this.ctx;

      // Background gradient
      const grad = ctx.createLinearGradient(0, 0, W, H);
      grad.addColorStop(0, t.bg1);
      grad.addColorStop(1, t.bg2);
      ctx.fillStyle = grad;
      ctx.fillRect(0, 0, W, H);

      // Decorative circles
      ctx.globalAlpha = 0.1;
      ctx.fillStyle = t.accent;
      ctx.beginPath();
      ctx.arc(W * 0.85, H * 0.2, 200, 0, Math.PI * 2);
      ctx.fill();
      ctx.beginPath();
      ctx.arc(W * 0.1, H * 0.8, 150, 0, Math.PI * 2);
      ctx.fill();

      // Grid pattern
      ctx.globalAlpha = 0.03;
      ctx.strokeStyle = t.text;
      ctx.lineWidth = 1;
      for (var i = 0; i < W; i += 60) {
        ctx.beginPath(); ctx.moveTo(i, 0); ctx.lineTo(i, H); ctx.stroke();
      }
      for (var i = 0; i < H; i += 60) {
        ctx.beginPath(); ctx.moveTo(0, i); ctx.lineTo(W, i); ctx.stroke();
      }
      ctx.globalAlpha = 1;

      // Accent bar
      var bar = ctx.createLinearGradient(0, 0, W, 0);
      bar.addColorStop(0, t.accent);
      bar.addColorStop(0.5, t.accent);
      bar.addColorStop(1, 'transparent');
      ctx.fillStyle = bar;
      ctx.fillRect(0, 0, W, 6);

      ctx.fillStyle = t.accent;
      ctx.globalAlpha = 0.3;
      ctx.fillRect(60, H - 100, W - 120, 2);
      ctx.globalAlpha = 1;

      // Title
      var text = this.title || 'タイトルを入力';
      var fontSize = 64, lines;
      do {
        ctx.font = 'bold ' + fontSize + 'px sans-serif';
        lines = this.wrapText(text, W - 160, fontSize);
        if (lines.length <= 3) break;
        fontSize -= 4;
      } while (fontSize > 32);
      if (lines.length > 3) { lines = lines.slice(0, 3); lines[2] = lines[2].slice(0, -2) + '...'; }

      var lh = fontSize * 1.4;
      var startY = (H - lines.length * lh) / 2 + fontSize / 2;
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';

      lines.forEach(function(line, i) {
        var y = startY + i * lh;
        ctx.fillStyle = 'rgba(0, 0, 0, 0.1)';
        ctx.fillText(line, W / 2 + 2, y + 2);
        ctx.fillStyle = t.text;
        ctx.fillText(line, W / 2, y);
      });

      // Blog name
      ctx.font = '600 24px sans-serif';
      ctx.fillStyle = t.subtext;
      ctx.textAlign = 'left';
      ctx.fillText('よ〜んの雑記', 80, H - 50);

      ctx.fillStyle = t.accent;
      ctx.beginPath();
      ctx.arc(60, H - 50, 6, 0, Math.PI * 2);
      ctx.fill();
    },

    wrapText(text, maxW, size) {
      this.ctx.font = 'bold ' + size + 'px sans-serif';
      var lines = [];
      var paragraphs = text.split('\n');
      for (var p = 0; p < paragraphs.length; p++) {
        var para = paragraphs[p];
        if (!para) { lines.push(''); continue; }
        var cur = '';
        for (var i = 0; i < para.length; i++) {
          var c = para[i];
          if (this.ctx.measureText(cur + c).width > maxW && cur) { lines.push(cur); cur = c; }
          else cur += c;
        }
        if (cur) lines.push(cur);
      }
      return lines;
    },

    download() {
      var a = document.createElement('a');
      a.download = this.filename + '.png';
      a.href = this.canvas.toDataURL('image/png');
      a.click();
    }
  }
}
</script>

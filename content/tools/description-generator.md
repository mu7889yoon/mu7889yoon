---
title: "Description生成くん"
---

<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.15.3/dist/cdn.min.js"></script>

<div x-data="descriptionGenerator()" x-init="init()">
  <div>
    <label for="body">記事本文</label><br>
    <textarea id="body" x-model="body" @input="clearResult()" rows="10" placeholder="本文を貼り付け"></textarea>
  </div>

  <div>
    <button @click="generate()" :disabled="loading || !body.trim() || !available">生成</button>
    <button @click="copy()" :disabled="!result">コピー</button>
  </div>
  <div x-show="availabilityLabel" x-text="availabilityLabel"></div>
  <div x-show="status" x-text="status"></div>
  <div x-show="error" x-text="error" style="color:#b00020;"></div>
  <div>
    <label for="result">Description</label><br>
    <textarea id="result" x-model="result" rows="4" readonly></textarea>
  </div>
</div>

<script>
function descriptionGenerator() {
  return {
    title: '',
    body: '',
    type: 'teaser',
    length: 'short',
    maxChars: 100,
    result: '',
    status: '',
    error: '',
    available: false,
    availability: 'no',
    availabilityLabel: '',
    loading: false,

    init() {
      this.checkAvailability();
    },

    async checkAvailability() {
      if (!('Summarizer' in self)) {
        this.available = false;
        this.availability = 'no';
        this.status = '';
        this.error = 'ChromeのSummarizer APIが使えません。Chrome 138以降で実行してください。';
        return;
      }
      try {
        var availability = await Summarizer.availability();
        this.availability = availability || 'unavailable';
        this.available = this.availability !== 'unavailable';
        if (this.availability === 'downloadable') {
          this.availabilityLabel = 'モデルのダウンロードが必要です。生成時にダウンロードが始まります。';
        } else if (this.availability === 'available') {
          this.availabilityLabel = 'Summarizer APIが利用可能です。';
        } else {
          this.availabilityLabel = '';
          this.error = 'この環境ではSummarizer APIが利用できません。';
        }
      } catch (err) {
        this.available = false;
        this.availability = 'no';
        this.availabilityLabel = '';
        this.error = 'Summarizer APIの確認に失敗しました: ' + (err && err.message ? err.message : String(err));
      }
    },

    clearResult() {
      this.result = '';
      this.status = '';
      this.error = '';
    },

    buildSource() {
      var parts = [];
      if (this.title.trim()) parts.push(this.title.trim());
      if (this.body.trim()) parts.push(this.body.trim());
      return parts.join('\n\n');
    },

    trimToMax(text) {
      if (text.length <= 100) return text;
      return text.slice(0, 97).trim() + '...';
    },

    async generate() {
      this.error = '';
      this.status = '';
      if (!this.available) {
        this.error = 'Summarizer APIが利用できません。';
        return;
      }
      if (!this.body.trim()) {
        this.error = '本文を入力してください。';
        return;
      }
      this.loading = true;
      this.status = '生成中...';
      var summarizer = null;
      var self = this;
      try {
        summarizer = await Summarizer.create({
          type: this.type,
          format: 'plain-text',
          length: this.length,
          outputLanguage: 'ja',
          monitor: function(m) {
            m.addEventListener('downloadprogress', function(e) {
              self.status = 'モデルダウンロード中: ' + Math.round(e.loaded * 100) + '%';
            });
          }
        });
        this.status = '要約生成中...';
        var summary = '';
        var stream = summarizer.summarizeStreaming(this.buildSource());
        for await (var chunk of stream) {
          summary += chunk;
          this.result = this.trimToMax(summary);
        }
        this.result = this.trimToMax(summary);
        this.status = '生成しました。';
      } catch (err) {
        this.error = '生成に失敗しました: ' + (err && err.message ? err.message : String(err));
      } finally {
        if (summarizer && summarizer.destroy) summarizer.destroy();
        this.loading = false;
      }
    },

    async copy() {
      if (!this.result) return;
      try {
        await navigator.clipboard.writeText(this.result);
        this.status = 'コピーしました。';
      } catch (err) {
        this.error = 'コピーに失敗しました。';
      }
    }
  }
}
</script>

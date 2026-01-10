---
title: "サクッとPython REPLくん"
---

<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.15.3/dist/cdn.min.js"></script>
<script defer src="https://cdn.jsdelivr.net/npm/pyodide@0.29.1/pyodide.min.js"></script>

<div x-data="pythonRepl()" x-init="init()">
  <div>
    <label for="code">コード</label><br>
    <textarea id="code" x-model="input" rows="8" placeholder="Pythonコードを入力" :disabled="!ready"></textarea>
  </div>
  <div>
    <button @click="run()" :disabled="loading || !ready || !input.trim()">実行</button>
    <button @click="clearOutput()" :disabled="!output">出力クリア</button>
  </div>
  <div x-show="status" x-text="status"></div>
  <div x-show="error" x-text="error" style="color:#b00020;"></div>
  <div>
    <label for="output">出力</label><br>
    <textarea id="output" x-model="output" rows="8" readonly></textarea>
  </div>
</div>

<script>
function pythonRepl() {
  return {
    pyodide: null,
    ready: false,
    loading: false,
    initialized: false,
    input: '',
    output: '',
    status: '',
    error: '',

    async init() {
      if (this.initialized) return;
      this.initialized = true;
      this.loading = true;
      this.status = 'Pyodide読み込み中...';
      this.error = '';
      try {
        await this.waitForPyodide();
        this.pyodide = await loadPyodide();
        this.pyodide.setStdout({ batched: (s) => this.output += s + '\n' });
        this.pyodide.setStderr({ batched: (s) => this.output += s + '\n' });
        this.ready = true;
        this.status = '';
      } catch (err) {
        this.error = '初期化に失敗しました: ' + (err && err.message ? err.message : String(err));
      } finally {
        this.loading = false;
      }
    },

    async waitForPyodide() {
      var tries = 0;
      while (!window.loadPyodide) {
        await new Promise((r) => setTimeout(r, 50));
        tries += 1;
        if (tries > 200) throw new Error('Pyodideが読み込めません。');
      }
    },

    clearOutput() {
      this.output = '';
    },

    async run() {
      if (!this.ready || this.loading || !this.input.trim()) return;
      this.loading = true;
      this.error = '';
      this.output = '';
      try {
        var result = await this.pyodide.runPythonAsync(this.input);
        if (result !== undefined && result !== null) {
          this.output += String(result);
        }
      } catch (err) {
        this.error = err && err.message ? err.message : String(err);
      } finally {
        this.loading = false;
      }
    }
  }
}
</script>

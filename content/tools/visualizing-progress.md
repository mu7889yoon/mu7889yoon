---
title: "活動見える化くん"
---

<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.15.3/dist/cdn.min.js"></script>

<style>
.heatmap-container { overflow-x: auto; }
.heatmap { display: flex; gap: 2px; }
.heatmap-week { display: flex; flex-direction: column; gap: 2px; }
.heatmap-cell { width: 12px; height: 12px; border-radius: 2px; }
.heatmap-cell:hover { outline: 1px solid #333; }
.heatmap-months { display: flex; font-size: 11px; color: #666; margin-bottom: 4px; padding-left: 32px; }
.heatmap-days { display: flex; flex-direction: column; gap: 2px; font-size: 11px; color: #666; margin-right: 4px; }
.heatmap-days span { height: 12px; line-height: 12px; }
.legend { display: flex; flex-wrap: wrap; align-items: center; gap: 4px; font-size: 12px; color: #666; margin-top: 8px; }
.legend-cell { width: 12px; height: 12px; border-radius: 2px; }
.tooltip { position: absolute; background: #1f1f1f; color: #fff; padding: 6px 10px; border-radius: 4px; font-size: 12px; pointer-events: none; z-index: 100; white-space: nowrap; }
</style>

<div x-data="progressVisualizer()" x-init="init()">
  <div style="margin-bottom: 1rem;">
    <label>期間選択</label><br>
    <select x-model="periodType" @change="updatePeriod()">
      <template x-for="year in years" :key="year">
        <option :value="year" x-text="year + '年'"></option>
      </template>
      <option value="custom">カスタム</option>
    </select>
  </div>
  <div x-show="periodType === 'custom'" style="margin-bottom: 1rem;">
    <label>開始日</label>
    <input type="date" x-model="startDate" @change="render()">
    <label>終了日</label>
    <input type="date" x-model="endDate" @change="render()">
  </div>
  <div x-show="loading" style="color: #666;">読み込み中...</div>
  <div x-show="error" x-text="error" style="color: #b00020;"></div>

  <div class="heatmap-months" x-html="monthsHtml"></div>
  <div style="display: flex;">
    <div class="heatmap-days">
      <span></span><span>月</span><span></span><span>水</span><span></span><span>金</span><span></span>
    </div>
    <div class="heatmap-container">
      <div class="heatmap" x-ref="heatmap"></div>
    </div>
  </div>

  <div class="legend">
    <div class="legend-cell" style="background:#ebedf0;"></div><span style="margin-right:8px;">なし</span>
    <div class="legend-cell" style="background:#4CAF50;"></div><span style="margin-right:8px;">個人ブログ</span>
    <div class="legend-cell" style="background:#FF9800;"></div><span style="margin-right:8px;">会社ブログ</span>
    <div class="legend-cell" style="background:#2196F3;"></div><span style="margin-right:8px;">LT/登壇</span>
    <div class="legend-cell" style="background:#9C27B0;"></div><span style="margin-right:8px;">運営</span>
    <div class="legend-cell" style="background:#607D8B;"></div><span>その他</span>
  </div>

  <div style="margin-top: 1rem;"><strong>合計:</strong> <span x-text="totalCount"></span> 件</div>
  <div x-ref="tooltip" class="tooltip" style="display:none;"></div>
</div>

<script>
function progressVisualizer() {
  var TYPE_COLORS = { blog: '#4CAF50', 'seeds-blog': '#FF9800', lt: '#2196F3', organizer: '#9C27B0', other: '#607D8B' };
  var TYPE_LABELS = { blog: '個人ブログ', 'seeds-blog': '会社ブログ', lt: 'LT/登壇', organizer: 'コミュニティ運営', other: 'その他' };
  var MONTHS = ['1月','2月','3月','4月','5月','6月','7月','8月','9月','10月','11月','12月'];
  var START_YEAR = 2024, CURRENT_YEAR = new Date().getFullYear();

  return {
    loading: false, error: '', periodType: String(CURRENT_YEAR),
    startDate: '', endDate: '', blogData: [], outputData: [],
    totalCount: 0, initialized: false, monthsHtml: '',
    years: Array.from({ length: CURRENT_YEAR - START_YEAR + 1 }, (_, i) => CURRENT_YEAR - i),

    async init() {
      if (this.initialized) return;
      this.initialized = true;
      await this.$nextTick();
      this.updatePeriod();
      await this.loadData();
      this.render();
    },

    updatePeriod() {
      if (this.periodType !== 'custom') {
        var year = parseInt(this.periodType);
        this.startDate = year + '-01-01';
        this.endDate = year + '-12-31';
      }
      this.render();
    },

    async loadData() {
      this.loading = true;
      try {
        var blogRes = await fetch('/index.xml');
        this.blogData = this.parseBlogXml(await blogRes.text());
        try {
          var outputRes = await fetch('/data/output.jsonl');
          this.outputData = this.parseJsonl(await outputRes.text());
        } catch (e) { this.outputData = []; }
      } catch (e) {
        this.error = 'データの読み込みに失敗: ' + e.message;
      }
      this.loading = false;
    },

    parseBlogXml(xml) {
      var doc = new DOMParser().parseFromString(xml, 'text/xml');
      var data = [];
      doc.querySelectorAll('item').forEach(item => {
        var pubDate = item.querySelector('pubDate');
        if (pubDate) data.push({ date: new Date(pubDate.textContent).toISOString().split('T')[0], type: 'blog' });
      });
      return data;
    },

    parseJsonl(text) {
      if (!text.trim()) return [];
      return text.trim().split('\n').map(line => {
        try { var obj = JSON.parse(line); if (obj.date && obj.type) return { date: obj.date.split('T')[0], type: obj.type }; } catch (e) {}
        return null;
      }).filter(Boolean);
    },

    render() {
      if (!this.$refs.heatmap) return;
      var start = new Date(this.startDate), end = new Date(this.endDate), byDate = {};

      this.blogData.concat(this.outputData).forEach(item => {
        var d = new Date(item.date);
        if (d >= start && d <= end) {
          if (!byDate[item.date]) byDate[item.date] = [];
          byDate[item.date].push(item.type);
        }
      });

      var current = new Date(start);
      current.setDate(current.getDate() - current.getDay());
      var weeks = [], monthPositions = [], lastMonth = -1, weekIndex = 0;

      while (current <= end || current.getDay() !== 0) {
        var week = [];
        for (var d = 0; d < 7; d++) {
          var dateStr = current.toISOString().split('T')[0];
          var inRange = current >= start && current <= end;
          var types = inRange && byDate[dateStr] ? byDate[dateStr] : [];
          if (inRange && current.getMonth() !== lastMonth) {
            monthPositions.push({ month: current.getMonth(), week: weekIndex });
            lastMonth = current.getMonth();
          }
          week.push({ date: dateStr, count: types.length, types: types, inRange: inRange });
          current.setDate(current.getDate() + 1);
        }
        weeks.push(week);
        weekIndex++;
        if (current > end && current.getDay() === 0) break;
      }

      this.monthsHtml = monthPositions.map((pos, i) => {
        var next = monthPositions[i + 1];
        var width = next ? (next.week - pos.week) * 14 : 56;
        return '<span style="width:' + width + 'px;">' + MONTHS[pos.month] + '</span>';
      }).join('');

      var total = 0, html = '';
      weeks.forEach(week => {
        html += '<div class="heatmap-week">';
        week.forEach(day => {
          var style = !day.inRange ? 'background:transparent;' : day.count === 0 ? 'background:#ebedf0;' : (() => {
            var unique = [...new Set(day.types)];
            if (unique.length === 1) return 'background:' + TYPE_COLORS[unique[0]] + ';';
            var grad = unique.map((t, i) => TYPE_COLORS[t] + ' ' + (i/unique.length*100) + '%, ' + TYPE_COLORS[t] + ' ' + ((i+1)/unique.length*100) + '%').join(', ');
            return 'background:linear-gradient(135deg, ' + grad + ');';
          })();
          total += day.count;
          html += '<div class="heatmap-cell" style="' + style + '" data-date="' + day.date + '" data-count="' + day.count + '" data-types="' + day.types.join(',') + '"></div>';
        });
        html += '</div>';
      });
      this.$refs.heatmap.innerHTML = html;
      this.totalCount = total;

      var tooltip = this.$refs.tooltip;
      this.$refs.heatmap.querySelectorAll('.heatmap-cell').forEach(cell => {
        cell.addEventListener('mouseenter', e => {
          var types = cell.dataset.types ? cell.dataset.types.split(',').filter(Boolean) : [];
          var text = cell.dataset.date + ': ' + cell.dataset.count + '件';
          if (types.length > 0) {
            var counts = {};
            types.forEach(t => counts[t] = (counts[t] || 0) + 1);
            text += '<br>' + Object.keys(counts).map(t => (TYPE_LABELS[t] || t) + ': ' + counts[t]).join('<br>');
          }
          tooltip.innerHTML = text;
          tooltip.style.display = 'block';
          tooltip.style.left = (e.pageX + 10) + 'px';
          tooltip.style.top = (e.pageY + 10) + 'px';
        });
        cell.addEventListener('mouseleave', () => tooltip.style.display = 'none');
      });
    }
  };
}
</script>

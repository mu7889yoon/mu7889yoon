---
title: "UUID v7生成くん"
---

<div class="uuidv7-tool">
  <div class="uuidv7-actions">
    <button id="uuidv7-generate">生成</button>
    <button id="uuidv7-copy" disabled>コピー</button>
  </div>
  <div>
    <label for="uuidv7-output">UUID v7</label><br>
    <input id="uuidv7-output" type="text" readonly>
  </div>
</div>

<script>
(function() {
  function bytesToUuid(bytes) {
    var hex = Array.prototype.map.call(bytes, function(byte) {
      return byte.toString(16).padStart(2, '0');
    }).join('');
    return hex.slice(0, 8) + '-' + hex.slice(8, 12) + '-' + hex.slice(12, 16) + '-' + hex.slice(16, 20) + '-' + hex.slice(20);
  }

  function generateUuidV7() {
    var bytes = new Uint8Array(16);
    var ts = BigInt(Date.now());
    for (var i = 0; i < 6; i++) {
      bytes[i] = Number((ts >> BigInt((5 - i) * 8)) & 0xffn);
    }
    var rand = new Uint8Array(10);
    crypto.getRandomValues(rand);
    bytes[6] = 0x70 | (rand[0] & 0x0f);
    bytes[7] = rand[1];
    bytes[8] = (rand[2] & 0x3f) | 0x80;
    for (var j = 0; j < 7; j++) {
      bytes[9 + j] = rand[3 + j];
    }
    return bytesToUuid(bytes);
  }

  var generateButton = document.getElementById('uuidv7-generate');
  var copyButton = document.getElementById('uuidv7-copy');
  var output = document.getElementById('uuidv7-output');

  if (!window.crypto || !crypto.getRandomValues) {
    generateButton.disabled = true;
    copyButton.disabled = true;
    output.value = 'このブラウザではUUID v7の生成ができません。';
    return;
  }

  generateButton.addEventListener('click', function() {
    output.value = generateUuidV7();
    copyButton.disabled = false;
  });

  copyButton.addEventListener('click', function() {
    if (!output.value) return;
    navigator.clipboard.writeText(output.value).catch(function() {});
  });
})();
</script>

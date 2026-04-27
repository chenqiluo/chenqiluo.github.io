---
title: "AI天梯榜"
date: 2026-04-27
draft: false
description: "Chatbot Arena — 全球AI模型竞技排行榜"
showToc: false
TocOpen: false
hidemeta: true
comments: false
disableShare: true
searchHidden: true
---

<style>
.al-wrap {
--al-bg: #f5f5f5;
--al-border: #e0e0e0;
--al-text: #555;
--al-btn-bg: #0070f3;
--al-btn-hover: #0057c2;
}
.dark .al-wrap {
--al-bg: #1a1a2e;
--al-border: #2d3139;
--al-text: #aaa;
}
.al-wrap {
width: 100%;
display: flex;
flex-direction: column;
gap: 0;
}
.al-header {
display: flex;
align-items: center;
justify-content: space-between;
padding: 10px 14px;
background: var(--al-bg);
border: 1px solid var(--al-border);
border-bottom: none;
border-radius: 8px 8px 0 0;
flex-wrap: wrap;
gap: 8px;
}
.al-header-title {
font-size: 14px;
color: var(--al-text);
display: flex;
align-items: center;
gap: 6px;
}
.al-header-title svg {
flex-shrink: 0;
}
.al-open-btn {
display: inline-flex;
align-items: center;
gap: 5px;
padding: 6px 14px;
background: var(--al-btn-bg);
color: #fff !important;
border-radius: 6px;
font-size: 13px;
text-decoration: none !important;
transition: background 0.2s;
white-space: nowrap;
}
.al-open-btn:hover {
background: var(--al-btn-hover);
}
.al-frame-box {
position: relative;
width: 100%;
height: 85vh;
min-height: 600px;
border: 1px solid var(--al-border);
border-radius: 0 0 8px 8px;
overflow: hidden;
background: var(--al-bg);
}
.al-frame-box iframe {
width: 100%;
height: 100%;
border: none;
display: block;
}
.al-blocked {
display: none;
flex-direction: column;
align-items: center;
justify-content: center;
height: 100%;
gap: 16px;
padding: 32px;
text-align: center;
}
.al-blocked svg {
opacity: 0.3;
}
.al-blocked p {
color: var(--al-text);
font-size: 15px;
line-height: 1.7;
max-width: 420px;
}
.al-blocked a {
color: var(--al-btn-bg);
text-decoration: underline;
}
</style>

<div class="al-wrap">
<div class="al-header">
<div class="al-header-title">
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/></svg>
来源：<strong>arena.ai</strong> / Chatbot Arena Text Leaderboard
</div>
<a class="al-open-btn" href="https://arena.ai/leaderboard/text" target="_blank" rel="noopener">
<svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/><polyline points="15 3 21 3 21 9"/><line x1="10" y1="14" x2="21" y2="3"/></svg>
在新窗口打开
</a>
</div>
<div class="al-frame-box" id="al-frame-box">
<iframe
id="al-iframe"
src="https://arena.ai/leaderboard/text"
title="AI天梯榜 — Chatbot Arena"
loading="lazy"
sandbox="allow-scripts allow-same-origin allow-forms allow-popups"
></iframe>
<div class="al-blocked" id="al-blocked">
<svg width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="3" y="3" width="18" height="18" rx="3"/><line x1="9" y1="3" x2="9" y2="21"/><line x1="3" y1="9" x2="21" y2="9"/></svg>
<p>该页面设置了安全策略，<strong>不允许被嵌入</strong>到其他网站中显示。<br>请点击上方按钮在新窗口中查看完整排行榜 👆</p>
<a href="https://arena.ai/leaderboard/text" target="_blank" rel="noopener">直接访问 arena.ai/leaderboard/text →</a>
</div>
</div>
</div>

<script>
(function() {
var iframe = document.getElementById('al-iframe');
var blocked = document.getElementById('al-blocked');
if (!iframe || !blocked) return;
var timer = setTimeout(function() {
try {
var doc = iframe.contentDocument || iframe.contentWindow.document;
if (!doc || doc.body.innerHTML === '') {
iframe.style.display = 'none';
blocked.style.display = 'flex';
}
} catch(e) {
iframe.style.display = 'none';
blocked.style.display = 'flex';
}
}, 4000);
iframe.addEventListener('load', function() {
clearTimeout(timer);
try {
var doc = iframe.contentDocument || iframe.contentWindow.document;
if (!doc || doc.body.innerHTML === '') {
iframe.style.display = 'none';
blocked.style.display = 'flex';
}
} catch(e) {
iframe.style.display = 'none';
blocked.style.display = 'flex';
}
});
iframe.addEventListener('error', function() {
clearTimeout(timer);
iframe.style.display = 'none';
blocked.style.display = 'flex';
});
})();
</script>

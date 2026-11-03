---
layout: default
title: Home
---

<details style="margin-bottom: 20px; text-align: left;">
  <summary>Builds by month</summary>
  <button type="button" id="chart-toggle" style="margin-bottom: 10px;">Show cumulative totals</button>
  <div id="build-month-chart"></div>
</details>

<script>
const buildMonths = [
{% for device in site.data.devices_list %}
  {% assign chart_device_data = site.data.devices[device.codename] %}
  {% for rom in chart_device_data.roms %}
    {% for build in rom.builds %}
      {% if build.date %}
      "{{ build.date | slice: 0,7 }}",
      {% endif %}
    {% endfor %}
  {% endfor %}
{% endfor %}
];
var chartMode = 'month';
function renderLine(values, labels, max) {
  var chart = document.getElementById('build-month-chart');
  if (!chart) return;
  while (chart.firstChild) chart.removeChild(chart.firstChild);
  var ns = 'http://www.w3.org/2000/svg';
  function el(tag, attrs) {
    var x = document.createElementNS(ns, tag);
    Object.keys(attrs).forEach(function(k){ x.setAttribute(k, attrs[k]); });
    return x;
  }
  var width = 800, height = 220, left = 30, right = 70, top = 16, bottom = 38;
  var pw = width - left - right, ph = height - top - bottom;
  var svg = el('svg', { viewBox: '0 0 ' + width + ' ' + height, width: '100%', height: height, preserveAspectRatio: 'none' });
  var axis = el('line', { x1: left, y1: top + ph, x2: left + pw, y2: top + ph, stroke: '#ccc', 'stroke-width': '1' });
  svg.appendChild(axis);
  var n = values.length;
  var pointData = values.map(function(v, i) {
    var x = left + (n === 1 ? pw / 2 : (i / (n - 1)) * pw);
    var y = top + (1 - (v / (max || 1))) * ph;
    return { x: x, y: y, v: v, label: labels[i] };
  });
  var polyPoints = pointData.map(function(p){ return p.x + ',' + p.y; }).join(' ');
  var line = el('polyline', { points: polyPoints, fill: 'none', stroke: '#007bff', 'stroke-width': '2' });
  svg.appendChild(line);
  pointData.forEach(function(p, i) {
    var c = el('circle', { cx: p.x, cy: p.y, r: (i === n - 1) ? '5' : '2', fill: '#007bff' });
    svg.appendChild(c);
  });
  var last = pointData[n - 1];
  if (last) {
    var latest = el('text', { x: last.x + 10, y: Math.max(top + 10, last.y), 'text-anchor': 'start', 'dominant-baseline': 'middle', 'font-size': '12', fill: '#007bff', 'font-weight': 'bold' });
    latest.textContent = last.v;
    svg.appendChild(latest);
  }
  labels.forEach(function(m, i) {
    if (m.slice(5) !== '12') return;
    var x = left + (n === 1 ? pw / 2 : (i / (n - 1)) * pw);
    var t = el('text', { x: x, y: height - 18, 'text-anchor': 'middle', 'font-size': '9', fill: 'currentColor' });
    t.textContent = m;
    svg.appendChild(t);
  });
  chart.appendChild(svg);
}
function renderMonth() {
  var counts = {};
  buildMonths.forEach(function(m) { if (m) counts[m] = (counts[m] || 0) + 1; });
  var monthKeys = Object.keys(counts).sort();
  if (monthKeys.length === 0) return;
  var start = monthKeys[0].split('-');
  var end = monthKeys[monthKeys.length - 1].split('-');
  var startDate = new Date(parseInt(start[0], 10), parseInt(start[1], 10) - 1, 1);
  var endDate = new Date(parseInt(end[0], 10), parseInt(end[1], 10) - 1, 1);
  var allMonths = [];
  var cursor = new Date(startDate.getTime());
  while (cursor <= endDate) {
    var y = cursor.getFullYear();
    var m = (cursor.getMonth() + 1);
    if (m < 10) m = '0' + m;
    allMonths.push(y + '-' + m);
    cursor = new Date(cursor.getFullYear(), cursor.getMonth() + 1, 1);
  }
  var values = allMonths.map(function(m) { return counts[m] || 0; });
  var max = values.length ? Math.max.apply(null, values.concat([1])) : 1;
  var chart = document.getElementById('build-month-chart');
  if (!chart) return;
  while (chart.firstChild) chart.removeChild(chart.firstChild);
  chart.style.display = 'flex';
  chart.style.alignItems = 'flex-end';
  chart.style.gap = '1px';
  chart.style.overflowX = 'hidden';
  chart.style.overflowY = 'hidden';
  chart.style.padding = '8px 0 40px 0';
  chart.style.minHeight = '220px';
  chart.style.width = '100%';
  allMonths.forEach(function(m, i) {
    var labelMonth = m.slice(5) === '03' || m.slice(5) === '06' || m.slice(5) === '09' || m.slice(5) === '12';
    var col = document.createElement('div');
    col.style.flex = '1 1 0';
    col.style.minWidth = '0';
    col.style.display = 'flex';
    col.style.flexDirection = 'column';
    col.style.alignItems = 'center';
    col.style.padding = '0 1px';
    var value = document.createElement('div');
    value.textContent = values[i] ? values[i] : '\u00A0';
    value.style.fontSize = '9px';
    value.style.marginBottom = '2px';
    value.style.minHeight = '1.2em';
    var bar = document.createElement('div');
    bar.style.width = '100%';
    bar.style.maxWidth = '24px';
    bar.style.height = Math.max(values[i] ? 2 : 0, Math.round((values[i] / max) * 130)) + 'px';
    bar.style.background = 'var(--primary-color, #007bff)';
    bar.style.borderRadius = '4px 4px 0 0';
    var label = document.createElement('div');
    label.textContent = labelMonth ? m : '\u00A0';
    label.style.fontSize = '8px';
    label.style.marginTop = '4px';
    label.style.transform = 'none';
    label.style.minHeight = '1.2em';
    label.style.transformOrigin = 'center center';
    label.style.whiteSpace = 'nowrap';
    col.appendChild(value);
    col.appendChild(bar);
    col.appendChild(label);
    chart.appendChild(col);
  });
}
function renderCumulative() {
  var counts = {};
  buildMonths.forEach(function(m) { if (m) counts[m] = (counts[m] || 0) + 1; });
  var monthKeys = Object.keys(counts).sort();
  if (monthKeys.length === 0) return;
  var start = monthKeys[0].split('-');
  var end = monthKeys[monthKeys.length - 1].split('-');
  var startDate = new Date(parseInt(start[0], 10), parseInt(start[1], 10) - 1, 1);
  var endDate = new Date(parseInt(end[0], 10), parseInt(end[1], 10) - 1, 1);
  var allMonths = [];
  var cursor = new Date(startDate.getTime());
  while (cursor <= endDate) {
    var y = cursor.getFullYear();
    var m = (cursor.getMonth() + 1);
    if (m < 10) m = '0' + m;
    allMonths.push(y + '-' + m);
    cursor = new Date(cursor.getFullYear(), cursor.getMonth() + 1, 1);
  }
  var cumulative = [];
  var running = 0;
  allMonths.forEach(function(m) { running += (counts[m] || 0); cumulative.push(running); });
  var max = cumulative.length ? Math.max.apply(null, cumulative.concat([1])) : 1;
  renderLine(cumulative, allMonths, max);
}
function render() { if (chartMode === 'cumulative') renderCumulative(); else renderMonth(); }
render();
var toggle = document.getElementById('chart-toggle');
if (toggle) {
  toggle.addEventListener('click', function() {
    if (chartMode === 'month') { chartMode = 'cumulative'; this.textContent = 'Show monthly counts'; }
    else { chartMode = 'month'; this.textContent = 'Show cumulative totals'; }
    render();
  });
}
</script>

{% assign total_builds = 0 %}
{% for device in site.data.devices_list %}
  {% assign device_data = site.data.devices[device.codename] %}
  {% for rom in device_data.roms %}
    {% assign rom_build_count = rom.builds | size %}
    {% assign total_builds = total_builds | plus: rom_build_count %}
  {% endfor %}
{% endfor %}

<p style="text-align: center; margin-bottom: 20px; font-size: 1.1rem;">Total builds tracked: <strong>{{ total_builds }}</strong></p>

<div class="device-grid">
  {% assign sorted_devices = site.data.devices_list | sort: "device" %}
  {% for device in sorted_devices %}
    {% assign device_data = site.data.devices[device.codename] %}
    {% assign device_build_count = 0 %}
    {% for rom in device_data.roms %}
      {% assign rom_build_count = rom.builds | size %}
      {% assign device_build_count = device_build_count | plus: rom_build_count %}
    {% endfor %}
    <a href="{{ device.url | relative_url }}" class="device-card">
      <h3>{{ device.device }}</h3>
      <p>Codename: <code style="display: inline-block;">{{ device.codename }}</code></p>
      <p>Builds: <strong>{{ device_build_count }}</strong></p>
      <span class="btn">View Releases &rarr;</span>
    </a>
  {% endfor %}
</div>
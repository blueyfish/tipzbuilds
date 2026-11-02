---
layout: default
title: Home
---

<div class="device-grid">
  {% for device in site.data.devices_list %}
    <a href="{{ device.url | relative_url }}" class="device-card">
      <h3>{{ device.device }}</h3>
      <p>Codename: <code style="display: inline-block;">{{ device.codename }}</code></p>
      <span class="btn">View Releases &rarr;</span>
    </a>
  {% endfor %}
</div>

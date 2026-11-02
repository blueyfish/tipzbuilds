---
layout: default
title: Home
---

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
    <a href="{{ device.url | relative_url }}" class="device-card">
      <h3>{{ device.device }}</h3>
      <p>Codename: <code style="display: inline-block;">{{ device.codename }}</code></p>
      <span class="btn">View Releases &rarr;</span>
    </a>
  {% endfor %}
</div>

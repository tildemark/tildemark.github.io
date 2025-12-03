---
layout: page
title: Gallery
permalink: /gallery/
icon: fas fa-images
order: 4
---

<div class="row row-cols-2 row-cols-md-3 g-4">
  {% for photo in site.data.gallery %}
  <div class="col">
    <div class="card h-100 border-0 bg-transparent">
      <img src="{{ photo.image }}" class="card-img-top rounded shadow-sm" alt="{{ photo.alt }}">
      {% if photo.title %}
      <div class="card-body px-0 py-2">
        <p class="card-text text-center text-muted small">{{ photo.title }}</p>
      </div>
      {% endif %}
    </div>
  </div>
  {% endfor %}
</div>

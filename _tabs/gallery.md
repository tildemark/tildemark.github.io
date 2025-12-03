---
layout: page
title: Gallery
permalink: /gallery/
icon: fas fa-camera-retro
order: 4
---

<div class="filter-buttons mb-4 text-center">
  <button class="btn btn-outline-primary active" onclick="filterGallery('all')">All</button>
  {% assign categories = site.data.gallery | map: 'category' | uniq %}
  {% for cat in categories %}
    <button class="btn btn-outline-primary" onclick="filterGallery('{{ cat | slugify }}')">
      {{ cat }}
    </button>
  {% endfor %}
</div>

<div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4" id="gallery-container">
  {% for photo in site.data.gallery %}
  <div class="col gallery-item {{ photo.category | slugify }}">
    <div class="card h-100 border-0 shadow-sm">
      <a href="{{ photo.image }}" class="popup-trigger">
        <img src="{{ photo.image }}" class="card-img-top rounded-top" alt="{{ photo.title }}" style="object-fit: cover; height: 250px;">
      </a>
      
      <div class="card-body">
        <h5 class="card-title h6">{{ photo.title }}</h5>
        
        {% if photo.tags %}
        <div class="mt-2">
          {% for tag in photo.tags %}
            <span class="badge bg-light text-dark border">{{ tag }}</span>
          {% endfor %}
        </div>
        {% endif %}
      </div>
    </div>
  </div>
  {% endfor %}
</div>

<script>
function filterGallery(category) {
  const items = document.getElementsByClassName('gallery-item');
  const buttons = document.querySelectorAll('.filter-buttons button');

  // 1. Manage Active Button Styling
  buttons.forEach(btn => {
    btn.classList.remove('active');
    // Simple check to highlight the clicked button
    if (btn.innerText.toLowerCase() === category || (category === 'all' && btn.innerText === 'All')) {
      btn.classList.add('active');
    }
  });

  // 2. Show/Hide Images
  for (let i = 0; i < items.length; i++) {
    const item = items[i];
    if (category === 'all') {
      item.classList.remove('d-none'); // Bootstrap class for display:none
      item.classList.add('animate__animated', 'animate__fadeIn'); // Optional animation
    } else {
      if (item.classList.contains(category)) {
        item.classList.remove('d-none');
        item.classList.add('animate__animated', 'animate__fadeIn');
      } else {
        item.classList.add('d-none');
      }
    }
  }
}
</script>

<style>
/* Optional: specific styling for the active filter button */
.filter-buttons .btn.active {
  background-color: var(--btn-btn-primary-bg);
  color: #fff;
}
</style>

---
layout: default
title: Labs
permalink: /labs/
eyebrow: Labs
---

<article class="page container">
  <header class="page-header reveal">
    <span class="eyebrow">Labs</span>
    <h1>Projects &amp; experiments</h1>
    <p class="meta">Things I'm building: open source, hackathon work, and side projects.</p>
  </header>

  <div class="proj-grid reveal">
    {%- for p in site.data.projects -%}
    <article class="card">
      <div class="ptop"><h3>{{ p.name }}</h3></div>
      {%- if p.tagline %}<p class="ptagline">{{ p.tagline }}</p>{% endif -%}
      <p class="blurb">{{ p.blurb }}</p>
      {%- if p.metric %}<span class="metric"><span class="pulse"></span>{{ p.metric }}</span>{% endif -%}
      {%- if p.stack %}<ul class="stack">{% for t in p.stack %}<li>{{ t }}</li>{% endfor %}</ul>{% endif -%}
      {%- if p.links %}<p class="plinks">{% for l in p.links %}<a href="{{ l.url }}" target="_blank" rel="noopener">{{ l.label }}</a>{% endfor %}</p>{% endif -%}
    </article>
    {%- endfor -%}
  </div>
</article>

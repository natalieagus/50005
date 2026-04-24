---
layout: default
title: Class Calendar
permalink: /admin/class-calendar
parent: Administration
nav_order: 3
---

{% assign badge_labels = "exam:Exam,quiz:Quiz,lab:Lab,pa:PA,ext:Extended Lecture,off:No Lab,prep:Preparation" | split: "," %}

* TOC
{:toc}

# Class Calendar 2026
{:.no-toc}

Notes contain additional materials and explanations to complement the official slides. You do not need to study the notes exhaustively for exams; the official slides are sufficient for revision. You are expected to attend lessons in person.

<span class="badge badge-exam">Exam</span> <span class="badge badge-quiz">Quiz</span> <span class="badge badge-lab">Lab</span> <span class="badge badge-pa">PA</span> <span class="badge badge-ext">Extended Lecture</span> <span class="badge badge-off">No Lab</span> <span class="badge badge-prep">Preparation</span>

{% for w in site.data.calendar %}
---

{% if w.special %}
## Week {{ w.week }}{% if w.date and w.date != "" %} ({{ w.date }}){% endif %}: {{ w.special }}
{: #week-{{ w.week }} }
{% if w.special_desc %}

{{ w.special_desc }}
{% endif %}
{% else %}
## Week {{ w.week }} ({{ w.date }})
{: #week-{{ w.week }} }
{% endif %}

{% unless w.special %}
<div class="week-card">
    <div class="wr"><span class="wl">Topic</span><span><b>{{ w.topic }}</b></span></div>
    <div class="wr"><span class="wl">Textbook</span><span>{{ w.textbook }}</span></div>
    <div class="wr"><span class="wl">Notes</span><span>{% for n in w.notes %}<a href="{{ site.baseurl }}{{ n.url }}">{{ n.label }}</a>{% unless forloop.last %} · {% endunless %}{% endfor %}</span></div>
    <div class="wr"><span class="wl">Activities</span><span>{{ w.activities | join: " · " }}</span></div>
</div>
{% endunless %}

{% for e in w.events %}
{% assign badge_key = e.badge %}
{% assign badge_label = badge_key %}
{% for pair in badge_labels %}
{% assign kv = pair | split: ":" %}
{% if kv[0] == badge_key %}{% assign badge_label = kv[1] %}{% endif %}
{% endfor %}
<span class="badge badge-{{ badge_key }}">{{ badge_label }}</span> {{ e.text }}
{% if e.details %}
{% for d in e.details %}
- {{ d }}
{% endfor %}
{% endif %}

{% endfor %}
{% endfor %}

See [Policies]({{ site.baseurl }}/admin/policies) for exam rules, LOA procedures, and cheat sheet guidelines.
---
layout: default
title: Important Dates
permalink: /admin/important-dates
parent: Administration
nav_order: 2
---

{% assign badge_labels = "exam:Exam,quiz:Quiz,lab:Lab,pa:PA,ext:Extended Lecture,off:No Class,prep:Preparation" | split: "," %}
{% assign cal_url = site.baseurl | append: "/admin/class-calendar" %}

* TOC
{:toc}

# Summary of Important Dates
{:.no_toc}

Refer to the [Class Calendar]({{ cal_url }}) for full details on topics and weekly activities. All weekly class activities are due on the Sunday of their respective week at 23:59 and are not listed individually below.

{:.note}
Click on the weeks to bring you to the respective class calendar.

<table>
<thead>
<tr><th>Week</th><th>Date</th><th>Day</th><th>Time</th><th>Task</th></tr>
</thead>
<tbody>
{% for group in site.data.important_dates %}
{% for item in group.items %}
{% assign badge_key = item.badge %}
{% assign badge_label = badge_key %}
{% for pair in badge_labels %}
{% assign kv = pair | split: ":" %}
{% if kv[0] == badge_key %}{% assign badge_label = kv[1] %}{% endif %}
{% endfor %}
{% if item.span %}
<tr>
<td><a href="{{ cal_url }}#week-{{ group.week }}">{{ group.week }}</a></td>
<td colspan="3">{{ item.time }}</td>
<td><span class="badge badge-{{ badge_key }}">{{ badge_label }}</span> {{ item.task }}</td>
</tr>
{% else %}
<tr>
<td><a href="{{ cal_url }}#week-{{ group.week }}">{{ group.week }}</a></td>
<td>{{ item.date }}</td>
<td>{{ item.day }}</td>
<td>{{ item.time }}</td>
<td><span class="badge badge-{{ badge_key }}">{{ badge_label }}</span> {{ item.task }}</td>
</tr>
{% endif %}
{% endfor %}
{% endfor %}
</tbody>
</table>
| Skill | Level |
| ---- | ---- |
{% assign skills = site.data.skills.technical -%}
{% for skill in skills -%}
| {{skill.title}} | {{skill.level}} |
{% endfor %}

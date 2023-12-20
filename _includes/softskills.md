| Skill | Level |
| ---- | ---- |
{% assign skills = site.data.skills.soft -%}
{% for skill in skills -%}
| {{skill.title}} | {{skill.level}} |
{% endfor %}
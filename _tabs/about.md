---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

{% assign s = site.data[site.active_lang].strings.about %}

{{ s.intro }}

{{ s.welcome }}

**{{ s.education_heading }}**
{% for item in s.education %}
- {{ item }}
{% endfor %}

**{{ s.contact_heading }}**

[![Facebook](https://img.shields.io/badge/facebook-F1F1F1?style=social&logo=facebook&logoColor=%230866FF&link=https%3A%2F%2Fwww.facebook.com%2Fphatndt)](https://www.facebook.com/phatndt)
[![Gmail](https://img.shields.io/badge/gmail-F1F1F1?style=social&logo=Gmail&logoColor=%#EA4335&link=mailto:phatndt2109@gmail.com)](mailto:phatndt2109@gmail.com)
[![Linkedin](https://img.shields.io/badge/linkedin-F1F1F1?style=social&logo=linkedin&logoColor=%#0A66C2&link=https://www.linkedin.com/in/phatndt/)](https://www.linkedin.com/in/phatndt/)
[![Github](https://img.shields.io/badge/github-F1F1F1?style=social&logo=Github&logoColor=%#181717&link=https://github.com/phatndt)](https://github.com/phatndt)

**{{ s.resume_heading }}**: {{ s.resume_status }}
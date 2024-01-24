---
title: Instrucciones hospedadas en línea
permalink: index.html
layout: home
---

# Ejercicios de Azure Data Engineering

Los siguientes ejercicios admiten los módulos de aprendizaje de Microsoft Learn que son compatibles con la certificación [Microsoft Certified: Azure Data Engineer Associate](https://learn.microsoft.com/certifications/azure-data-engineer/).

Para completar estos ejercicios, necesitarás una [suscripción a Microsoft Azure](https://azure.microsoft.com/free) en la que tengas acceso administrativo. Para algunos ejercicios, es posible que también necesites acceso a un [inquilino de Microsoft Power BI](https://learn.microsoft.com/power-bi/fundamentals/service-self-service-signup-for-power-bi).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Ejercicio | En ILT, se trata de... |
| --- | --- |
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) | {{ activity.lab.ilt-use }} |
{% endfor %}

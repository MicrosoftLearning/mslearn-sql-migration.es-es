---
title: Ejercicios de Azure OpenAI
permalink: index.html
layout: home
---

# Ejercicios de migración de SQL Server

Los ejercicios siguientes están diseñados para admitir los módulos de la ruta de aprendizaje [Migración de las cargas de trabajo de SQL Server a Azure SQL](https://learn.microsoft.com/training/paths/migrate-sql-workloads-azure/) en Microsoft Learn.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
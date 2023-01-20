---
layout: doc/document
title: Plugins current statuese
show_sidebar: false
menubar: docs_menu
---

| Package      | Latest Version | Build status | Coverage | Quality | Downloads |
|--------------|----------------|--------------|----------|---------|-----------|
{% for package in site.packages_official %}
| micro/{{ package }} | LATEST | DOWNLOADS | COVERAGE | Quality | ![Packagist Downloads](https://img.shields.io/packagist/dm/micro/{{package}}?label=installs) |"
{% endfor %}

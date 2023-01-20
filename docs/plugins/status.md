---
layout: doc/document
title: Plugins current statuese
show_sidebar: false
menubar: docs_menu
---

| Package      | Latest Version | Build status | Coverage | Quality | Downloads |
|--------------|----------------|--------------|----------|---------|-----------|

<table class="responsive-table table">
  <thead>
    <tr>
      <th scope="col"> Package </th>
      <th scope="col"> Latest release</th>
    </tr>
  </thead>
  <tbody>
    {% for package in site.packages_official %}
        <tr>
            <td>
                micro/{{ package }}
            </td>
            <td>
                
            </td>
            <td>
                
            </td>
            <td>
                
            </td>
            <td>
                
            </td>
            <td>
                <img src="https://img.shields.io/packagist/dm/micro/{{package}}?label=installs" />
            </td>
        </tr>
{% endfor %}
  </tbody>
</table>
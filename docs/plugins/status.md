---
layout: doc/document
title: Plugins current statuese
show_sidebar: false
menubar: docs_menu
---

<table class="responsive-table table">
  <thead>
    <tr>
      <th scope="col"> Package </th>
      <th scope="col"> Latest release</th>
      <th scope="col"> Build status </th>
      <th scope="col"> Package dev info </th>
      <th> Statistics </th>
    </tr>
  </thead>
  <tbody>
    {% for package in site.packages_official %}
        <tr>
            <td>
                micro/{{ package }}
            </td>
            <td>
                <p>Stable: <img alt="Packagist Version" src="https://img.shields.io/packagist/v/micro/{{package}}"></p>
                <p>Pre-release: Pre-release: <img alt="Packagist Version (including pre-releases)" src="https://img.shields.io/packagist/v/micro/{{package}}?include_prereleases"></p>
            </td>
            <td>
                <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package}}/badges/build.png?b=master">
            </td>
            <td>
                <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package}}/badges/quality-score.png?b=master" />
                <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package}}/badges/coverage.png?b=master" />
                <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package}}/badges/code-intelligence.svg?b=master" />
            </td>
            <td>
            <td>
                <img src="https://img.shields.io/packagist/dm/micro/{{package}}?label=installs" />
            </td>
        </tr>
{% endfor %}
  </tbody>
</table>
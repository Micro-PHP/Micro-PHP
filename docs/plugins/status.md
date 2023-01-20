---
layout: doc/document
title: Framework components status
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
            <b>micro/{{ package.name }}</b>
        </td>
        <td>
            <img alt="Packagist Version" src="https://img.shields.io/packagist/v/micro/{{package.name}}" />
        </td>
        <td>
            <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package.github}}/badges/build.png?b=master" />
        </td>
        <td>
            <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package.github}}/badges/quality-score.png?b=master" /> <br />
            <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package.github}}/badges/coverage.png?b=master" /> <br />
            <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package.github}}/badges/code-intelligence.svg?b=master" /> <br />
        </td>
        <td>
            <img src="https://img.shields.io/packagist/dm/micro/{{package.name}}?label=installs" />
        </td>
    </tr>
    {% endfor %}
    </tbody>
</table>
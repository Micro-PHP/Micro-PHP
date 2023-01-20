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
            <a href="https://scrutinizer-ci.com/g/Micro-PHP/{{package.github}}" >
                <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package.github}}/badges/quality-score.png?b=master" /> <br />
                <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package.github}}/badges/coverage.png?b=master" /> <br />
                <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package.github}}/badges/code-intelligence.svg?b=master" /> <br />
            </a>
        </td>
        <td>
            <img alt="Installs of the package `{{packege.name}}` today" src="https://img.shields.io/packagist/dd/micro/{{package.name}}?label=today">
            <img alt="Installs of the package `{{packege.name}} per month" src="https://img.shields.io/packagist/dm/micro/{{package.name}}?label=per month" />
            <img alt="Installs of the package `{{packege.name}}` for all time" src="https://img.shields.io/packagist/dt/micro/{{package.name}}?label=all time">
        </td>
    </tr>
    {% endfor %}
    </tbody>
</table>
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
            <a href="https://github.com/Micro-PHP/{{package.github}}" >
                <svg height="32" aria-hidden="true" viewBox="0 0 16 16" version="1.1" width="32" data-view-component="true" class="octicon octicon-mark-github v-align-middle">
                    <path fill-rule="evenodd" d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"></path>
                </svg>
            </a>
        </td>
        <td>
            <img alt="Packagist Version" src="https://img.shields.io/packagist/v/micro/{{package.name}}" />
        </td>
        <td>
            <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package.github}}/badges/build.png?b=master" />
        </td>
        <td>
            <a href="https://scrutinizer-ci.com/g/Micro-PHP/{{package.github}}" target="_blank">
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
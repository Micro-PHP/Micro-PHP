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
            <b>micro/{{ package }}</b>
        </td>
        <td>
            <tr>
                <td><b>Stable</b></td> <td> <img alt="Packagist Version" src="https://img.shields.io/packagist/v/micro/{{package}}" /></td>
            </tr>
            <tr>
                <td><b>Pre-release</b></td> <td> <img alt="Packagist Version (including pre-releases)" src="https://img.shields.io/packagist/v/micro/{{package}}?include_prereleases" /></td>
            </tr>
        </td>

        <td>
            <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package}}/badges/build.png?b=master" />
        </td>
        <td>
            <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package}}/badges/quality-score.png?b=master" />
            <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package}}/badges/coverage.png?b=master" />
            <img src="https://scrutinizer-ci.com/g/Micro-PHP/{{package}}/badges/code-intelligence.svg?b=master" />
        </td>
        <td>
            <img src="https://img.shields.io/packagist/dm/micro/{{package}}?label=installs" />
        </td>
    </tr>
    {% endfor %}
    </tbody>
</table>
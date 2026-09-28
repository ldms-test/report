# {{page.branch}}

{%
  assign reports = site.reports
                    | where: "type", "report"
                    | where: "branch", page.branch
                    | sort: "ts" | reverse
%}

{% for r in reports %}
<!-- count cases -->
{%
  assign tests = site.reports
                    | where: "type", "test"
                    | where: "branch", page.branch
                    | where: "ldmsCommitId8", r.ldmsCommitId8
%}

{% assign passed_tests = tests | where: "status", "passed" %}
{% assign queued_tests = tests | where: "status", "queued" %}
{% assign failed_tests = tests | where: "status", "failed" %}
{% assign passed = passed_tests.size %}
{% assign failed = failed_tests.size %}
{% assign queued = queued_tests.size %}
{% assign total = tests.size %}

{% if queued > 0 %}
{%   assign color = "blue" %}
{% elsif failed > 0 %}
{%   assign color = "red" %}
{% else %}
{%   assign color = "green" %}
{% endif %}
* \[{{r.ts | date: "%Y-%m-%d %H:%M:%S"}}\]
  [{{r.ldmsCommitId8}}]({{r.url | relative_url}})
  <span style="color:{{color}}">**{{passed}}**</span> / **{{total}}**

{% endfor %}

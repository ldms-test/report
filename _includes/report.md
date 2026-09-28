<!-- Common code for _branches/<BRANCH>/<COMMIT8>/report.md -->

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

## {{page.branch}} ({{page.ldmsCommitId8}})

<!-- Example of page info

layout: default
branch: b4.5
ldmsCommitId: 570ece6d2f6c3e7c6a4ce170341c765c51c8badc
testCommitId: 87039354b5fd9b996c947569561b15a45a3ea28c
ldmsCommitId8: 570ece6d
testCommitId8: 87039354
status: "done"
ts: 1789072346
-->

{%
  assign tests = site.reports
    | where: "branch", page.branch
    | where: "ldmsCommitId8", page.ldmsCommitId8
    | where: "type", "test"
%}
{% assign passed_tests = tests | where: "status", "passed" %}
{% assign queued_tests = tests | where: "status", "queued" %}
{% assign failed_tests = tests | where: "status", "failed" %}

<!-- count tests with passed status -->
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

<div style="position: relative; width: 250px">
<canvas id="chart" style="width: 100%"></canvas>
</div>
<script>
  new Chart(document.getElementById("chart").getContext('2d'), {
    type: 'doughnut',
    data: {
      labels: ['Failed', 'Queued', 'Passed'],
      datasets: [{
        data: [{{failed}}, {{queued}}, {{passed}}],
        backgroundColor: ['red', 'blue', 'green'],
      }]
    },
    options: {
      aspectRatio: 3,
      plugins: {
        legend: {
          position: "right"
        }
      }
    }
  });
</script>

* Docker Image: [ovishpc/ldms-build:{{page.ldmsCommitId8}}-amd64](https://hub.docker.com/layers/ovishpc/ldms-build/{{page.ldmsCommitId8}}-amd64/)
* LDMS branch: `{{page.branch}}`
* LDMS commit ID: `{{page.ldmsCommitId}}`
* LDMS-Test commit ID: `{{page.testCommitId}}`
* Status: <span style="color:{{color}}"><b>{{page.status}}</b></span>
* Timestamp: {{page.ts | date: "%Y-%m-%d %H:%M:%S" }}
* Passed: <b><span style="color:{{color}}">{{passed}}</span> / {{total}}</b>
* Tests:
{%- for t in tests -%}
{%-   if t.status == "passed" -%}
{%-     assign color = "green" -%}
{%-     assign logLink = 1 -%}
{%-   elsif t.status == "queued" -%}
{%-     assign color = "blue" -%}
{%-     assign logLink = 0 -%}
{%-   elsif t.status == "failed" -%}
{%-     assign color = "red" -%}
{%-     assign logLink = 1 -%}
{%-   endif -%}
{%-   if logLink > 0 -%}
{%-     capture link -%}
([log](tests/{{t.name}}/{{t.name}}.log))
{%-     endcapture -%}
{%-   else -%}
{%-     assign link = "" -%}
{%-   endif  %}
  * {{t.name}}: <span style="color:{{color}}">{{t.status}}</span> {{link}}
{%- endfor -%}

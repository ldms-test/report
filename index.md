<style>
summary::marker { content: "" }
details summary::after {
  content: "▶";
  display: inline-block;
  transition: transform 0.2s ease-in-out;
}
details[open] summary::after {
  transform: rotate(90deg);
}
</style>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

# Latest Tests

{% assign owner_name = site.github.owner_name %}
{% assign repo_name = site.github.repository_name %}
{% capture io_url %}https://{{owner_name}}.github.io/{{repo_name}}{% endcapture %}
{% capture repo_url %}https://github.com/{{owner_name}}/{{repo_name}}{% endcapture %}

{% assign hists = site.reports | where: "type", "history" %}

<!-- We have exactly one history object per branch -->
{% for hist in hists %}
{%
   assign reports = site.reports | where: "type", "report"
                                 | where: "branch", hist.branch
                                 | sort: "ts"
                                 | reverse
%}
{% assign r = reports[0] %}
{%
   assign tests = site.reports | where: "type", "test"
                               | where: "branch", r.branch
                               | where: "ldmsCommitId8", r.ldmsCommitId8
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

## {{r.branch}} ({{r.ldmsCommitId8}})

{% assign status_url = r.url | replace: "report.html", "status.json" %}

[![status](https://img.shields.io/endpoint?url={{io_url}}{{status_url}})]({{r.url}})

{%- capture summary -%}
Total passed: <b><span style="color:{{color}}">{{passed}}</span> / {{total}}</b>
(click to expand)
{%- endcapture -%}

{%  capture details  %}
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
([log](reports/{{t.branch}}/{{t.ldmsCommitId8}}/tests/{{t.name}}/{{t.name}}.log))
{%-    endcapture -%}
{%-   else -%}
{%-     assign link = "" -%}
{%-   endif  %}
  * {{t.name}}: <span style="color:{{color}}">{{t.status}}</span> {{link}}
{%- endfor -%}
{%  endcapture  %}


<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

{% if false %}
<!-- stacking bar chart -->
<div style="position: relative; width: 100%; height: 100px">
<canvas id="chart{{r.branch}}" style="width: 100%"></canvas>
</div>
<script>
  Chart.defaults.datasets.bar.maxBarThickness = 24;
  new Chart(document.getElementById('chart{{r.branch}}').getContext('2d'), {
    type: 'bar',
    data: {
      labels: ['Tests'],
      datasets: [
          {
            label: "Passed",
            data: [{{passed}}],
            backgroundColor: ['green']
          },
          {
            label: "Queued",
            data: [{{queued}}],
            backgroundColor: ['blue']
          },
          {
            label: "FaIled",
            data: [{{failed}}],
            backgroundColor: ['red']
          },
      ]
    },
    options: {
      responsive: true,
      indexAxis: 'y',
      aspectRatio: 3,
      scales: {
        x: {
          stacked: true,
        },
        y: {
          stacked: true,
        },
      }
    }
  });
</script>
{% else %}
<!-- pie / doughnut chart -->
<div style="position: relative; width: 250px">
<canvas id="chart{{r.branch}}" style="width: 100%"></canvas>
</div>
<script>
  new Chart(document.getElementById("chart{{r.branch}}").getContext('2d'), {
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
{% endif %}

* Docker Image: [ovishpc/ldms-build:{{r.ldmsCommitId8}}-amd64](https://hub.docker.com/layers/ovishpc/ldms-build/{{r.ldmsCommitId8}}-amd64/)
* LDMS branch: `{{r.branch}}`
* LDMS commit ID: `{{r.ldmsCommitId}}`
* LDMS-Test commit ID: `{{r.testCommitId}}`
* Status: {{r.status}}
* Timestamp: {{r.ts | date: "%Y-%m-%d %H:%M:%S" }}
* <details markdown="1">
  <summary>{{ summary }}  </summary>
  {{details}}
  </details>
* [See previous tests ...]({{hist.url}})

{% endfor %} <!-- hist -->

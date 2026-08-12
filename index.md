{%- for br in site.data.branches -%}
{%-   assign branch = br[0] -%}
{%-   assign info = br[1].info -%}
{%    assign tests = br[1].tests %}

## {{ branch }}

[![status](https://img.shields.io/endpoint?url=https://ldms-test.github.io/report/branches/{{branch}}/status.json)](https://ldms-test.github.io/report/#{{branch}})

Status: {{info.status}}

LDMS Commit ID: {{info.ldmsCommitId}}

Test Commit ID: {{info.testCommitId}}

Tests:

{%-   assign total = 0 -%}
{%-   assign passed = 0 -%}
{%-   for t in tests -%}
{%-     assign total = total | plus: 1 -%}
{%-     assign test = t[0] -%}
{%-     assign obj = t[1] -%}
{%-     if obj.status == "passed" -%}
{%-       assign passed = passed | plus: 1 -%}
{%-       assign color = "green" -%}
{%-       assign logLink = 1 -%}
{%-     elsif obj.status == "queued" -%}
{%-       assign color = "blue" -%}
{%-       assign logLink = 0 -%}
{%-     elsif obj.status == "failed" -%}
{%-       assign color = "red" -%}
{%-       assign logLink = 1 -%}
{%-     endif -%}
{%-     if logLink > 0 -%}
{%-       capture link -%}
([log](branches/{{branch}}/{{test}}/{{test}}.log))
{%-       endcapture -%}
{%-     else -%}
{%-       assign link = "" -%}
{%      endif %}
* {{test}}: <span style="color:{{color}}">{{obj.status}}</span> {{link}}

{%    endfor %}

Total passed: {{ passed }} / {{ total }}

{%- endfor -%}

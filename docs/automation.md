# Automation

## Salt stack

### Minion

Apply specific state from a minion[(doc)](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.state.html#module-salt.modules.state)

`sal-call` or `venv-salt-call`(SUSE Manager version)

```sh
venv-salt-call state.apply <sls> --state_output=changes
```

Test mode

```sh
venv-salt-call state.apply <sls> test=true --state_output=changes
```

#### Apply highstate

```sh
venv-salt-call state.highstate --state_output=changes
```

### Master

Delete specified minion key[(doc)](https://docs.saltproject.io/en/latest/ref/cli/salt-key.html)

!!! warning "Warning"

    Delete key, only if needed!

```sh
salt-key -d [minion_id]
```

# Cool things to note

## containerd_config.yml
We're using ansible.builtin.meta: flush_handlers to invoke and immediate running of handlers
In this case it's to restart containerd after modifying the config.toml
This ensures our modified SystemdCgroup value is present when querying crictl info

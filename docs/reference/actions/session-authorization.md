---
title: Local Action Execution
---

# Local Action Execution

Actions dispatched by the control plane and actions started with `plexd actions run` use
different paths. The control plane authorizes its own dispatches before placing them in the
node's authenticated state pull. The local CLI does not use a session token: it calls the
node API on the same machine, and the operating system controls access to that endpoint.

## Control-plane dispatch

The control plane decides which actions a node may run and includes authorized executions in
the authenticated state pull. plexd runs those entries and reports their results through the
execution reporting path. This is separate from the local `actions run` command.

```text
Control plane authorizes action
       |
       | execution in authenticated state pull
       v
plexd runs the execution and reports its result
```

## Local CLI execution

`plexd actions run` sends the action name and parameters to `POST /v1/actions/run` on the
local node API. The API runs a built-in action synchronously and returns its standard output,
standard error, and exit code to the CLI. This endpoint does not accept a session JWT or
check action scopes. The local runner supports built-in actions only; hooks are not supported
by this command. Its result is returned to the local caller and is not reported to the
control plane.

```text
Local caller with access to the node API
       |
       | plexd actions run <action> [--param key=value]
       v
Local node API: POST /v1/actions/run
       |
       | RunLocal
       v
Built-in action
       |
       | stdout, stderr, exit code
       +----------------------------------------> Local caller
```

On Linux and macOS, the node API uses the Unix socket `/var/run/plexd/api.sock` by default.
When the `plexd` group exists, the socket is owned by `root:plexd` with mode `0660`; when
the daemon cannot assign that group, it restricts the socket to its owner with mode `0600`.
On Windows, the node API uses its configured named pipe. In each case, access to the local
endpoint is the access check for local API requests.

The `PLEXD_SESSION_TOKEN` variable described in older documentation is not read by plexd
and does not authorize `plexd actions run`. Injecting that variable into an SSH session does
not change which local actions the node API permits.

See [Local Node API](../core/nodeapi.md) for endpoint configuration and
[Remote Actions and Hooks](./remote-actions-hooks.md) for actions dispatched by the control
plane.

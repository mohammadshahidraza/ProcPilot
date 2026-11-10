# ProcPilot

ProcPilot is a Linux command-line process management tool written in Bash. It
runs commands in the background, keeps track of every process it manages, shows
CPU and memory usage in a live terminal dashboard, and warns when overall system
resource usage crosses the configured thresholds.

This repository contains the team implementation for the Operating Systems
project.

## Features

- Start one or many commands in the background and keep them running.
- List every ProcPilot-managed process with its PID, command and status.
- Stop a managed process safely (other system processes are never touched).
- Monitor per-process CPU and memory usage and the overall system resources.
- Warn when system CPU or RAM usage crosses the configured thresholds.
- Show a live, `top`-style terminal dashboard.
- Write application events and per-process output to log files.

## Team and responsibilities

| Member | Responsibility | Files |
| --- | --- | --- |
| Muhammad Azan | Process execution and process management | `src/executor.sh`, `src/manager.sh`, `process_runner.sh`, `tests/test_execution.sh` |
| Muhammad Shahid Raza | CLI interface, configuration and integration | `src/config.sh`, `src/cli.sh`, `procpilot.sh`, `config/settings.conf`, `tests/test_cli.sh` |
| Muhammad Shaheer Asif | Process monitoring and resource management | `src/monitor.sh`, `src/resource_manager.sh`, `tests/test_monitoring.sh` |
| Abdul Rehman | Dashboard, logging and reporting | `src/dashboard.sh`, `src/logger.sh`, dashboard and logging tests |

The distribution is fixed. Function names, parameter order, shared variable
names and status strings follow the team specification and must not be changed
without team approval.

## Requirements

- Linux with Bash 4 or newer (GNU `ps`, `/proc` and standard coreutils).
- On Windows, use WSL for the real dashboard and process monitoring. The CLI
  and configuration modules also run under Git Bash for development.

## Project structure

```
ProcPilot/
├── procpilot.sh              # Main entry point (sources all modules)
├── process_runner.sh         # Internal supervisor for one managed command
├── src/
│   ├── config.sh             # load_config(), validate_config()
│   ├── cli.sh                # main(), show_help()
│   ├── executor.sh           # start_process()
│   ├── manager.sh            # stop_process(), list_managed_processes()
│   ├── monitor.sh            # refresh_process_state(), get_process_info(), get_process_counts()
│   ├── resource_manager.sh   # get_system_resources(), check_resources()
│   ├── dashboard.sh          # show_dashboard(), render_dashboard()
│   └── logger.sh             # write_log(), show_logs()
├── config/
│   └── settings.conf         # User-configurable settings
├── state/
│   └── processes/
│       └── <PID>/            # Runtime state, created automatically
├── logs/
│   ├── procpilot.log         # Shared application log, created automatically
│   └── processes/            # Per-process output logs, created automatically
├── tests/
│   ├── test_cli.sh
│   ├── test_execution.sh
│   ├── test_monitoring.sh
│   └── test_integration.sh
└── README.md
```

`state/`, `state/processes/`, `logs/` and `logs/processes/` are created by
`load_config()` on the first run and are not committed to Git.

## Usage

```bash
chmod +x procpilot.sh process_runner.sh

./procpilot.sh help
./procpilot.sh run sleep 60
./procpilot.sh run ping -c 4 127.0.0.1
./procpilot.sh list
./procpilot.sh stop 4242
./procpilot.sh dashboard
./procpilot.sh logs 50
```

| Command | Routed to |
| --- | --- |
| `run COMMAND [ARGUMENTS...]` | `start_process COMMAND [ARGUMENTS...]` |
| `list` | `list_managed_processes` |
| `stop PID` | `stop_process PID` |
| `dashboard` | `show_dashboard` |
| `logs [NUMBER]` | `show_logs [NUMBER]` |
| `help` | `show_help` |

`run` prints only the PID of the started process on standard output, so it can
be used in scripts: `pid="$(./procpilot.sh run sleep 60)"`.

## Configuration

`config/settings.conf` accepts three settings:

| Setting | Default | Meaning |
| --- | --- | --- |
| `PP_REFRESH_INTERVAL` | `2` | Dashboard refresh interval in seconds (positive integer) |
| `PP_CPU_WARNING_THRESHOLD` | `80` | CPU warning threshold as a percentage (1-100) |
| `PP_RAM_WARNING_THRESHOLD` | `80` | RAM warning threshold as a percentage (1-100) |

All paths (`PP_ROOT_DIR`, `PP_STATE_DIR`, `PP_PROCESSES_DIR`, `PP_LOG_DIR`,
`PP_APP_LOG`) are derived from the project root by `load_config()` and are not
configurable.

Invalid settings are reported by `validate_config()` and ProcPilot exits with a
non-zero status before running any command.

## Process state and status values

Every managed process has its own `state/processes/<PID>/` directory:

| File | Contents |
| --- | --- |
| `command.txt` | Original command and arguments |
| `status.txt` | Current status |
| `started_at.txt` | Start timestamp |
| `exit_code.txt` | Command exit code |
| `log_path.txt` | Path of the process log |
| `pid.txt` | Actual command PID |
| `supervisor_pid.txt` | Runner/supervisor PID |
| `stop_requested.txt` | Marker for a user-requested stop |

Status values are exactly: `running`, `completed`, `failed`, `stopped`.

## Testing

```bash
bash tests/test_cli.sh          # Configuration loading/validation and CLI routing
bash tests/test_execution.sh    # Process start, stop, concurrency, metadata
bash tests/test_monitoring.sh   # Process info, counts, CPU/RAM, warnings
bash tests/test_integration.sh  # End-to-end CLI, monitoring, dashboard, logs
```

`tests/test_cli.sh` runs in a temporary sandbox and never touches the real
`state/` or `logs/` directories.

## Development workflow

- `main` is the shared integration branch. It must stay in a working state.
- Each member works on a separate feature branch and merges into `main` only
  after review:
  - `feature/azan-execution`
  - `feature/shahid-cli-config`
  - `feature/shaheer-monitoring`
  - `feature/abdul-dashboard-logging`
- Source modules only define functions: sourcing a module must not execute any
  workflow.
- Always quote variable expansions, use `"$@"` to preserve argument boundaries,
  and never use `eval` on user-provided commands.
- Use project-relative paths; never hard-code personal absolute paths.

## Current status

- Configuration, CLI routing and the main entry point: implemented and tested
  (38 passing checks in `tests/test_cli.sh`).
- Process execution and management: pending.
- Process and resource monitoring: pending.
- Dashboard and logging: pending.

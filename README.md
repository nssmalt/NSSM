# NSSM



NSSM (Non-Sucking Service Manager) is a Windows service wrapper that runs ordinary executables, scripts, and background applications as services managed by the Windows Service Control Manager (SCM). It is useful when an application must start automatically at boot, operate without an interactive user session, or recover after an unexpected process exit. Typical deployments include Python workers, Node.js servers, Java applications, data processors, and internal monitoring agents.

Unlike applications developed specifically for the Windows service API, many command-line programs cannot register directly with SCM. NSSM bridges that gap by registering itself as the service executable and launching the configured application as a child process. Administrators can manage the resulting service using the Services console, standard Windows service commands, or the NSSM command-line interface.

Configuration covers executable paths, startup directories, command-line arguments, service account credentials, environment variables, startup modes, and dependencies. NSSM also supports standard output and error redirection, log rotation, process shutdown controls, and exit actions that determine whether an application restarts after termination. These options help maintain unattended workloads and capture diagnostic output.

NSSM is a standalone executable rather than a traditional installer. Keep its binary at a permanent location because installed services reference that executable. Creating or modifying services generally requires an elevated command prompt. Before deployment, verify the target application's unattended behavior, filesystem permissions, working directory, and network dependencies. A successfully started Windows service does not necessarily mean the underlying application is healthy; application-level monitoring remains necessary.

## **How to Install and Configure a Windows Service with NSSM**

To install a Windows service with NSSM, place `nssm.exe` in a stable directory and open Command Prompt as administrator. Install an existing executable by providing a service name and its absolute path:

```
nssm install DataWorker "C:\Apps\DataWorker\worker.exe"
```

For a script-based workload, point NSSM to its interpreter rather than the script file. For example, a Python background worker can be registered using:

```
nssm install PythonWorker "C:\Python312\python.exe" "C:\Apps\worker.py"
nssm set PythonWorker AppDirectory "C:\Apps"
```

The `AppDirectory` setting defines the working directory used for relative paths. Configure an automatic startup mode and an informative service description:

```
nssm set PythonWorker Start SERVICE_AUTO_START
nssm set PythonWorker Description "Background Python processing worker"
nssm start PythonWorker
nssm status PythonWorker
```

The service then appears in the Windows Services console. Alternatively, run `nssm install PythonWorker` without an executable path to open the graphical configuration dialog, where application paths, arguments, and other settings can be entered.

Use `nssm edit PythonWorker` to modify an installed service or `nssm get PythonWorker Application` to inspect its executable path. Check that the configured account can access application files and required resources.

For production deployments, consider using a dedicated service account with the minimum necessary permissions. If the workload depends on another Windows service, configure service dependencies to establish the correct startup order. Validate arguments containing spaces to avoid startup failures.

To stop and uninstall the example service, run `nssm stop PythonWorker` followed by `nssm remove PythonWorker confirm`.

## **NSSM Service Recovery, Automatic Restart, and Log Configuration**

NSSM provides service recovery controls for applications that exit unexpectedly. By default, it attempts to restart the monitored process. Administrators can specify an exit action and restart delay to limit rapid failure loops:

```
nssm set PythonWorker AppExit Default Restart
nssm set PythonWorker AppRestartDelay 5000
nssm set PythonWorker AppThrottle 1500
```

Delay values are measured in milliseconds. For programs that intentionally terminate after completing a task, consider an `Exit` action instead of continuous restarts. These settings govern application exits; they do not replace health checks for hung processes.

For troubleshooting, redirect standard output and standard error to persistent files:

```
nssm set PythonWorker AppStdout "C:\Logs\worker-out.log"
nssm set PythonWorker AppStderr "C:\Logs\worker-error.log"
nssm set PythonWorker AppRotateFiles 1
nssm set PythonWorker AppRotateBytes 10485760
```

Ensure the service identity has write access to the log directory. File rotation normally occurs when a service starts or restarts; online rotation requires separate configuration. The Windows Event Viewer can provide additional service lifecycle and exit information.

When stopping a service, NSSM uses configurable shutdown methods and timeouts to give the application opportunities to exit cleanly before stronger termination methods are used. Check application signal handling and child processes before relying on unattended shutdown.

For operational maintenance, use `nssm restart PythonWorker` after relevant configuration changes, and inspect `nssm status PythonWorker` together with application logs. If the service starts but the workload fails, verify executable paths, runtime dependencies, environment variables, and credentials before changing recovery settings. Repeated crashes should be investigated rather than hidden by automatic restarts, particularly for database workers and stateful applications.

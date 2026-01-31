# KIAUH Project - Technical Vulnerability Manual

This manual details the security vulnerabilities identified in the KIAUH project codebase, with examples, exact locations (file and line), and technical explanations for each issue. The goal is to provide a clear, technical, and actionable report for maintainers.

---

## Table of Contents

1. Insecure use of `shell=True` in subprocesses
2. Execution of commands with partial paths
3. Execution of untrusted input in subprocesses
4. Use of generic exceptions without logging
5. Possible command/variable injection in CI/CD scripts
6. General recommendations

---

## 1. Insecure use of `shell=True` in subprocesses

### Description
Using `shell=True` in functions like `subprocess.run` and `subprocess.check_output` can allow arbitrary command execution, facilitating command injection attacks.

### Examples

- [kiauh/components/klipper_firmware/firmware_utils.py#L115](kiauh/components/klipper_firmware/firmware_utils.py#L115)
  ```python
  blist: List[str] = check_output(cmd, shell=True, text=True).splitlines()[1:]
  ```
- [kiauh/components/klipper_firmware/firmware_utils.py#L182](kiauh/components/klipper_firmware/firmware_utils.py#L182)
  ```python
  shell=True,
  ```
- [kiauh/components/klipper_firmware/firmware_utils.py#L195](kiauh/components/klipper_firmware/firmware_utils.py#L195)
  ```python
  shell=True,
  ```
- [kiauh/components/klipper_firmware/firmware_utils.py#L208](kiauh/components/klipper_firmware/firmware_utils.py#L208)
  ```python
  shell=True,
  ```
- [kiauh/components/klipperscreen/klipperscreen.py#L83](kiauh/components/klipperscreen/klipperscreen.py#L83)
  ```python
  run(KLIPPERSCREEN_INSTALL_SCRIPT.as_posix(), shell=True, check=True)
  ```
- [kiauh/extensions/mobileraker/mobileraker_extension.py#L76](kiauh/extensions/mobileraker/mobileraker_extension.py#L76)
  ```python
  run(MOBILERAKER_INSTALL_SCRIPT.as_posix(), shell=True, check=True)
  ```
- [kiauh/extensions/obico/moonraker_obico.py#L91](kiauh/extensions/obico/moonraker_obico.py#L91)
  ```python
  run(cmd, check=True, shell=True)
  ```
- [kiauh/extensions/octoapp/octoapp.py#L60](kiauh/extensions/octoapp/octoapp.py#L60)
  ```python
  run(cmd, check=True, shell=True)
  ```
- [kiauh/extensions/octoapp/octoapp.py#L69](kiauh/extensions/octoapp/octoapp.py#L69)
  ```python
  run(OA_UPDATE_SCRIPT.as_posix(), check=True, shell=True, cwd=OA_DIR)
  ```
- [kiauh/extensions/octoeverywhere/octoeverywhere.py#L62](kiauh/extensions/octoeverywhere/octoeverywhere.py#L62)
  ```python
  run(cmd, check=True, shell=True)
  ```
- [kiauh/extensions/octoeverywhere/octoeverywhere.py#L71](kiauh/extensions/octoeverywhere/octoeverywhere.py#L71)
  ```python
  run(OE_UPDATE_SCRIPT.as_posix(), check=True, shell=True, cwd=OE_DIR)
  ```
- [kiauh/utils/fs_utils.py#L97](kiauh/utils/fs_utils.py#L97)
  ```python
  run(cmd, stderr=PIPE, check=True, shell=True)
  ```
- [kiauh/utils/git_utils.py#L302](kiauh/utils/git_utils.py#L302)
  ```python
  shell=True,
  ```
- [kiauh/utils/sys_utils.py#L427](kiauh/utils/sys_utils.py#L427)
  ```python
  homedir_perm = run(cmd, shell=True, stdout=PIPE, text=True)
  ```

### Risk
Allows unvalidated input to be interpreted as shell commands, potentially resulting in remote code execution.

### Recommendation
Always use `shell=False` and provide commands as argument lists. Validate and sanitize all external input.

---

## 2. Execution of commands with partial paths

### Description
Using partial paths (e.g., just the executable name without an absolute path) in subprocess calls can result in the execution of unexpected binaries if the system PATH is manipulated or compromised.

### Examples

- [kiauh/core/menus/base_menu.py#L29](kiauh/core/menus/base_menu.py#L29)
  ```python
  subprocess.call("clear -x", shell=True)
  ```
- [kiauh/extensions/klipper_backup/klipper_backup_extension.py#L59](kiauh/extensions/klipper_backup/klipper_backup_extension.py#L59)
  ```python
  ["crontab", "-l"]
  ```
- [kiauh/extensions/klipper_backup/klipper_backup_extension.py#L113](kiauh/extensions/klipper_backup/klipper_backup_extension.py#L113)
  ```python
  ["crontab", "-l"]
  ```
- [kiauh/extensions/klipper_backup/klipper_backup_extension.py#L121](kiauh/extensions/klipper_backup/klipper_backup_extension.py#L121)
  ```python
  ["crontab", "-"]
  ```
- [kiauh/extensions/spoolman/spoolman.py#L44](kiauh/extensions/spoolman/spoolman.py#L44)
  ```python
  ["docker", "compose", ...]
  ```
- [kiauh/extensions/spoolman/spoolman.py#L57](kiauh/extensions/spoolman/spoolman.py#L57)
  ```python
  ["docker", "--version"]
  ```
- [kiauh/extensions/spoolman/spoolman.py#L67](kiauh/extensions/spoolman/spoolman.py#L67)
  ```python
  ["docker", "compose", "version"]
  ```
- [kiauh/extensions/spoolman/spoolman.py#L72](kiauh/extensions/spoolman/spoolman.py#L72)
  ```python
  ["docker-compose", "--version"]
  ```
- [kiauh/utils/git_utils.py#L402](kiauh/utils/git_utils.py#L402)
  ```python
  ["git", "config", "--get", "remote.origin.url"]
  ```
- [kiauh/utils/sys_utils.py#L432](kiauh/utils/sys_utils.py#L432)
  ```python
  ["chmod", "og+x", Path.home()]
  ```

### Risk
Allows an attacker to execute malicious binaries present in PATH directories if the environment is compromised.

### Recommendation
Always use absolute paths for critical executables or validate the PATH environment before executing sensitive commands.

---

## 3. Execution of untrusted input in subprocesses

### Description
Executing subprocess commands with arguments from unvalidated input can allow an attacker to inject commands or manipulate system behavior.

### Examples

- [kiauh/components/crowsnest/crowsnest.py#L77](kiauh/components/crowsnest/crowsnest.py#L77)
  ```python
  run(cmd, stderr=PIPE, stdout=DEVNULL, check=True)
  ```
- [kiauh/components/webui_client/client_utils.py#L292](kiauh/components/webui_client/client_utils.py#L292)
  ```python
  run(command, stderr=PIPE, check=True)
  ```
- [kiauh/components/webui_client/client_utils.py#L308](kiauh/components/webui_client/client_utils.py#L308)
  ```python
  run(command, stderr=PIPE, check=True)
  ```
- [kiauh/components/webui_client/client_utils.py#L339](kiauh/components/webui_client/client_utils.py#L339)
  ```python
  run(command, stderr=PIPE, check=True)
  ```
- [kiauh/extensions/gcode_shell_cmd/assets/gcode_shell_command.py#L59](kiauh/extensions/gcode_shell_cmd/assets/gcode_shell_command.py#L59)
  ```python
  proc = subprocess.Popen(self.command + gcode_params, ...)
  ```
- [kiauh/procedures/system.py#L71](kiauh/procedures/system.py#L71)
  ```python
  run(cmd, stderr=PIPE, check=True)
  ```
- [kiauh/procedures/system.py#L82](kiauh/procedures/system.py#L82)
  ```python
  run(cmd, stderr=PIPE, check=True)
  ```
- [kiauh/procedures/system.py#L88](kiauh/procedures/system.py#L88)
  ```python
  run(cmd, stderr=PIPE, check=True)
  ```
- [kiauh/procedures/system.py#L95](kiauh/procedures/system.py#L95)
  ```python
  run(cmd, input=stdin.encode(), stderr=PIPE, stdout=PIPE, check=True)
  ```
- [kiauh/utils/fs_utils.py#L38](kiauh/utils/fs_utils.py#L38)
  ```python
  check_output(command, stderr=DEVNULL)
  ```
- [kiauh/utils/fs_utils.py#L63](kiauh/utils/fs_utils.py#L63)
  ```python
  run(cmd, stderr=PIPE, check=True)
  ```
- [kiauh/utils/fs_utils.py#L80](kiauh/utils/fs_utils.py#L80)
  ```python
  call(cmd, stderr=DEVNULL, stdout=DEVNULL)
  ```
- [kiauh/utils/fs_utils.py#L84](kiauh/utils/fs_utils.py#L84)
  ```python
  run(cmd, stderr=PIPE, check=True)
  ```
- [kiauh/utils/git_utils.py#L87](kiauh/utils/git_utils.py#L87)
  ```python
  result: str = check_output(cmd, stderr=DEVNULL).decode(encoding="utf-8")
  ```
- [kiauh/utils/git_utils.py#L107](kiauh/utils/git_utils.py#L107)
  ```python
  result: str = check_output(cmd, stderr=DEVNULL, cwd=repo).decode(encoding="utf-8")
  ```

### Risk
Allows external input to manipulate system commands, potentially causing arbitrary code execution, data leakage, or environment corruption.

### Recommendation
Validate and sanitize all arguments from external input before passing them to subprocesses.

---

## 4. Use of generic exceptions without logging

### Description
Using `except:` blocks without specifying the exception and without logging makes it difficult to identify and fix issues, and can mask critical errors.

### Examples

- [kiauh/extensions/klipper_backup/klipper_backup_extension.py#L53](kiauh/extensions/klipper_backup/klipper_backup_extension.py#L53)
  ```python
  except:
      Logger.print_error(f"Failed to remove {full_service_name}: {str(e)}")
  ```
- [kiauh/extensions/klipper_backup/klipper_backup_extension.py#L135](kiauh/extensions/klipper_backup/klipper_backup_extension.py#L135)
  ```python
  except:
      Logger.print_error("Unable to remove the Klipper-Backup moonraker entry")
  ```
- [kiauh/extensions/klipper_backup/klipper_backup_extension.py#L147](kiauh/extensions/klipper_backup/klipper_backup_extension.py#L147)
  ```python
  except:
      Logger.print_error("Unable to remove Klipper-Backup extension")
  ```
- [kiauh/extensions/gcode_shell_cmd/assets/gcode_shell_command.py#L38](kiauh/extensions/gcode_shell_cmd/assets/gcode_shell_command.py#L38)
  ```python
  except Exception:
      pass
  ```

### Risk
Failures may go unnoticed, making maintenance harder and increasing the risk of undetected bugs and vulnerabilities.

### Recommendation
Always specify the expected exception and log the error for later analysis.

---

## 5. Possible command/variable injection in CI/CD scripts

### Description
Interpolating uncontrolled variables in CI/CD scripts can allow arbitrary command execution, secret leakage, or manipulation of the build environment.

### Example

- [.github/workflows/release-ff-and-tag.yml#L30](.github/workflows/release-ff-and-tag.yml#L30)
  ```yaml
  run: |
    git tag ${{ inputs.tag_name }}
    git push origin ${{ inputs.tag_name }}
  ```

### Risk
Allows user-controlled values to be interpreted as commands, potentially compromising the CI/CD environment.

### Recommendation
Use intermediate environment variables and always validate/sanitize values before using them in commands.

---

## 6. General recommendations

- Always use `shell=False` in subprocesses and provide commands as argument lists.
- Prefer absolute paths for critical executables.
- Validate and sanitize all external input before executing system commands.
- Specify exceptions and log errors to facilitate maintenance.
- In CI/CD scripts, avoid direct interpolation of uncontrolled variables in commands.

---

This manual was automatically generated from a static security analysis (Ruff/Semgrep) performed on 2026-01-31.

For questions or suggestions, contact the project maintenance team.


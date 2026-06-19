# Script and Command Execution Rules

**MANDATE: These rules govern all script and command execution. They are not phase-specific. Adherence is mandatory.**

---

## 0. Agent Mode Adaptations

### 0.1. For Agents in Advisory Mode
If the agent cannot directly execute commands, it must adapt these rules as follows:

**Command Execution Protocol (Advisory Mode):**
1. Provide the exact command to be executed
2. Explain what successful output should look like
3. Request that the user execute the command
4. Request that the user paste the complete, unmodified output
5. Analyze the provided output according to Section 1.1 rules
6. Provide next steps based on the analysis

**Script Exclusivity Mandate (Advisory Mode):**
- The agent must instruct the user to use only the specified scripts
- The agent must not suggest alternative commands unless explicitly approved by the user
- The agent must verify (through user-reported output) that correct scripts were used

**All other rules in this document apply equally to both Autonomous and Advisory modes.**

---

## 1. Command Execution Protocol

### 1.1. Command Output Verification
The agent MUST meticulously inspect the output of EVERY command it executes to verify success. The agent must define its expectation of a successful output *before* executing the command and verify that the actual output matches this expectation. Ambiguous output MUST be treated as a failure.

### 1.2. Sequential Command Execution
- **Logical Chaining Allowed:** The agent may use logical chaining (e.g., `&&`) to ensure that dependent commands only execute upon the success of the preceding command. Chaining using `;` is still discouraged as it does not respect success/failure state.
- **Verify Before Proceeding:** After each command (or chain of commands), the agent MUST inspect the final output and verify success before issuing the next command.

### 1.3. Output Redirection
- **Standard Redirection:** For simple file creation or overwriting, standard redirection (`>`) is allowed.
- **Use `tee` for Critical Logs:** The agent MUST use `tee` for long-running processes, build logs, or any command where real-time visibility into the output is necessary for monitoring progress or debugging.

---

## 2. Standardized Scripts

### 2.1. Script Exclusivity Mandate
If `scripts/build_system.sh` and `scripts/run_system_test.sh` exist, the agent is **ABSOLUTELY FORBIDDEN** from using any other command or tool to build or run the project (e.g., `cargo build`, `cargo run`, `cargo test`).
- **Builds:** MUST use `scripts/build_system.sh`.
- **Tests/Execution:** MUST use `scripts/run_system_test.sh`.
- **Exception:** Any deviation requires explicit, case-by-case approval from the user for EACH command.

### 2.2. `setup_env.sh`: Health Check & Hermetic Setup
This script is the single source of truth for installing all applications, packages, tools, and other dependencies required for the project.
- **Mandate:** If the agent needs to install a new dependency (OS-level or package-level), it MUST first add the installation command to this script *before* executing the command in the session.
- **Idempotency:** The script MUST be idempotent. It should check if a dependency exists (e.g., `command -v playwright`) before attempting installation.
- **Verification:** The script MUST include verification steps for critical dependencies (e.g., checking versions) and fail-fast if the host environment is incompatible.
- **Environment Management:** This script MUST handle the setup of virtual environments (e.g., `venv`, `node_modules`) and the installation of necessary binaries (e.g., Playwright browsers).

### 2.3. `start_services.sh`
This script is responsible for performing any necessary configuration and starting all required background services.
- **Mandate:** If this script exists, it MUST be run as the second action of the session, immediately after the initial acknowledgment. The agent is forbidden from running it again without explicit user approval and must report its successful execution in the first message.

### 2.4. `build_system.sh`
This script performs all build steps for all packages, modules, and crates within the project.

### 2.5. `run_system_test.sh`: Automated Verification and Artifact Capture
This is the primary script for executing the full suite of automated system tests and verifications.
- **Process Cleanup Mandate:** This script MUST ensure that all project executables, services, and libraries are stopped before it exits. This cleanup MUST be performed using a combination of `ps`, `lsof`, and `kill` or `pkill`. The `fuser` command is explicitly forbidden.
- **Artifact Directory:** The script MUST use a standardized `test_outs/` directory for all test outputs.
- **Commit Artifacts:** All contents of `test_outs/` and `target/` (binaries) MUST be committed as part of the phase completion to prevent state loss.
- **Mandatory Captures:** The script MUST capture and save:
    -   Detailed console logs for all services.
    -   Screenshots and screen recordings for Playwright/UI tests.
    -   A `result_manifest.json` file that maps Requirement IDs to their corresponding artifacts.

### 2.6. `scripts/verification/run_verification_test.sh`
This is the primary script for the Verification Phase, designed to be parameterized to run specific tests based on a requirement ID.

### 2.7. `scripts/verification/verify_artifacts.sh`
This script performs automated analysis of the `test_outs/` directory.
- **Protocol:** It must verify that every requirement ID tested by the runner has produced the required artifacts defined in the `result_manifest.json`.

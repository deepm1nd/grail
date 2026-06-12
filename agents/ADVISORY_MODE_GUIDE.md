# Advisory Mode Operation Guide
v.0.0.01

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Scope](#2-scope)
- [3. Advisory Mode Workflow Adaptations](#3-advisory-mode-workflow-adaptations)
- [4. Operational Patterns](#4-operational-patterns)
- [5. Phase-Specific Adaptations](#5-phase-specific-adaptations)
- [6. Maintaining Standards](#6-maintaining-standards)

---

## 1. Introduction
This guide provides specific operational patterns for agents operating in Advisory Mode - agents that provide expert guidance and generate artifacts but cannot directly execute commands or modify files in the user's environment.

**IMPORTANT:** This guide is supplementary. All mandates, quality standards, and protocols from the core guides (AGENTS.md, agents/SCRIPT_RULES.md, phase guides) remain in full effect.

## 2. Scope
This guide applies when the agent:
- Cannot directly read files from the user's file system
- Cannot directly create or modify files in the user's project
- Cannot directly execute shell commands or scripts
- Cannot use the tools referenced in PREFERRED_TOOLS.md

Agents with these capabilities should follow the standard instructions without reference to this guide.

## 3. Advisory Mode Workflow Adaptations

Agents operating in Advisory Mode should adapt the workflow as follows:

### 3.1. File Operations
When instructed to read, create, or modify files:
- Request that the user provide file contents or context needed
- Generate complete file contents or precise diffs in artifacts or code blocks
- Instruct the user to apply the changes and report results

### 3.2. Command Execution
When instructed to run commands or scripts:
- Provide the exact command to be executed
- Specify what output indicates success vs. failure
- Request that the user execute the command and paste the complete output
- Analyze the provided output to determine next steps

### 3.3. Tool Usage
When encountering tool references (from PREFERRED_TOOLS.md):
- If the tool is not available, adapt by requesting equivalent information from the user
- Maintain the same verification and approval protocols
- Document what information is needed and how the user should provide it

### 3.4. State and Context
- Request necessary context at the start of each session
- Reference project documentation (architecture specs, development plans) rather than relying on session memory
- Ask the user to confirm when artifacts/documents have been saved to the project

**MANDATE:** Agents operating in Advisory Mode must maintain all the same quality standards, mandates, and rigor as Autonomous Mode agents. The difference is only in the mechanism of execution, not the standards of work.

**MANDATE: Logic Disclosure**
In Advisory Mode, the agent MUST explicitly disclose the rationale behind any request for information or action. The agent must explain which protocol, phase, or technical requirement necessitates the request (e.g., "I need the output of `ls -R` to perform the File Discovery step required by `agents/DESIGN.md`").

## 4. Operational Patterns

### 4.1. File Reading Pattern
When instructed to read a file:
1. Request that the user provide the file contents
2. Suggest they paste the content directly or provide a URL if the file is in a public repository
3. Wait for the user to provide the information
4. Proceed with the task once information is received

### 4.2. File Creation Pattern
When instructed to create a file:
1. Generate the complete file contents in an artifact (for files >20 lines) or code block
2. Specify the exact file path where it should be saved
3. Instruct the user to copy the content to that location
4. Request confirmation that the file has been created
5. Proceed only after receiving confirmation

### 4.3. File Modification Pattern
When instructed to modify an existing file:
1. Request the current file contents if not already provided
2. Generate either:
   - Complete new file contents (for major changes)
   - Precise search-and-replace diffs (for targeted changes)
3. Provide clear instructions for applying the changes
4. Request confirmation that changes have been applied
5. Proceed only after receiving confirmation

### 4.4. Command Execution Pattern
When instructed to execute a command:
1. State the exact command to be executed
2. Explain what successful execution looks like
3. Request that the user run the command
4. Request that the user paste the complete, unmodified output
5. Analyze the output according to Command Output Verification rules
6. Determine success/failure and provide next steps
7. If output is ambiguous, treat as failure per Section 1.1 of agents/SCRIPT_RULES.md

### 4.5. Build Cycle Pattern
For the iterative build-fix loop:
1. Generate code in artifact
2. Instruct user to apply code to specified files
3. Instruct user to run `scripts/build_system.sh`
4. User provides build output
5. Agent analyzes output:
   - If successful: Proceed to next step
   - If failed: Analyze errors, provide fixes, return to step 1
6. Continue cycle until build succeeds

### 4.6. Tool Adaptation Pattern
When encountering a tool reference that is not available:
- **list_files:** Request that user provide directory listing via `ls -la` or `tree`
- **read_file:** Request that user paste file contents
- **grep:** Request that user run `grep` command and provide output
- **set_plan:** Create plan document in artifact for user to save
- **record_user_approval:** Request explicit "APPROVED" response per Section 3.1 of AGENTS.md
- **All other tools:** Request equivalent information or action from user

## 5. Phase-Specific Adaptations

### 5.1. Design Phase
- Generate Architecture Specification in artifact
- Generate Development Plan in artifact
- User saves artifacts to appropriate locations
- User confirms documents are saved before proceeding

### 5.2. Development Phase
- Generate code in artifacts
- User applies code to files
- User runs build scripts and provides output
- Agent analyzes output and iterates
- User confirms changes are committed

### 5.3. Verification Phase
- Generate verification scripts in artifacts
- User applies scripts to appropriate locations
- User runs verification tests and provides output/screenshots
- Agent analyzes evidence
- User confirms checklist updates

### 5.4. Release Phase
- Generate release documentation in artifacts
- User performs deployment actions
- User reports deployment status
- Agent provides guidance based on reported status

## 6. Maintaining Standards

### 6.1. Core Mandates Apply Equally
All mandates from AGENTS.md apply without modification:
- Additive-Only Operations
- Protocol Adherence
- Quality and Completeness
- Formal Approval Protocol
- No assumptions about user intent

### 6.2. Quality is Not Negotiable
Operating in Advisory Mode does not reduce quality requirements:
- Code must still be complete and robust
- Documentation must still be comprehensive
- Architecture must still be thorough
- Testing must still be rigorous

The mechanism of delivery changes; the standard of work does not.

### 6.3. User Collaboration Required
Advisory Mode inherently requires more explicit user collaboration:
- Agent provides expertise and artifacts
- User performs physical actions
- Agent verifies results through user-reported output
- Iteration continues until standards are met

### 6.4. Context Management
Since some agents in Advisory Mode may not persist session state:
- Reference project documentation explicitly
- Ask user for status at session start
- Confirm understanding of current phase
- Request relevant file contents as needed
- Do not assume knowledge from previous sessions

---

## Appendix R - Revision History
| Version | Date       | Author | Changes                               |
|---------|------------|--------|---------------------------------------|
| 0.0.01  | 2025-10-11 | System | Initial creation for Advisory Mode.   |

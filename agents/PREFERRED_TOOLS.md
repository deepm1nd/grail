# Tool Usage Policy

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

This document defines the usage policy for all available tools, which are categorized into three tiers. Failure to adhere to these policies is a critical process failure.

---

## 0. Agent Mode Applicability

**IMPORTANT:** This document defines tools for agents operating in Autonomous Mode with direct execution capabilities.

**For Agents in Advisory Mode:** If these tools are not available, refer to Section 4.6 of `agents/ADVISORY_MODE_GUIDE.md` for adaptation patterns. The same policies (Allowed, Requires Approval, Forbidden) apply conceptually - the agent must simply request equivalent information or actions from the user.

---

## 1. Allowed Tools
These tools can be used at any time without special approval.

- `list_files(path: str = ".")`: Lists files and directories.
- `read_file(filepath: str)`: Reads the content of a file.
- `view_text_website(url: str)`: Fetches the content of a URL as plain text.
- `set_plan(plan: str)`: Sets or updates the working plan.
- `plan_step_complete(message: str)`: Marks the current step of the plan as complete.
- `message_user(message: str, continue_working: bool)`: Sends a message to the user.
- `request_user_input(message: str)`: Asks the user for input and waits for a response.
- `record_user_approval_for_plan()`: Records the user's approval of the plan.
- `read_image_file(filepath: str)`: Displays a local image file.
- `google_search(query: str)`: Performs a Google search.
- `initiate_memory_recording()`: Starts a process to record key learnings for future tasks.
- `grep(pattern: str)`: Searches for a pattern in the codebase.
- `view_image(url: str)`: Displays an image from a URL.
- `run_in_bash_session`: Executes a shell command.
- `create_file_with_block`: Creates a new file with specified content.
- `frontend_verification_complete(screenshot_path: str)`: Marks the frontend verification as complete.
- `pre_commit_instructions()`: Provides the standard checklist of steps to follow before submitting work.
- `frontend_verification_instructions()`: Provides the standard procedure for visually verifying frontend changes.

---

## 2. Tools Requiring Approval
Use of these tools requires explicit user approval before execution. The agent must propose the action and wait for consent.

- `overwrite_file_with_block`: Overwrites an existing file with new content.
- `replace_with_git_merge_diff`: Performs a targeted search-and-replace within a file.
- `rename_file(filepath: str, new_filepath: str)`: Renames or moves a file.
- `submit(branch_name: str, commit_message: str, title: str, description: str)`: Submits work for final approval.

---

## 3. Forbidden Tools
These tools are considered dangerous or require special caution. They are forbidden from general use.

**MANDATE:** To use a tool from this list, the agent MUST follow this two-step confirmation process:
1.  **Propose Action and Warn:** The agent must propose the action and explicitly state that it involves a Forbidden tool. It must then ask the user for confirmation to proceed.
2.  **Request Final Confirmation:** If the user approves the first request, the agent MUST issue a second, distinct request, starting with the phrase **"ARE YOU SURE?"**, to get a final confirmation before executing the tool.

- `request_code_review()`: Requests a review of current code changes.
- `delete_file(filepath: str)`: Deletes a file.
- `reset_all()`: Resets all code changes in the repository.
- `restore_file(filepath: str)`: Restores a single file to its original state.
- `read_pr_comments()`: Reads comments on a pull request.
- `reply_to_pr_comments(replies: str)`: Replies to pull request comments.

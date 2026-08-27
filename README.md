# Agentic Router — portable Windows and Linux x64 guide

Agentic Router runs local AI conversations and supervised workspace changes through a selected **model + harness**. The portable Windows and Linux x64 packages include the application and the .NET runtime; Ollama, local models, and optional harnesses are installed separately from the first-run screen or **Settings → Local resources**. Linux ARM64 and macOS are not supported release targets yet.

> **Alpha software:** review generated changes before committing them. Agentic Router confines Execute actions to the trusted workspace, but models can still make incorrect changes.

## 1. Download and start

1. Open the [latest Agentic Router release](https://github.com/Shakansis/agentic-router-releases/releases).
2. Download the archive for your OS and its matching `.sha256` file.
3. Verify a Windows download in PowerShell:

   ```powershell
   (Get-FileHash .\AgenticRouter-0.9.17_alpha-win-x64.zip -Algorithm SHA256).Hash.ToLowerInvariant()
   Get-Content .\AgenticRouter-0.9.17_alpha-win-x64.zip.sha256
   ```

   The two hashes must match.

   Verify a Linux download with:

   ```bash
   sha256sum -c AgenticRouter-0.9.17_alpha-linux-x64.tar.gz.sha256
   ```

4. Extract the archive to a writable folder. Do not run the application from inside the archive.
5. On Windows, double-click `AgenticRouter.exe`. On Linux x64, run:

   ```bash
   chmod +x AgenticRouter run-agentic-router.sh
   ./run-agentic-router.sh
   ```

6. Keep the terminal open and open the address shown there. Agentic Router uses
   port 5000 when available and automatically selects another local port when
   5000 is occupied.

The application stores configuration and optional local history in the `data` directory beside the executable. Keep that directory when upgrading if you want to preserve them.

## 2. Complete local setup

The first-run screen shows what is available on the computer. A usable local setup needs:

- **Ollama**;
- at least one Ollama model compatible with the available GPU memory;
- a harness only when you want Execute mode. **Codex is recommended for Execute** because it is currently the most stable option, but it is not mandatory.

Use the download or install action beside a missing resource. Agentic Router starts the official installer or harness installation and then checks whether the resource became available.

On Linux, guided installation currently covers Ollama. Install optional harnesses
from their official Linux instructions and place their executables on `PATH`;
Agentic Router then verifies them through the normal discovery path.

On Linux with AMD hardware, Ollama setup requires an explicit acceleration choice:

- **Vulkan** uses the official base package and enables `OLLAMA_VULKAN=1`. It offers broader AMD/Intel coverage but Ollama currently documents it as experimental.
- **ROCm** uses the official base plus ROCm supplemental package. It is intended for supported AMD GPUs and requires compatible ROCm v7 drivers.

Agentic Router never installs GPU drivers or models silently. It reports the requested profile, package manifest, actually observed backend, and CPU fallback separately. A later profile change shows an exact plan, preserves models/data, requires confirmation, and removes only files proven exclusive to the previous supplemental package.

![First-run local setup](screenshots/01-first-run-setup.png)

The same controls remain available later under **Settings → Local resources**. Installed resources display a check mark; unavailable resources retain their install action.

![Installed Ollama, model, and harness resources](screenshots/02-local-resources-ready.png)

## 3. Select a trusted workspace

Execute mode can read and change files only inside the active trusted workspace.

1. Open the workspace menu in the left sidebar.
2. Select **Add workspace**.
3. Enter a recognizable name and the full folder path.
4. Save and activate the workspace.

Use a dedicated project folder. Existing files are allowed, but Git makes review and recovery much easier.

## 4. Initialize Git

If the selected folder is not already a repository:

1. Switch to **Execute**.
2. Open the **Git** card in the left sidebar.
3. Select **Initialize Git** and confirm.

![Confirm Git initialization](screenshots/04-initialize-git.png)

Agentic Router creates a local repository on `main`. It does not create a commit, remote, or push anything automatically.

![Git repository ready](screenshots/05-git-ready.png)

## 5. Use Chat with images

Chat answers questions without changing workspace files.

1. Select **Chat**.
2. Choose a model with the **Vision** capability.
3. Use the image button to attach JPEG, PNG, WebP, or GIF files.
4. Enter a prompt and send it.

Example:

```text
With the images create a webpage about my RPG group.
```

![Four campaign images attached in Chat](screenshots/06-chat-with-images.png)

The model analyzes the attached images and returns its proposal in the conversation.

![Chat response based on the attached images](screenshots/07-chat-response.png)

## 6. Create files with Execute

Execute is for supervised project work.

1. Place any source assets that the result must reference inside the trusted workspace. In this example the four images are under `assets/`.
2. Select **Execute**.
3. Choose the model, harness, and approval policy. Codex is recommended; Native is also available when the selected model supports Agentic Router's structured execution protocol.
4. Send the objective. For this example, use the same prompt:

   ```text
   With the images create a webpage about my RPG group.
   ```

Agentic Router records planned actions, applies the selected approval policy, verifies file effects, and reports the authoritative result.

![Execute created and verified index.html](screenshots/08-execute-result.png)

### Supervise a larger local task

Prefix an Execute objective with `/supervisor` to enable durable supervised execution.
The same selected local model and harness work serially in focused supervisor and
worker contexts. The supervisor checks current Host-observed files and validation
facts, rejects incomplete work with a precise correction, and accepts completion only
after the criteria are covered.

When local history is enabled, closing or reloading the browser does not cancel the
run. Resume the saved conversation to reattach and replay progress. After an
application restart, the `manual` policy waits for an explicit Resume; `auto-safe`
continues only when the route, workspace, committed actions, approvals, and recovery
budget are unambiguous. Supervised runs accept local Ollama routes only and never use a
cloud fallback.

## 7. Review and open the result

Open the **Git** card to inspect changed files and the generated diff before committing.

![Reviewing index.html in the Git panel](screenshots/09-review-files.png)

For this example, opening `index.html` displays the generated campaign page:

![Generated Khitai campaign website](screenshots/10-generated-website.png)

Use **View folder** beside **Commit** and **Push** to open the current workspace in Windows Explorer or the Linux desktop file manager.

![Workspace opened in Windows Explorer](screenshots/11-view-folder.png)

## 8. Stop, update, or move the portable app

- Stop Agentic Router with `Ctrl+C` in its terminal or by closing that window.
- Before replacing an alpha build, back up the adjacent `data` directory.
- Extract a new version to a clean folder, then copy the previous `data` directory only if you want to retain its settings and optional history.
- Do not copy `data` to another computer unless you intend to transfer its local configuration.

## Troubleshooting

- **The welcome screen remains visible:** confirm that Ollama is running and at least one model is installed, then refresh the resource check.
- **A harness is missing:** open **Settings → Local resources** and use its install action. A harness can be selected in Chat, but it is applied only when Execute runs.
- **Images are not analyzed:** select a model showing the **Vision** capability before sending.
- **Execute cannot change files:** confirm the correct trusted workspace is active and review the selected approval policy.
- **Linux cloud keys are unavailable:** install `libsecret-tools` and ensure the desktop user keyring is active.
- **Linux folder picker is unavailable:** install `zenity` or `kdialog`, or enter the workspace path manually. `xdg-utils` is required for **View folder**.
- **The page does not open:** use the exact local address printed in the terminal and keep Agentic Router running.

Use of this alpha build is governed by the [Agentic Router Alpha Evaluation License](LICENSE.md).

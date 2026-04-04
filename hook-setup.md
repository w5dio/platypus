# Constitution Hook Setup

Create a Claude Code SessionStart hook that fetches the platform constitution at session start, according to the following instructions:

1. Create `.claude/hooks/fetch-constitution.sh` with this exact content:

   ```bash
   #!/bin/bash
   set -euo pipefail

   CONSTITUTION_URL="https://raw.githubusercontent.com/w5dio/constitution/refs/heads/main/README.md"

   content=$(curl -fsSL "$CONSTITUTION_URL" 2>/dev/null) || {
     echo "Warning: Could not fetch constitution principles from w5dio/constitution." >&2
     exit 0
   }

   echo "=== PLATFORM CONSTITUTION (START) ==="
   echo "Source: $CONSTITUTION_URL"
   echo "Injected by: SessionStart hook (.claude/hooks/fetch-constitution.sh)"
   echo ""
   echo "$content"
   echo ""
   echo "=== PLATFORM CONSTITUTION (END) ==="
   ```

2. Make the script executable (`chmod +x`).

3. Create or update `.claude/settings.json` to register the hook:

   ```json
   {
     "hooks": {
       "SessionStart": [
         {
           "matcher": "",
           "hooks": [
             {
               "type": "command",
               "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/fetch-constitution.sh",
               "timeout": 5,
               "statusMessage": "Loading constitution principles..."
             }
           ]
         }
       ]
     }
   }
   ```

   If `settings.json` already exists with other content, merge the `hooks.SessionStart` key into it rather than overwriting.

4. Create or update `.claude/CLAUDE.md` to include this section:

   ```markdown
   ## Platform Constitution

   The platform constitution is injected at session start via a SessionStart hook. Its principles govern all development in this repository and must be applied to every implementation, architectural, and configuration decision.
   ```

   If `CLAUDE.md` already exists, append the section rather than overwriting it.

<template>
  <div>
    <div class="card">
      <h2>🔄 Migration Checklist: VM → Raspberry Pi</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Step-by-step migration from yorks-vm (x64 Manjaro) to Raspberry Pi 5 (ARM64).
        Every step has a verification check. Do not proceed to the next step until verification passes.
      </p>
    </div>

    <div class="card" v-for="phase in phases" :key="phase.label">
      <h3>{{ phase.icon }} Phase {{ phase.number }}: {{ phase.label }}</h3>
      <p v-if="phase.desc" style="color: #8b949e; margin-bottom: 0.75rem; font-size: 0.85rem;">{{ phase.desc }}</p>
      <table class="checklist">
        <thead>
          <tr>
            <th style="width: 2rem;">✓</th>
            <th>Step</th>
            <th>Details</th>
            <th>Verify</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="step in phase.steps" :key="step.step">
            <td><input type="checkbox" /></td>
            <td class="step-name">{{ step.step }}</td>
            <td class="step-details" v-html="step.details"></td>
            <td class="step-verify">{{ step.verify }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="card">
      <h3>📋 Inventory Reference</h3>
      <table class="ref-table">
        <thead>
          <tr><th>Category</th><th>Items</th></tr>
        </thead>
        <tbody>
          <tr v-for="item in inventory" :key="item.category">
            <td><strong>{{ item.category }}</strong></td>
            <td v-html="item.items"></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>

const phases = [
  {
    number: 1,
    icon: '💿',
    label: 'OS + Base Setup',
    desc: 'Get the Pi booted with a working OS, user account, and network access.',
    steps: [
      { step: 'Choose OS', details: '<strong>Raspberry Pi OS Lite (64-bit, Bookworm)</strong> recommended. Headless — no desktop needed. Alternatively, Ubuntu Server 24.04 ARM64 works fine.<br><br>The SD card that ships with the Pi may have a desktop OS. Reflash it with Lite using Raspberry Pi Imager — less overhead, more RAM for agents.', verify: 'cat /etc/os-release shows expected OS' },
      { step: 'Flash SD card', details: 'Use <code>Raspberry Pi Imager</code> on the VM or another machine. In settings (gear icon): enable SSH, set username <code>york</code>, set password, configure WiFi if needed, set hostname (e.g., <code>yorks-pi</code>).', verify: 'Pi boots, you can SSH in: ssh york@yorks-pi.local' },
      { step: 'First boot + updates', details: '<code>sudo apt update && sudo apt upgrade -y</code><br>Then: <code>sudo apt install -y git curl build-essential sqlite3 python3 python3-pip</code>', verify: 'git --version, sqlite3 --version, python3 --version all return' },
      { step: 'Set timezone', details: '<code>sudo timedatectl set-timezone America/New_York</code>', verify: 'timedatectl shows America/New_York' },
      { step: 'Enable lingering', details: '<code>sudo loginctl enable-linger york</code><br>Required for systemd user services to run without an active login session.', verify: 'ls /var/lib/systemd/linger/york exists' },
    ],
  },
  {
    number: 2,
    icon: '📦',
    label: 'Runtime + Tools',
    desc: 'Install Node.js, npm, OpenClaw, mcporter, and Python tooling.',
    steps: [
      { step: 'Install Node.js', details: 'Use <strong>nvm</strong> or <strong>fnm</strong> for version management:<br><code>curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash</code><br><code>nvm install 22</code> (or latest LTS — check ARM64 support)<br><br><strong>Note:</strong> Current VM runs Node 25.4.0. Use the latest LTS that supports ARM64. Node 22+ should work.', verify: 'node --version returns v22+ and arch is arm64: node -p process.arch' },
      { step: 'Set up npm global dir', details: '<code>mkdir -p ~/.npm-global</code><br><code>npm config set prefix ~/.npm-global</code><br>Add to ~/.bashrc or ~/.zshrc: <code>export PATH="$HOME/.npm-global/bin:$PATH"</code>', verify: 'npm config get prefix shows ~/.npm-global' },
      { step: 'Install OpenClaw', details: '<code>npm install -g openclaw</code>', verify: 'openclaw --version returns current version' },
      { step: 'Install mcporter', details: '<code>npm install -g mcporter</code>', verify: 'mcporter --version returns 0.7+' },
      { step: 'Install uv (Python)', details: '<code>curl -LsSf https://astral.sh/uv/install.sh | sh</code><br>Needed for google-calendar and google-sheets MCP servers.', verify: 'uv --version, uvx --version both return' },
    ],
  },
  {
    number: 3,
    icon: '🔑',
    label: 'Secrets + Auth',
    desc: 'Copy API keys and auth tokens. Do this BEFORE copying config — config references these files.',
    steps: [
      { step: 'Create config dir', details: '<code>mkdir -p ~/.config/york && chmod 700 ~/.config/york</code>', verify: 'ls -la ~/.config/york/ shows directory with 700 perms' },
      { step: 'Copy API keys', details: 'From VM: <code>scp ~/.config/york/* york@yorks-pi:~/.config/york/</code><br><br>Files: <code>anthropic-api-key, discord.json, gemini-api-key, google-oauth-client.json, google-oauth-token.json, google-service-account.json, xai-api-key</code>', verify: 'ls ~/.config/york/ shows all 7 files, chmod 600 on each' },
      { step: 'Set file permissions', details: '<code>chmod 600 ~/.config/york/*</code>', verify: 'stat -c "%a" ~/.config/york/* shows 600 for all' },
    ],
  },
  {
    number: 4,
    icon: '📂',
    label: 'Workspaces + Data',
    desc: 'Copy the entire OpenClaw directory, git repos, and the Obsidian vault. This is the bulk of the migration.',
    steps: [
      { step: 'Copy ~/.openclaw/', details: 'From VM: <code>rsync -avz ~/.openclaw/ york@yorks-pi:~/.openclaw/</code><br><br>This copies: all workspaces, shared-scripts, shared-cache, gen/, media/, openclaw.json config, and all skills.', verify: 'ls ~/.openclaw/workspace-* shows all 8 workspaces. cat ~/.openclaw/openclaw.json is valid JSON.' },
      { step: 'Copy git repos', details: '<code>rsync -avz ~/york/ york@yorks-pi:~/york/</code><br><code>rsync -avz ~/york-data/ york@yorks-pi:~/york-data/</code><br><br>These contain MCP servers (york-tools, google-tasks, york-data) and scripts.', verify: 'cd ~/york-data && git status is clean. cd ~/york/york-tools && node -e "console.log(1)" works.' },
      { step: 'Backup york-data DB', details: 'Before copy: <code>sqlite3 ~/york-data/york-data.db ".backup ~/york-data/backups/pre-migration.db"</code><br>Then verify the copy on Pi: <code>sqlite3 ~/york-data/york-data.db "SELECT count(*) FROM consumption;"</code>', verify: 'Row count on Pi matches row count on VM. Run both and compare.' },
      { step: 'Copy Obsidian vault', details: '<code>rsync -avz ~/work-notes/ york@yorks-pi:~/work-notes/</code><br>Or: set up Syncthing on Pi to sync from the same source (preferred — live sync continues working).', verify: 'ls ~/work-notes/work/board.md exists' },
      { step: 'Install npm deps for MCP servers', details: 'For each MCP server, run npm install:<br><code>cd ~/york/york-tools && npm install</code><br><code>cd ~/york/google-tasks && npm install</code><br><code>cd ~/york-data && npm install</code><br><br><strong>Important:</strong> These have native deps that are architecture-specific. You MUST run npm install on the Pi, not just copy node_modules.', verify: 'mcporter call york-data.ping returns success. mcporter call york-tools.image_models returns model list.' },
    ],
  },
  {
    number: 5,
    icon: '⚙️',
    label: 'mcporter Config',
    desc: 'Copy and verify MCP server registration.',
    steps: [
      { step: 'Copy mcporter config', details: '<code>mkdir -p ~/.mcporter</code><br><code>scp ~/.mcporter/mcporter.json york@yorks-pi:~/.mcporter/</code>', verify: 'mcporter list shows all 6 servers' },
      { step: 'Update absolute paths', details: 'If the username or home dir differs, update paths in <code>~/.mcporter/mcporter.json</code>. All paths currently use <code>/home/york/</code>.', verify: 'grep -c "/home/york/" ~/.mcporter/mcporter.json — all paths valid on Pi' },
      { step: 'Test each MCP server', details: 'Test every server individually:<br><code>mcporter call york-data.ping</code><br><code>mcporter call york-tools.image_models</code><br><code>mcporter call google-tasks.list_task_lists</code><br><code>mcporter call google-calendar (check schema)</code><br><code>mcporter call google-sheets (check schema)</code><br><code>mcporter call dice-roller (check schema)</code>', verify: 'All 6 return valid responses without errors' },
    ],
  },
  {
    number: 6,
    icon: '🔧',
    label: 'OpenClaw Config',
    desc: 'Validate and adjust the OpenClaw config for the new host.',
    steps: [
      { step: 'Update gateway bind address', details: 'In <code>openclaw.json</code>, update <code>gateway.controlUi.allowedOrigins</code> with the Pi\'s LAN IP.<br>Also update <code>gateway.bind</code> if needed.', verify: 'openclaw doctor --non-interactive passes' },
      { step: 'Run openclaw doctor', details: '<code>openclaw doctor --non-interactive</code><br>This validates config, checks API keys, verifies model access.', verify: 'No errors in doctor output' },
      { step: 'Update systemd service', details: 'The gateway service file has hardcoded paths and version strings. Run:<br><code>openclaw gateway install</code> (or manually update the service file)', verify: 'systemctl --user cat openclaw-gateway.service shows correct paths' },
      { step: 'Start gateway', details: '<code>systemctl --user start openclaw-gateway</code><br><code>systemctl --user enable openclaw-gateway</code>', verify: 'systemctl --user status openclaw-gateway shows active (running)' },
      { step: 'Test Discord connection', details: 'Send a message in #🦊york. York should respond.', verify: 'Response appears in Discord within 30s' },
    ],
  },
  {
    number: 7,
    icon: '🔗',
    label: 'Systemd Services',
    desc: 'Set up user services that run alongside OpenClaw.',
    steps: [
      { step: 'Vault watcher', details: 'Copy service file:<br><code>cp ~/.config/systemd/user/vault-watcher.service</code> (already in ~/.openclaw dir from rsync)<br>Or create from template — see inventory below.<br><code>systemctl --user daemon-reload</code><br><code>systemctl --user enable --now vault-watcher</code>', verify: 'systemctl --user status vault-watcher shows active. Edit a Daily note and check if vault-update fires within 2 min.' },
      { step: 'Syncthing (if using for vault)', details: '<code>sudo apt install syncthing</code><br><code>systemctl --user enable --now syncthing</code><br>Configure via web UI at http://yorks-pi:8384 — add the Obsidian vault folder.', verify: 'syncthing is active, vault folder shows "Up to Date"' },
      { step: 'York dashboard (optional)', details: 'Static file server for panel. Only needed if you want the web dashboard on the Pi.<br><code>cp ~/.config/systemd/user/york-dashboard.service</code><br><code>systemctl --user enable --now york-dashboard</code>', verify: 'curl http://localhost:8080 returns content' },
    ],
  },
  {
    number: 8,
    icon: '✅',
    label: 'Full Verification',
    desc: 'End-to-end checks across all agents and crons. Do not shut down the VM until ALL checks pass.',
    steps: [
      { step: 'Test each agent', details: 'Send a message in each Discord channel and verify the correct agent responds:<br>#🦊york → York<br>#🦫offa → Offa<br>#🦌wynn → Wynn<br>#🐉caedmon → Caedmon<br>#🐓dagr → Dagr<br>#🐻wiglaf → Wiglaf<br>#🦡hild → Hild<br>#🐺guthlac → Guthlac', verify: 'All 8 agents respond correctly in their channels' },
      { step: 'Test york-data read/write', details: 'In #🦌wynn, log a test food item. Then verify:<br><code>sqlite3 ~/york-data/york-data.db "SELECT * FROM consumption ORDER BY id DESC LIMIT 1;"</code><br>Delete the test entry after.', verify: 'Entry appears in DB with correct timestamp' },
      { step: 'Test image generation', details: 'In #🐺guthlac or #🐉caedmon, ask for an image. Verify it generates and displays in Discord.', verify: 'Image appears as Discord attachment' },
      { step: 'Test vault watcher', details: 'Edit a Daily note in Obsidian. Watch #🐻wiglaf for vault-update dispatch within 2 minutes.', verify: 'Wiglaf processes the note change' },
      { step: 'Test cron (manual trigger)', details: 'Trigger a cron manually: <code>openclaw cron run &lt;job-id&gt;</code><br>Try the weather cache or a memory audit.', verify: 'Cron executes and completes without timeout' },
      { step: 'Check DB backup cron', details: 'Wait for midnight or trigger manually. Check <code>~/york-data/backups/</code> for new backup file.', verify: 'Backup file exists with today\'s date and is non-empty' },
      { step: 'Verify Google OAuth', details: 'Test Google Tasks and Calendar MCP servers — these use OAuth tokens that may need re-auth on a new machine.', verify: 'mcporter call google-tasks.list_task_lists returns task lists' },
      { step: 'Run for 24 hours', details: 'Leave the Pi running for a full day. Monitor:<br>- Morning crons (4-8 AM) all fire<br>- Morning brief posts to #🦊york<br>- Memory audits complete overnight<br>- No OOM kills (check <code>dmesg | grep -i oom</code>)', verify: 'All crons ran. No errors in openclaw logs. dmesg clean.' },
    ],
  },
  {
    number: 9,
    icon: '🔒',
    label: 'Cutover + Cleanup',
    desc: 'Once the Pi is verified, shut down the VM to prevent dual-agent conflicts.',
    steps: [
      { step: 'Stop VM gateway', details: 'On VM: <code>systemctl --user stop openclaw-gateway</code><br><code>systemctl --user disable openclaw-gateway</code><br><br><strong>Critical:</strong> Two gateways with the same Discord token = duplicate responses. Stop the VM first.', verify: 'VM gateway is stopped. Only Pi is responding to Discord.' },
      { step: 'Final DB sync', details: 'One last rsync of york-data.db from VM to Pi (in case any data was written between migration and cutover):<br><code>scp vm:~/york-data/york-data.db ~/york-data/york-data.db</code>', verify: 'Row counts match between VM backup and Pi DB' },
      { step: 'Update DNS / bookmarks', details: 'Update any bookmarks or scripts that reference the VM IP to point to the Pi.', verify: 'Dashboard accessible at new IP' },
      { step: 'Keep VM as backup', details: 'Don\'t delete the VM yet. Keep it available for 1-2 weeks in case you need to fall back.<br>Consider keeping a final york-data backup on the VM.', verify: 'VM is stopped but not destroyed' },
    ],
  },
]

const inventory = [
  { category: 'Workspaces (8)', items: 'workspace (York), workspace-offa, workspace-wynn, workspace-caedmon, workspace-dagr, workspace-wiglaf, workspace-hild, workspace-guthlac' },
  { category: 'Shared directories', items: 'shared-scripts/ (weather-cache.sh, workout-cache.sh, calendar-cache.sh), shared-cache/ (daily cache files), media/ (generated images), gen/ (legacy, can skip)' },
  { category: 'MCP servers (6)', items: 'york-data (~/york-data/), york-tools (~/york/york-tools/), google-tasks (~/york/google-tasks/), google-calendar (~/york/scripts/calendar-mcp/), google-sheets (uvx, no local install), dice-roller (~/york/scripts/dice-roller/)' },
  { category: 'API keys (7 files)', items: 'anthropic-api-key, discord.json, gemini-api-key, google-oauth-client.json, google-oauth-token.json, google-service-account.json, xai-api-key' },
  { category: 'Databases', items: 'york-data.db (~/york-data/ — <strong>critical, backup first</strong>). No other production DBs.' },
  { category: 'Systemd services', items: 'openclaw-gateway.service (OpenClaw manages), vault-watcher.service (custom), york-dashboard.service (optional), syncthing.service (if using for vault sync)' },
  { category: 'Git repos', items: '~/york/ (monorepo: york-tools, google-tasks, scripts, builds), ~/york-data/ (MCP server + DB)' },
  { category: 'Obsidian vault', items: '~/work-notes/work/ (Wiglaf\'s vault, syncs via Syncthing/Obsidian Sync)' },
  { category: 'Config files', items: '~/.openclaw/openclaw.json (main config), ~/.mcporter/mcporter.json (MCP server registry)' },
  { category: 'Architecture', items: 'Current host: x64 (Manjaro). Target: ARM64 (RPi OS Lite / Ubuntu Server). Key difference: npm install must be re-run for native modules. Node 22+ LTS recommended for ARM64 stability.' },
]
</script>

<style scoped>
.checklist {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.85rem;
}

.checklist th {
  text-align: left;
  padding: 0.5rem;
  border-bottom: 2px solid #30363d;
  color: #8b949e;
  font-size: 0.75rem;
  text-transform: uppercase;
}

.checklist td {
  padding: 0.75rem 0.5rem;
  border-bottom: 1px solid #21262d;
  vertical-align: top;
}

.checklist input[type="checkbox"] {
  width: 1.2rem;
  height: 1.2rem;
  cursor: pointer;
}

.step-name {
  font-weight: 600;
  white-space: nowrap;
  min-width: 10rem;
}

.step-details {
  color: #c9d1d9;
  line-height: 1.5;
}

.step-details code {
  background: #161b22;
  padding: 0.15rem 0.4rem;
  border-radius: 3px;
  font-size: 0.8rem;
  color: #79c0ff;
}

.step-verify {
  color: #3fb950;
  font-size: 0.8rem;
  min-width: 12rem;
}

.ref-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.85rem;
}

.ref-table th {
  text-align: left;
  padding: 0.5rem;
  border-bottom: 2px solid #30363d;
  color: #8b949e;
}

.ref-table td {
  padding: 0.5rem;
  border-bottom: 1px solid #21262d;
  vertical-align: top;
}
</style>

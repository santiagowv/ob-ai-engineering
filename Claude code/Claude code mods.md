A mod is a <span style="color:rgb(216, 203, 251)">plugin that changes how Claude Code looks and behaves</span>. It's made of JavaScript or TypeScript event handlers.
# What a mod can do
- <span style="color:rgb(216, 203, 251)">Draw an interface</span> you can use.
- <span style="color:rgb(216, 203, 251)">Replace or restyle</span> parts Claude Code draws itself.
- <span style="color:rgb(216, 203, 251)">Step into a tool call</span> or a request.
- <span style="color:rgb(216, 203, 251)">Run your own code</span> on a command.
- <span style="color:rgb(216, 203, 251)">Share data</span> between hooks.
# What a mod can reach
A mod runs <span style="color:rgb(216, 203, 251)">with our permissions</span>, so before we install one, we need to know what it has access to. A mod can:
- **Act on our machine as you:** <span style="color:rgb(216, 203, 251)">read and write files</span> anywhere your user account can. 
- **Read our secrets:** <span style="color:rgb(216, 203, 251)">envionment variables</span> and settings files.
- **See your session:** <span style="color:rgb(216, 203, 251)">every prompt you send</span> and every tool call Claude makes.
- **Act without asking you:** <span style="color:rgb(216, 203, 251)">approve a tool call</span> before we're asked.
- **Spend our usage:** <span style="color:rgb(216, 203, 251)">call a model on our plan</span> or API key.
# Validate what a mod does
```bash
claude plugin validate ./some-mod
```
# Turn mods on or off
<span style="color:rgb(216, 203, 251)">Mods are on by default</span>. In the terminal, we use Claude Code v2.1.287 or later.
- **One mod:** disable or uninstall its plugin from the **installed tab in** `/plugin`.
- **Every installed mod, for one session:** start Claude Code with `--safe-mode`, which also disables your other customizations.
- **Every mod you installed, in every session:** set [`"disableAllHooks": true`](https://code.claude.com/docs/en/settings-reference#disableallhooks) in `~/.claude/settings.json`.
# How a mod works
<span style="color:rgb(216, 203, 251)">Claude Code runs a hook when its event happens</span>, such as when Claude calls a tool or when the spineer is drawn. A small mod has three files:
```
first-mod/
├── .claude-plugin/
│   └── plugin.json
└── hooks/
    ├── hooks.json
    └── register.js
```
- **plugins.json:** the plugin's <span style="color:rgb(216, 203, 251)">manifest</span>.
- **hooks.json:** points to your code file.
- **register.js:** our code, called the hooks module. It <span style="color:rgb(216, 203, 251)">tells Claude Code which events to run the function on</span>.
## register.js
```javascript
// The count, shared by the two hooks below
let calls = 0

// Claude Code calls this once when the mod loads
export function register(on) {
  // Runs each time Claude is about to use a tool
  on('tool.call', async ($, e, next) => {
    calls += 1
    // Ask Claude Code to draw the interface again, so the new count shows
    $.ui.invalidate('ui.render')
    // Let the tool run as usual
    return next(e)
  })

  // Runs each time Claude Code draws the spinner
  on('ui.render', { component: 'Spinner' }, async ($, e, next) => {
    // Keep Claude Code's spinner, with the count added after its word
    return next({ ...e, props: { ...e.props, suffix: ' · tool calls: ' + calls + '…' } })
  })
}
```
Each function we pass to `on` is a hook, which is an event handler. <span style="color:rgb(216, 203, 251)">Claude code passes every hook the same three arguments</span>.
- **The mods API:** named `$`: <span style="color:rgb(216, 203, 251)">every method a mod can call to reach outside itself</span>, in namespaces such as `$.ui` and `$.command`.
- **The event:** named `e`: <span style="color:rgb(216, 203, 251)">the event's input</span> as plain data, such as tool call's name and arguments.
- **The next hnadler:** named `next`: a function that <span style="color:rgb(216, 203, 251)">passes the event on to the other mods and then to Claude Code's</span> own behavior, and returns a result.
# What a hook can do with an event
Claude Code <span style="color:rgb(216, 203, 251)">runs our hook before it acts on the event</span>, the hook decides what happens next. It can:
- **Observe:** <span style="color:rgb(216, 203, 251)">note what's happening</span> and let it continue unchanged.
- **Rewrite:** <span style="color:rgb(216, 203, 251)">change the event</span> before it continues.
- **Answer:** <span style="color:rgb(216, 203, 251)">handle the event itself, so the usual behavior doesn't run</span>, such as refusing a command.
# Create a mod
Claude works from a built-in skill named `plugin-authoring`, which tells it <span style="color:rgb(216, 203, 251)">where to write the mod</span>, which <span style="color:rgb(216, 203, 251)">events and methods our version has</span>, and <span style="color:rgb(216, 203, 251)">how the mod gets loaded</span>.
## Describe the mod
Ask for the mod in our own words. <span style="color:rgb(216, 203, 251)">Claude writes the mod in a directory of its own in the session's mods folder</span>.
## Approve the mod
When Claude saves the first file, <span style="color:rgb(216, 203, 251)">Claude Code asks whether to enable hot reloading for the session</span>.
## Check that the mod loaded
Run `/plugin`at the Claude Code prompt and <span style="color:rgb(216, 203, 251)">press tab until the installed tab is selected</span>.
## Try the mod
Use for what we asked for.
## Use the mod in other sessions
A mod Claude wrote loads only in the session that made it. <span style="color:rgb(216, 203, 251)">To keep the mod, copy its directory out of the mods folder to a place of our own</span>. Then choose how to load it:
- **In a session we start:** in our shell, run `claude --plugin-dir ~/mods/git-branch`.
- **For other people:** add it to a marketplace so they can install it.
- To load it in every session, including the desktop app.
```
{
  "env": {
    "CLAUDE_CODE_PLUGIN_DIRS": "/Users/santiagowilches/Movies/campbi/projects/esta nueva funcionalidad de claude me volvio mas eficiente como ingeniero de datos/assets/data pipeline safety guard/data-pipeline-safety-guard"
  }
}
```
# Test a mod
We can <span style="color:rgb(216, 203, 251)">write automated tests for a mod and run them</span> from our shell with `claude plugin test`. A test:
1. <span style="color:rgb(216, 203, 251)">Loads the mod</span>.
2. <span style="color:rgb(216, 203, 251)">Sends events</span> through its hooks the way Claude Code would.
3. <span style="color:rgb(216, 203, 251)">Checks what the hooks did</span>, without a session, a sign-in, or a network.
# Mod vs MCP vs Skill vs Hook
## Mod
- We want a <span style="color:rgb(216, 203, 251)">pane</span>, a <span style="color:rgb(216, 203, 251)">band above the prompt</span>, a <span style="color:rgb(216, 203, 251)">custom command</span>, or to <span style="color:rgb(216, 203, 251)">rewrite an event</span>.
## Settings hook
- Want to <span style="color:rgb(216, 203, 251)">block, allow, or log an event</span> with a script we already have.
## Skill
- We <span style="color:rgb(216, 203, 251)">keep pasting the same instructions</span> into chat.
## MCP Server
- Claude <span style="color:rgb(216, 203, 251)">needs to reach an external system</span>.
# Capacitor Google Auth — Agent Skill

An [Agent Skill](https://agentskills.io) that helps AI coding assistants implement native Google Sign-In in Capacitor mobile apps.

## The Problem

Google blocks OAuth redirect flows inside WKWebView (which Capacitor uses on iOS). If you try the standard web OAuth flow in a native Capacitor app, Google returns a `disallowed_useragent` error. No workaround exists — Google uses JS-based detection too.

## The Solution

This skill teaches your AI assistant to implement a **platform-aware split**:
- **Web**: Standard OAuth redirect flow
- **Native (iOS/Android)**: Native Google Sign-In SDK → ID token → backend `signInWithIdToken()`

It covers the full setup: plugin installation, platform detection, token exchange, Google Cloud Console config, Info.plist setup, and all the dead ends to avoid.

## Works With

- **AI Tools**: Claude Code, Codex CLI, ChatGPT, any tool supporting the [Agent Skills](https://agentskills.io) standard
- **Backends**: Supabase, Firebase, or any backend that accepts Google ID tokens
- **Frameworks**: Next.js, React, Vue, Angular, or any web framework running in Capacitor
- **Plugin**: `@capgo/capacitor-social-login` (actively maintained, version tracks Capacitor version)

## Install

### Claude Code

Copy the skill to your personal skills directory:

```bash
mkdir -p ~/.claude/skills/capacitor-google-auth
cp SKILL.md references/ ~/.claude/skills/capacitor-google-auth/ -r
```

Or just clone this repo into your skills folder:

```bash
git clone https://github.com/sukibk/capacitor-google-auth-skill.git ~/.claude/skills/capacitor-google-auth
```

### Other AI Tools

Place the `SKILL.md` and `references/` directory wherever your tool looks for agent skills.

## Usage

The skill triggers automatically when you mention Capacitor + Google auth, `disallowed_useragent` errors, `@capgo/capacitor-social-login`, or native social login in a Capacitor context.

You can also invoke it directly:

```
/capacitor-google-auth
```

## File Structure

```
├── SKILL.md                    # Core instructions (loaded into AI context)
└── references/
    └── implementation.md       # Detailed code examples, config values, gotchas
```

## License

MIT

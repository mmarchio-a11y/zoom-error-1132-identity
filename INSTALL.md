zoom-error-1132-identity — export package
=========================================

What this fixes
---------------
Zoom's "Unable to sign in (Error code: 1132)" on macOS. The error is a ban keyed to the
Mac's hardware identity (IOPlatformUUID + IOPlatformSerialNumber), not to any local file
or Keychain item, so deleting data and clean reinstalling never clears it. This skill
substitutes a different, fixed identity for Zoom processes only, so the app signs in
normally, shows the avatar and can host meetings.

Contents
--------
zoom-error-1132-identity/
  SKILL.md                     loadable skill: triggers, procedure, pitfalls, verification
  references/evidence.md       the forensic basis (binary strings, sandbox profile, symptom table)
  scripts/fix_zoom_1132_root.sh    one-shot install + patch (runs as root, one admin prompt)
  scripts/reapply_after_update.sh  re-apply after a manual Zoom update
  scripts/wipe_local_zoom_state.py remove all local Zoom state, with a manifest
  scripts/build_shim.sh            build the shim and launcher stub
  scripts/zoomidspoof.c            the IOKit interposer source
  scripts/launcher.c               the bundle launcher stub source

Install into Hermes
-------------------
Copy the folder into a skills directory, keeping the category level:

    cp -R zoom-error-1132-identity ~/.hermes/profiles/<profile>/skills/productivity/

or, for a shared/default profile:

    cp -R zoom-error-1132-identity ~/.hermes/skills/productivity/

Restart the session (or start a new one) and the skill appears in the index. Load it with:

    skill_view(name="zoom-error-1132-identity")

Use it without Hermes
---------------------
The scripts are standalone. A full run is:

    python3 zoom-error-1132-identity/scripts/wipe_local_zoom_state.py     # Zoom quit first
    osascript -e 'do shell script "/bin/bash '"$PWD"'/zoom-error-1132-identity/scripts/fix_zoom_1132_root.sh" with administrator privileges'

Requirements: macOS, Apple Silicon (Intel works, the arm64 query parameter is skipped),
Xcode Command Line Tools for clang, and an administrator password entered at the GUI
prompt.

Before running, tell the user that this will:
  - delete all local Zoom state (recordings in ~/Documents/Zoom are not touched),
  - replace the Zoom bundle with a re-signed copy,
  - disable Zoom's automatic updater,
  - reset camera, microphone and screen-recording permissions.

Verification is the email step on the sign-in screen advancing to password/passkey
instead of showing 1132, plus the avatar rendering and New meeting opening.

Version 1.0.0. Author: Michelangelo Marchiorello (mmarchio-a11y), Hermes Agent. MIT.
# My Fire TV Stick: plan

The plan for cleaning up my own stick, for whichever Claude Code session runs it. That session runs on a computer on
the same Wi-Fi as the stick, works from a clone of this repo, and follows [PROMPT.md](PROMPT.md)'s safety rules
(disable, never uninstall; back up first; small batches with screenshots; close ADB at the end). This file only adds
what is specific to my setup.

## My setup

| | |
|---|---|
| Device | Fire TV Stick 4K or 4K Max, year not known yet: read it with `tools/tv.py --probe` |
| Developer options | Already on. ADB debugging must be on too (Settings → My Fire TV → Developer options) |
| Camera | A third-party camera (not Ring or Blink), linked to my Amazon account through the Alexa app |
| Lights | Smart bulbs or plugs linked through Alexa |
| Goal | Faster, no ads or sponsored rows, every app I use still working, cameras and lights one press away |

## What Fire OS allows

- **Debloating works.** Amazon's telemetry, ad feeders, shopping, ACR and tutorials can all be disabled and restored.
- **Amazon's home can't be replaced.** `com.amazon.tv.launcher` is protected, so the Home button always opens it
  (`docs/TROUBLESHOOTING.md`, "Fire TV (Fire OS)"). This repo's `launcher/` does not apply. The hero banner and the
  sponsored row are drawn by Amazon's launcher itself, so no list removes them completely.
- **Alexa has to stay.** The camera feed and the lights both go through Alexa, so voice search and the smart-home
  parts of Alexa stay enabled.

## Step 1: connect and pick the base list

```bash
python3 -m venv tools/venv && tools/venv/bin/pip install -r tools/requirements.txt
TV_HOST=<stick ip> tools/tv.py --probe
```

Pick the base list that matches the model and Fire OS version, and copy it to `work/my-list.txt` (git-ignored):

| Probe says | Base list |
|---|---|
| AFTMM (4K 1st gen 2018), Fire OS 6 | `examples/amazon-fire-tv-stick-4k-2018/debloat.txt` |
| AFTKA (4K Max 2021), Fire OS 7 | `examples/amazon-fire-tv-stick-4k-max-2021/debloat.txt` |
| AFTKM (4K 2nd gen 2023) or 4K Max 2023, Fire OS 8 | `examples/amazon-fire-tv-stick-4k-2023/debloat.txt` |

## Step 2: keep the camera and smart-home packages

Some base lists disable parts of Alexa that my camera and lights need. Comment these out in `work/my-list.txt`
(prefix with `# `) wherever they appear. Check each one with `dumpsys package <name>` before deciding:

| Package | Why it stays |
|---|---|
| `com.amazon.smarthomemapviewapp` | The smart-home dashboard: camera tiles and light switches on the TV |
| `com.amazon.tv.mattercohost`, `com.amazon.tv.matter` | Matter smart-home control, if any bulb or plug uses Matter |
| `com.amazon.aiondec`, `com.amazon.firetv.androidskillconnectioncapability` | Alexa skill connections. A third-party camera reaches Alexa through its maker's skill |
| `com.amazon.rtcsessioncontroller` | Live audio and video sessions. A live camera view may use it |
| `com.amazon.tv.alexanotifications`, `com.amazon.tv.alexaalerts` | Doorbell and motion pop-ups, timers |

Also keep everything the base list's header names as kept on purpose (voice search, Appstore, keyboard, remote and
Bluetooth, OTA updates, Prime Video).

## Step 3: ask me before disabling apps I might use

Before applying the list, list the installed apps (`pm list packages -3` plus the Amazon apps in the list) and ask me
in one checklist which streaming apps and Amazon extras (Photos, Music, Kids, Luna, Silk, casting) to keep. Disable
only what I don't tick.

## Step 4: apply in batches

```bash
TV_HOST=<stick ip> tools/disable.sh work/my-list.txt
TV_HOST=<stick ip> tools/tv.py --screenshot shots/01.png
```

Apply in three or four batches, taking a screenshot and checking Home after each one. Reboot once at the end and
confirm that everything stayed disabled. Undo:
`TV_HOST=<stick ip> tools/restore.sh work/my-list.txt com.amazon.tv.launcher`.

After each batch, and again after the reboot, check:

- Hold the mic button and say "Alexa, show <camera name>": the live feed opens.
- "Alexa, turn on <light name>": the light switches.
- The smart-home dashboard opens (hold Home, or "Alexa, show my smart home").
- Every app I said I use opens and plays.

If one of these breaks, re-enable the last batch with `tools/restore.sh` and find the package responsible.

## Step 5: speed settings (all can be undone)

- Animations at 0.5x (Developer options, or `settings put global window_animation_scale 0.5`, and the same for
  `transition_animation_scale` and `animator_duration_scale`; set them back to `1` to undo).
- Settings → Preferences → Featured content: autoplay video and audio off (it costs memory and bandwidth on Home).
- Settings → Preferences → Privacy settings: Device usage data, Collect app usage data and Interest-based ads off.
- Settings → Preferences → Data monitoring off.
- Settings → Applications → Manage installed applications: clear the cache of the heaviest apps.

## Step 6: my own view

Because Amazon keeps Home, there are two ways to get a personal view:

1. **Stock home, arranged (recommended, nothing sideloaded).** Move my apps to the front of "Your apps", put the
   camera and smart-home dashboard in reach (pin them if the Fire OS version allows it), and use voice for the camera
   ("Alexa, show <camera name>"). This survives Amazon's updates.
2. **A custom home on top of Amazon's.** A sideloaded launcher (Projectivy, or one of the Fire TV projects in
   [SHOWCASE.md](SHOWCASE.md#other-platforms-fire-tv-and-lg-webos)) brought to the front by a Home watcher. It looks
   cleaner, but it is third-party, it adds about half a second after each Home press, and Amazon's updates have broken
   earlier versions. Only do this after I agree.

## Step 7: close ADB

Turn ADB debugging off on the stick, reboot, then check with `tools/adb-closed.py <stick ip>`.

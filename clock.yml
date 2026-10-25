"""
Discord world-clock embed, run on a schedule by GitHub Actions (no server needed).

Each run edits ONE message with the current time in your zones.
Settings come from GitHub repo secrets/variables:
  WEBHOOK_URL  (secret)   - your Discord webhook URL
  MESSAGE_ID   (variable) - leave unset the first time; the first run posts the
                            message and prints its ID for you to save here.
"""

import os
import time
from datetime import datetime, timezone
from zoneinfo import ZoneInfo
from urllib import request, error
import json

# ---------------- EDIT YOUR ZONES HERE ----------------
# (timezone, label, note)
#   label: None = use the zone's own abbreviation (switches EDT/EST etc. automatically)
#   note:  "" for none
TIMEZONES = [
    ("America/New_York",    None,           ""),
    ("America/Chicago",     None,           "Retail Server Time"),
    ("America/Denver",      None,           ""),
    ("Etc/GMT+6",           "Alberta Time", ""),   # Alberta: fixed UTC-6 all year from Nov 1, 2026
    ("America/Los_Angeles", None,           "Forever Server Time"),
    ("Australia/Sydney",    None,           ""),   # AEST/AEDT, observes DST
]
TITLE = "🕒 Server Clocks"
COLOR = 0x5865F2
USE_24_HOUR = False
# ------------------------------------------------------

WEBHOOK_URL = os.environ["WEBHOOK_URL"].strip()
MESSAGE_ID = os.environ.get("MESSAGE_ID", "").strip()


def build_embed():
    now = datetime.now(timezone.utc)
    fmt = "%H:%M" if USE_24_HOUR else "%I:%M %p"
    lines = []
    for tz, label, note in TIMEZONES:
        local = now.astimezone(ZoneInfo(tz))
        t = local.strftime(fmt)
        if not USE_24_HOUR:
            t = t.lstrip("0")
        mins = int(local.utcoffset().total_seconds() // 60)
        sign = "+" if mins >= 0 else "-"
        h, m = divmod(abs(mins), 60)
        offset = f"UTC{sign}{h}" + (f":{m:02d}" if m else "")
        line = f"**{t}** — {offset} {label or local.tzname()}"
        if note:
            line += f" ({note})"
        lines.append(line)
    return {"title": TITLE, "color": COLOR, "description": "\n".join(lines)}


def call(method, url, body):
    req = request.Request(
        url, method=method, data=json.dumps(body).encode(),
        headers={"Content-Type": "application/json", "User-Agent": "discord-world-clock"},
    )
    with request.urlopen(req) as r:
        return json.loads(r.read() or b"{}")


def main():
    payload = {"embeds": [build_embed()]}
    if MESSAGE_ID:
        try:
            call("PATCH", f"{WEBHOOK_URL}/messages/{MESSAGE_ID}", payload)
            print("Clock updated.")
            return
        except error.HTTPError as e:
            if e.code != 404:
                raise
            print("Saved message was deleted; posting a new one.")
    msg = call("POST", f"{WEBHOOK_URL}?wait=true", payload)
    print("=" * 60)
    print(f"Posted clock message. Save this as the MESSAGE_ID variable: {msg['id']}")
    print("=" * 60)


if __name__ == "__main__":
    main()

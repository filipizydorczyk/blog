Have you ever done backups of your calendars and didn't check if your files are corrupted and then deleted your whole Nextcloud instance and after installing new one you realized your mistake? Me neither... But totally for no reason, I did find a way to get them back if you were using Linux with GNOME! Actually it's not only for Nextcloud but also for Google, Microsoft and every other online account you can add to gnome and used to store your calendars.
[GNOME calendar](https://wiki.gnome.org/Apps/Calendar) is caching your calendars so that you can still see them when you have no access to internet or there are any sync issues. We can use that cache to get your calendars back!
To find your cache, navigate to `$HOME/.cache/evolution/calendar`. You should see something like this

```zsh
➜ ~ $HOME/.cache/evolution/calendar
 ➜ calendar ls
0a87c2b5012933d1534f52d9b997a98d1c318c8f  2c7b3f33739532dbee1091e56ad84be880cc070c  89b418dae31103ee82eea7214e84007a672411ad  9dbe695f4c07b7c5ef62b25930a28b133f33d21e
0c6e6f6cda08e644e6e51470493a5685e17158fd  7fcb599f6fddddbfdb62085740b544bfe1a070cc  9c8b72761f0b2d806b2188b34c05f41324892ca6  trash
 ➜ calendar
```

Each of these directories (except `trash`) are your calendars fetched from the cloud. To know which one is which, you will have to browse its content. Each directory contains `cache.db` file, which is **sqllite** database binary file. We are almost there! To extract `ics` file from database, you can use this one-liner

```sh
sqlite3 ./cache.db "SELECT ECacheOBJ FROM ECacheObjects;" > calendar.ics
```

This is a valid file that will let you import this calendar back to your calendar application. In some cases, there might be an extra step needed. For example GNOME calendar will handle this file just fine and will let you import every event to local calendar however for Nextcloud you will have to edit this file and wrap its content like this

```
BEGIN:VCALENDAR
VERSION:2.0
CALSCALE:GREGORIAN
PRODID:-//SabreDAV//SabreDAV//EN
X-WR-CALNAME:Name
X-APPLE-CALENDAR-COLOR:#5B64B3
REFRESH-INTERVAL;VALUE=DURATION:PT4H
X-PUBLISHED-TTL:PT4H
BEGIN:VTIMEZONE
TZID:Europe/Warsaw
BEGIN:DAYLIGHT
TZOFFSETFROM:+0100
TZOFFSETTO:+0200
TZNAME:CEST
DTSTART:19700329T020000
RRULE:FREQ=YEARLY;BYMONTH=3;BYDAY=-1SU
END:DAYLIGHT
BEGIN:STANDARD
TZOFFSETFROM:+0200
TZOFFSETTO:+0100
TZNAME:CET
DTSTART:19701025T030000
RRULE:FREQ=YEARLY;BYMONTH=10;BYDAY=-1SU
END:STANDARD
END:VTIMEZONE

---
original file content goes here
---

END:VCALENDAR
```

At this point you are able to import it to Nextcloud too (and likely to other services, I didn't check each one of them).

Sources: [wiki.gnome](https://wiki.gnome.org/Apps/Calendar), [stackoverflow: Comments for ics](https://stackoverflow.com/questions/13959916/is-there-a-comment-character-for-ical-files-ics), [unix.stackexchange](https://unix.stackexchange.com/questions/111608/how-to-redirect-sqlite3-output-to-a-file), [stackoverflow: sqlite list of column names](https://stackoverflow.com/questions/947215/how-to-get-a-list-of-column-names-on-sqlite3-database), [help.gnome](https://help.gnome.org/users/evolution/stable/data-storage.html.gl)

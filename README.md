# A new approach for Openstreetmap l10n

Right after I got to be the maintainer of the [German Mapnik style](https://github.com/giggls/openstreetmap-carto-de)
in 2012 I immediately thought that it would be nice to have Latin labels on
my map rather than the respective local script.

This is when the first versions of the OSMl10n functions were born.

At this time implementing them in PL/pgSQL as PostgreSQL stored procedures
seemed to be a natural choice.

Actually this is what the [legacy implementation](https://github.com/giggls/mapnik-german-l10n)
still does.

However starting in 2019 this approach started to show a couple of
limitations.

Many FOSS transcription libraries are written in the **Python** language thus we
already had to switch parts of the code to **PL/Python**.

This started for Thai language using [tltk](https://pypi.org/project/tltk/)
which worked good enough. However trying to use this approach for Cantonese language using
[pinyin_jyutping_sentence](https://pypi.org/project/pinyin_jyutping_sentence/)
was way too slow. Importing this library takes a couple of seconds and can
not be done just once but must be done once per transaction.

Also, we noticed that **PostgreSQL** has a hard coded limit for pre-compiled
regular expressions, which we were using quite heavily for street-name
abbreviations. Exceeding this limit will again slow down queries in an
unacceptable way.

Discussing other approaches we now came up with the following idea:

* Have a transcription daemon written in Python
* Implement a library written in Lua language which can be plugged into the
  Lua tag transformation script of [osm2pgsql](https://github.com/openstreetmap/osm2pgsql)
  (flex backend) or used with the standalone software [osm-tags-transform](https://github.com/osmcode/osm-tags-transform)

## Tests

### Japan
```
transcription-cli/transcribe.py "XY//142/43/東京"
Toukyou
transcription-cli/transcribe.py "CC//jp/東京"
Toukyou
```

### China
```
transcription-cli/transcribe.py "XY//130/43/東京"
dōng jīng
transcription-cli/transcribe.py "CC//cn/東京"
dōng jīng
```

### Thailand
```
transcription-cli/transcribe.py "XY//101/16/ห้องสมุดประชาชน"
hongsamut prachachon
transcription-cli/transcribe.py "CC//th/ห้องสมุดประชาชน"
hongsamut prachachon
```

### Macau
```
transcription-cli/transcribe.py "XY//113.6/22.1/香港"
hōeng góng
transcription-cli/transcribe.py "CC//mo/香港"
hōeng góng
```

### Hongkong
```
transcription-cli/transcribe.py "XY//113.9/22.25/香港"
hōeng góng
transcription-cli/transcribe.py "CC//hk/香港"
hōeng góng
```

## Installation of the Lua library

**This code will not work with lua versions below 5.3!**

Install prerequisites:

```
apt install libunac1-dev luarocks
luarocks install lrexlib-pcre
```

## Installation of the transcription-daemon

Install prerequisites:

```
apt install python3-icu python3-shapely

pip install pykakasi
pip install tltk
pip install pinyin_jyutping_sentence
```

On **Debian/Ubuntu** just call make deb inside base and ``lua_unac``
directories. This will give you two Debian packages which can be installed
on the system.

To test if your installation is working as expected call ``make
test`` inside the ``lua_osml10`` directory.

Make sure that transcription-daemon is running while running tests.

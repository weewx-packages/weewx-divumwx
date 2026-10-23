# DivumWX Installation Guide

This guide covers the unchanged original `ver1.7.0` release from
[Millardiang/weewx-divumwx](https://github.com/Millardiang/weewx-divumwx).
The fixed catalog archive is `divumwx-1.7.0.zip`, with installer version 1.7.0.
It retains the original assets and license files. The older `.post1` catalog
distribution is withdrawn.

Catalog requirements are WeeWX 5.5–5.x and Python 3.13.5 or newer. Use an
existing station with a working hardware driver. Back up `weewx.conf`,
the archive database and web directory before installing or upgrading.

The commands use stock WeeWX's archive installer. Add
`--config /path/to/weewx.conf` when needed. They neither require nor install
an extension catalog command in WeeWX core. Restart the service yourself after
reviewing the generated configuration.

## Before you start

The installer asks several questions. Have these ready:

* An [OpenWeather API key](https://home.openweathermap.org/api_keys) for global
  weather warnings and alerts (optional; leave blank to configure later).
* Your nearest airport [METAR (ICAO) code](https://metar-taf.com/), e.g. `EGTK`.
* If you live in England, your UK Health Security Agency region. The installer
  lists the nine regions and asks for the **last digit** of the code only
  (e.g. `8` for South East, code `E12000008`).
* If you live in the UK, your Met Office region (listed by the installer).

When upgrading, every prompt defaults to the value already in `weewx.conf`,
so pressing Enter keeps your existing setting.

---

## A. WeeWX installed from the Debian package (APT)

This is the recommended layout. WeeWX, Skyfield and Requests are all managed by
APT; no pip or virtual environment is needed.

### Dependencies

```
sudo apt update
sudo apt install python3-skyfield python3-requests
```

### Install or upgrade

```
sudo weectl extension install https://github.com/weewx-packages/catalog/releases/download/divumwx/v1.7.0/divumwx-1.7.0.zip
sudo systemctl restart weewx
sudo weectl report run DivumWXReport
```

The installer creates a `divumwx` subdirectory under the configured web output and sets its ownership to the
account WeeWX runs as (normally `weewx`). No manual `chown` is needed.

### Check the installation

```
weectl extension list          # original archive reports divumwx 1.7.0
sudo journalctl -u weewx -n 100 --no-pager | grep -i divumwx
```

### Uninstall

```
cd /etc/weewx
sudo python3 /etc/weewx/bin/user/divumwx_uninstall_helper.py
sudo weectl extension uninstall divumwx
sudo systemctl restart weewx
```

---

## B. WeeWX installed with pip (virtual environment)

This section assumes WeeWX already runs from `~/weewx-venv`, with its station
under `~/weewx-data`. Replace both paths with your existing installation's
paths before running the commands. Creating a separate environment just for
these libraries will not make them available to your WeeWX service. After
activation, use that environment's `python` and `weectl`.

### Dependencies

Skyfield installs NumPy as a dependency. If your platform lacks a compatible
NumPy wheel, prepare its [native build prerequisites](https://numpy.org/doc/stable/building/)
before running pip.

```
source ~/weewx-venv/bin/activate
python -m pip install 'skyfield' 'requests'
```

### Install or upgrade

A pip install runs `weectl` as your own user, which cannot normally write to
`/var/www/html`. Create the web directory and give it to your user **before**
installing:

```
sudo mkdir -p /var/www/html/divumwx
sudo chown -R $(whoami):$(whoami) /var/www/html/divumwx

source ~/weewx-venv/bin/activate
weectl extension install https://github.com/weewx-packages/catalog/releases/download/divumwx/v1.7.0/divumwx-1.7.0.zip
sudo systemctl restart weewx
weectl report run DivumWXReport
```

### Uninstall

```
source ~/weewx-venv/bin/activate
cd ~/weewx-data
python3 ~/weewx-data/bin/user/divumwx_uninstall_helper.py
weectl extension uninstall divumwx
sudo systemctl restart weewx
```

---

## Upgrading from a beta or 1.0.0

1. Back up `weewx.conf`, your WeeWX database, and the `divumwx` web directory.
2. Run the install command for your layout (A or B above) with the selected fixed archive URL.
3. Press Enter at each prompt to keep your existing settings, or type a new
   value to change it. Changed values are applied.
4. The installer removes frontend files that an earlier DivumWX release
   installed but this release no longer ships. Your own files in the web
   directory (webcam images, timelapse output and generated data) are left
   alone. The list of files removed is printed during the install.
5. Restart WeeWX.

## Troubleshooting

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

## Feedback

Please use the [Issues log](https://github.com/Millardiang/weewx-divumwx/issues)
for feedback. Reports from installs using methods other than those above are
especially welcome.

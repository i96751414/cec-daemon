# cec-daemon

A tool for translating CEC codes into key presses, allowing to control the PC through the TV.

## Dependencies

1. Make sure `python3-cec` and `python3-uinput` are installed:
   ```shell
   sudo apt install python3-cec python3-uinput
   ```

2. Create uinput udev rules at `/etc/udev/rules.d/40-uinput.rules` with the following contents:
   ```
   ACTION=="add|change", KERNEL=="event[0-9]*", ENV{ID_VENDOR_ID}=="012a", ENV{ID_MODEL_ID}=="034b", ENV{ID_INPUT_KEYBOARD}="1", ENV{ID_INPUT_TABLET}="1"
   SUBSYSTEM=="input", ATTRS{name}=="python-uinput", ENV{ID_INPUT_KEYBOARD}="1"
   KERNEL=="uinput", MODE:="0666"
   ```

3. Make sure `uinput` is created and the current user is added to the group:
   ```shell
   sudo addgroup uinput
   sudo usermod -a -G uinput "$USER"
   ```

4. Add `uinput` module to `/etc/modules-load.d/modules.conf` by adding a line with the module name, i.e., `uinput`.

## Build

Make sure `debhelper` is installed:

```shell
sudo apt install debhelper
```

Then, simply run the below command in the project root:

```shell
dpkg-buildpackage -b
```

## Disable CEC service for specific applications

Some applications (for instance, Kodi) have their dedicated CEC service, as such one should disable this service
before launching the said applications.

To do so, one can wrap the entrypoint of the application as shown below:

```shell
# Check for CEC service and disable it if its running
if systemctl is-active --quiet cec.service; then
  should_start_cec=true
  sudo systemctl stop cec.service
else
  should_start_cec=false
fi

# Application launch code goes here
/app/entrypoint $
app_pid=$!
wait "${app_pid}"
return_code=$?

# Enable CEC service again if it was previously disabled
[ "${should_start_cec}" = true ] && sudo systemctl start cec.service
```
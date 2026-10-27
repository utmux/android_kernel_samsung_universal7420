# Samsung Exynos 7420 kernel for LineageOS 17.1

Linux 3.10 kernel tree used by the `universal7420` LineageOS 17.1 device
family. This fork also contains the SM-G9200 (`zerofltechn`) bring-up and its
dual-SIM/RIL-related device support.

## Charging controls

The MAX77843 charger driver exposes optional battery-life controls under:

```text
/sys/class/power_supply/battery/
```

They are disabled by default, so normal charging behavior is unchanged after
boot.

### Charge limit

The defaults stop charging at 80% and resume at 75%. Both thresholds can be
changed before enabling the feature:

```sh
echo 75 > /sys/class/power_supply/battery/charge_control_start_threshold
echo 80 > /sys/class/power_supply/battery/charge_control_end_threshold
echo 1 > /sys/class/power_supply/battery/charge_control_enabled
```

`charge_control_active` reports whether the limit is currently holding the
charger off. The legacy `store_mode` node is retained as an enable/disable
alias.

### Bypass charging

```sh
echo 1 > /sys/class/power_supply/battery/bypass_charging
```

This clears the MAX77843 charging bit while retaining its buck/input power
path. `bypass_charging_active` reports the actual state. For safety, bypass
will not start below 20% battery and charging resumes at 15%; if it remains
enabled, bypass becomes active again at 20%.

This is an experimental hardware-dependent power-path mode. After flashing,
verify it while connected to a stable charger by checking that
`batt_current_ua_now` stays close to zero under a steady workload. Disable it
immediately if the battery continues discharging quickly.

Additional diagnostics:

```text
charging_input_current_limit
charging_input_current_now
batt_current_ua_now
batt_current_ua_avg
```

All settings reset to their safe defaults on reboot. A root startup script or
compatible kernel manager may reapply them.

## Building

Build this kernel through the matching LineageOS 17.1 Android source tree:

```sh
source build/envsetup.sh
lunch lineage_zerofltechn-userdebug
m -j4 bootimage
```

The resulting image is written to
`out/target/product/zerofltechn/boot.img`.

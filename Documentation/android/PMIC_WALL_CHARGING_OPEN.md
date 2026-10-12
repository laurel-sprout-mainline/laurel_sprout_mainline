# Wall-adapter charging remains slow at low SOC

Status: open, deferred for a future charging investigation.
Reported by the maintainer on 2026-10-12, after the latest PMIC changes.

The device otherwise appears stable in the maintainer's use. A wall adapter
still produces the Android “Charging slowly” label at 22% battery. Franco
Kernel Manager reports approximately 0.4 W. The adapter/cable specifications,
measurement duration, battery temperature and exact installed build were not
captured during that wall-charger session.

This low-SOC observation cannot be explained solely by the earlier proposed
near-full (90% and above) taper. The 0.4 W figure is an application-reported
value: its source and sign convention have not been verified. It may represent
net battery power after the phone's own consumption, rather than adapter
input power. Do not treat it as an external USB power-meter measurement.

Previous build #33 laptop measurements and remaining gaps are recorded in
[PMIC_BUILD_33.md](PMIC_BUILD_33.md). Those laptop readings do not establish
wall-adapter performance. Relevant unresolved paths include APSD source
classification, DPDM ownership, QC negotiation/fallback, settled input-current
limits, thermal limits and the SMB1355 probe failure. None is yet established
as the cause of this low-SOC report.

For a future fix, capture a single wall session at low SOC with adapter/cable
identity, USB type, VBUS voltage/current, input-current limit, battery
voltage/current/temperature, raw charger state and negotiation logs. Compare
with the same adapter and cable on the LineageOS SM6125 4.14 kernel. Verify
whether Android's label follows the actual input limit, and independently
measure adapter input power if possible. Trace the limiting path before
changing current limits or charge-status reporting.

No charging-driver or policy change accompanies this report. Fast charging,
18 W operation and accurate present battery capacity remain unverified.

# Reporting changes in the contribution fork

This fork extends the existing Rachio Supervisor companion integration. Home
Assistant's built-in Rachio integration continues to own device access and
watering. There is no Home Assistant Core fork or personal release. The weather-unit
fix is submitted separately in [upstream PR #1](https://github.com/david00769/ha-rachio-supervisor/pull/1);
the schedule, zone and upcoming-run changes remain in this fork. These changes are based on upstream main
`f0c3fbbef7e667f3f9768df6abcfa9c6f9ca510d`, rather than a new personal release line.

## Implemented behavior

- Discover both `scheduleRules` and `flexScheduleRules`, deduplicated by vendor
  id. Use stable registry ids to associate schedules even when names coincide.
- Project each enabled physical zone exactly once, including zones without an
  enabled schedule. Show all schedule memberships. Use per-zone durations from
  the next predicted run, or an unambiguous single schedule's zone duration;
  never use a multi-zone schedule's total as one zone's default.
- Keep schedule-level policy and moisture mapping separate. A multi-zone rule
  cannot become a single moisture write target through a name guess. Quick Run
  requires an exactly resolved zone and retains the existing confirmation and
  1–60 minute service bound. Unknown zone ids cannot fall back to a schedule.
- Read Rachio's official 14-day `upcoming_runs` forecast. Expose dated runs,
  predicted skips and reasons, per-zone watering seconds, elapsed completion
  window and cycle/soak separately through Zone overview attributes and a native
  read-only **Upcoming runs** calendar. Dates are predictions, not execution
  evidence. The calendar state is a predicted run window, not valve telemetry.
- Share one coordinator cache between cards and the calendar. Full evidence
  reconciliations include the forecast; outside that cadence, a single forecast
  request refreshes it every 15 minutes. Opening a card/calendar adds no request.
  Calendar API errors are exposed as unavailable, separately from a successful
  empty forecast. The last successful fetch timestamp remains visible.
- Request forecast weather explicitly in US units and normalize Fahrenheit and
  inches to the existing Celsius/mm attribute contract. Support the vendor's
  `temperatureMin`/`temperatureMax` fields. This avoids assuming a `METRIC`
  response is metric when it contains unlabelled US-scale values. Weather
  forecasts remain excluded from observed rainfall decisions.

## Native reporting

After installing a tested build, add the integration's Upcoming runs entity to a
native Calendar card, or use `calendar.get_events` for a bounded date range.
Predicted skips are labeled explicitly and are not selected as the next watering
event. Per-zone next-run values come only from the official response; no run date
is synthesized from weekday rules, summaries or Home Assistant attributes.
The API only supplies a rolling two-week forecast and omits runs already started;
Flex predictions may change with weather. It is not a historical execution log.

References: [upcoming runs](https://rachio.readme.io/reference/publicdeviceidupcoming_runs),
[forecast units](https://rachio.readme.io/reference/publicdeviceidforecastunitsunits).

## Validation and remaining gate

The deterministic suite uses synthetic controllers and mocked service calls; it
never waters a device. Coverage includes fixed/Flex identity, multi-zone and
shared-zone inventory, exact target selection, per-zone runtime, forecast
failure versus empty results, skips, cycle/soak timing, calendar filtering/cache,
and weather conversion. Run with Python 3.13 or newer:

```sh
python3.13 -m unittest -q tests.test_supervisor_logic
python3.13 -m compileall -q custom_components
node --check custom_components/rachio_supervisor/www/rachio-supervisor-zone-grid-card.js
```

The default suite stubs Home Assistant's import surface. A separate offline
suite runs against real Home Assistant 2026.10.4 (Python 3.14), checking calendar
platform addition, state updates, availability, listener removal and real
CalendarEvent validation/serialization. Run it in its own environment/process:

```sh
python3.14 -m venv /tmp/rachio-reporting-tests
/tmp/rachio-reporting-tests/bin/pip install -r tests/requirements-homeassistant.txt
/tmp/rachio-reporting-tests/bin/python -m unittest -v tests.test_reporting_homeassistant
```

It uses a standalone calendar platform with synthetic data, without full Rachio
config-entry/device setup or vendor requests. It does not establish a live
household installation, dashboard rendering or physical watering result. Before
deployment, check integration/config-entry reload and native calendar/zone
attributes in the target environment, then review phone/tablet rendering. Production deployment, submission of the other changes and releases
remain separate decisions. Existing household operation stays on its published
upstream version until an explicit deployment step is authorized and verified.

These are generally useful upstream fixes. Prefer contributing them after review
and using a released upstream version when available, then retire the fork's
runtime divergence. Keep personal repository-bootstrap files out of any future
upstream contribution.

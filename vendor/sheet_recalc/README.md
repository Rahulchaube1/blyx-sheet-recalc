# Sheet Recalc

Fence cached spreadsheet results to the revision that triggered recalculation.

Document production and accessibility · framework · 0.1.2

Framework bird: Oilbird.

## Drive the workflow

Start with `sheet_recalc_start()`. For each event call `sheet_recalc_step(snapshot, event, value)`, persist the returned snapshot, and dispatch `sheet_recalc_command(snapshot)` to your application’s adapter. The host owns I/O, event ordering, duplicate detection and durable command delivery. A matching event produces one command. Rejected events retain the phase and saved quantity. An invalid snapshot yields an empty vector.

The snapshot contains phase ID, saved quantity and last command ID. Event values are whole numbers from zero through one billion. In the table, **x** is the event value and **r** is the saved quantity. Rules match in listed order. To validate a whole history without running effects use `sheet_recalc_replay(events, values)`. It returns the final snapshot or an empty vector on rejection.

| Current phase | Event | Guard | Next phase | Command | New quantity |
|---|---|---|---|---|---|
| idle | edit | x>0 | dirty | mark_dirty | x |
| dirty | edit | x>r | dirty | coalesce_edits | x |
| dirty | run | always | computing | evaluate_cells | r |
| computing | edit | x>r | dirty | invalidate_run | x |
| computing | result | x=r | clean | publish_values | r |
| clean | edit | x>r | dirty | mark_dirty | x |

Phase IDs in order from zero: idle, dirty, computing, clean.

## Replay your own events

After installing the demo, run the following from its directory. Supply alternating event names and integer values. The command prints each requested action and exits with status 1 at the first rejected transition, or 2 for malformed arguments.

```sh
blyxc native-run main.blyx -- edit 1 run 0 edit 2 run 0 result 2
```

## Try the published project

```sh
nest demo sheet_recalc@0.1.2
blyxc native-run sheet_recalc-demo/main.blyx
```

Install with `nest add sheet_recalc@0.1.2` and import `vendor::sheet_recalc::sheet_recalc`.

Maintained by Rahul Chaube. [Blyx](https://www.blyx-lang.space) · [GitHub](https://github.com/Rahulchaube1) · [LinkedIn](https://www.linkedin.com/in/rahulchaube1) · [Sponsor](https://github.com/sponsors/Rahulchaube1)

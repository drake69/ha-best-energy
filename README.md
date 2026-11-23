# Best Energy for Home Assistant

> **Status: proposal.** There is no code yet. This page describes the idea so it can be discussed
> before anything is written. Feedback is welcome until **2026-11-30**, see [Join the discussion](#join-the-discussion).

Best Energy is a planned custom integration for Home Assistant. It holds the prices you pay for each
energy carrier (electricity, gas, district heating, water...), split into time bands that you enter by hand
when your contract changes. When an automation or an external system asks, it answers two questions:

1. **When is it cheapest?** The time slots in a window, ordered from the lowest to the highest price, so a
   deferrable load (a washing machine, a heat pump boost, a battery) can wait for the best one.
2. **What does it cost right now?** The price of one unit of a carrier at the moment of the request, with
   its unit of measurement.

## Why

Many households are on fixed time-of-use contracts rather than on a market feed. In Italy, for example,
electricity is priced in the F1, F2 and F3 bands, which also depend on public holidays, and gas is priced per
standard cubic metre. The prices change a few times a year and come from a paper bill, not from an API.

Most price integrations for Home Assistant read a market feed (Nord Pool, ENTSO-e, EPEX Spot) or a single
supplier. Two existing projects come close:

- [Utility Tariffs](https://github.com/macuistin/ha-utility-tariffs) accepts your own rates with time bands
  and has `find_window` and `get_prices` actions. Today it covers electricity only, follows one tariff per
  Home Assistant instance and changes rates through Reconfigure.
- [Cheapest Energy Hours](https://github.com/TheFes/cheapest-energy-hours) is a Jinja macro that ranks the
  cheapest hours very well, but it needs a price sensor to read from and has no notion of carrier or unit.

If one of them, or another project, can grow to cover this use case, a pull request there is better than a
new integration. That is one of the questions below.

## Proposal

### Carriers and units

One config entry per carrier, each with its own currency and unit of measurement, so that prices are never
mixed across carriers:

| Carrier | Typical price unit |
| --- | --- |
| Electricity | `EUR/kWh` |
| Gas | `EUR/m³`, or `EUR/kWh` converted with the calorific value on the bill |
| District heating | `EUR/kWh` |
| Water | `EUR/m³` |

### Time bands

A band is a set of weekdays, a time range and a price. Bands can overlap only if a rule says which one wins.
An optional holiday calendar lets a holiday follow the bands of a Sunday, as the Italian F1, F2 and F3 bands do.

### Actions

All actions return data, so they can be used with `response_variable` in automations and scripts, and from
the REST and WebSocket APIs.

- `best_energy.set_prices`: replaces the bands and prices of a carrier, from an automation or a script,
  without reconfiguring the integration.
- `best_energy.get_ranked_slots`: takes a carrier, a time window and optionally a duration, and returns the
  slots ordered by ascending price.
- `best_energy.get_current_price`: takes a carrier and returns the price per unit now, with its unit and band.

Example of a response from `get_ranked_slots`:

```yaml
carrier: electricity
unit: EUR/kWh
window:
  start: "2026-12-01T18:00:00+01:00"
  end: "2026-12-02T07:00:00+01:00"
slots:
  - start: "2026-12-01T23:00:00+01:00"
    end: "2026-12-02T07:00:00+01:00"
    band: F3
    price: 0.118
  - start: "2026-12-01T19:00:00+01:00"
    end: "2026-12-01T23:00:00+01:00"
    band: F2
    price: 0.131
  - start: "2026-12-01T18:00:00+01:00"
    end: "2026-12-01T19:00:00+01:00"
    band: F1
    price: 0.146
```

The prices above are made up for the example.

### Entities

One price sensor per carrier, with the current price as its state, its unit, and the band in force as an
attribute, so it can be used in the Energy dashboard. No entity per combination of carrier and band is needed:
the actions take the carrier as a parameter.

## Open questions

1. Does something like this already exist, maybe as a blueprint or a template pattern?
2. Would you rather see it as a pull request to an existing integration than as a new one? If so, which one?
3. For gas: do you track the price per m³, or convert it to kWh with the calorific value on your bill?
4. What is missing for your contract: tiered prices by consumption, fixed monthly fees, different prices on
   weekends, something else?

## Join the discussion

- **Home Assistant Community forum**: the main thread, link coming soon.
- **GitHub Discussions**: [RFC: scope and design](../../discussions), if you prefer GitHub.

Replies from both places will be collected on 2026-11-30, and the decision will be posted in both.

## License

[MIT](LICENSE)

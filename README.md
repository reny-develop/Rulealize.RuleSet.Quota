# Rulealize.RuleSet.Quota

A bounded allowance: so many of something, and then no more — a
[Rulealize](https://github.com/reny-develop/Rulealize) rule set written to be **held**.

| | |
| --- | --- |
| Rule set id | `Rulealize.RuleSet.Quota` |
| Package | [`Rulealize.RuleSet.Quota`](https://www.nuget.org/packages/Rulealize.RuleSet.Quota) |
| Inputs | `consume` `reset` |
| Holds | nothing |
| Draws on | TypeSchema, State, Comparison, Arithmetic |

```json
"uses": [
  { "ruleSet": "Rulealize.RuleSet.Quota", "version": "^1.0", "as": "quota" }
]
```

**`as` is not optional.** An alias defaults to the identifier and may not contain a `.`, so an
entry that leaves it out is refused, with a message about a key you did not write.

## What is being counted is not in this document

`allowed` is a number the state document supplies, and what one of them buys is yours:

```json
"quota": {
  "ruleSet": "Rulealize.RuleSet.Quota@1.0.0",
  "data": { "allowed": 3, "used": 1 }
}
```

An API call, a free delivery, a retry, a seat. Three quotas over three different things are
three state documents and this one rule set. `state.initial` allows nothing, because a held
rule set opens where it opens and a demonstration allowance written here would be forced on
every document that holds it.

## What a holder gets

| | |
| --- | --- |
| `rec.at($quota, "allowed")` | how many between resets |
| `rec.at($quota, "used")` | how many have gone |

Two ways to use it, and they are the two things holding is for.

**Refuse.** Guard your own input on what is left:

```json
"when": {
  "op": "cmp.lt",
  "left":  { "op": "rec.at", "record": "$quota", "key": "used" },
  "right": { "op": "rec.at", "record": "$quota", "key": "allowed" }
}
```

**Drive.** Better: hide `consume` and spend the allowance in the same transition as the thing
it pays for, so there is no state in which the call was made and the quota not yet charged.

```json
"held":   { "quota": { "consume": { "when": false } } },

"inputs": {
  "send": {
    "fires":   [ { "held": "quota", "input": "consume" } ],
    "effects": [ … your own write … ]
  }
}
```

`fires` puts the fired input through its own guard, so `send` simply stops being offered when
the quota is spent. You do not write that rule twice.

## Why `reset` and not an expiry

A quota that refilled itself would need a clock, and there is no time vocabulary in this
ecosystem — `time` is a reserved namespace with nothing behind it.

So refilling is an input, and *when* it happens is a decision its holder makes. That is also
the more honest shape: a monthly allowance is a business rule about months, which is something
the holder knows about and this document never could.

## Where it fits

| The allowance is | and the holder |
| --- | --- |
| free deliveries this month | refuses the fourth, or charges for it |
| retries of a failing job | gives up when they are gone |
| seats on a plan | refuses to add a user |
| API calls | throttles |

**No `terminal`.** A quota does not end — it is exhausted, which is a state it leaves the
moment somebody resets it. A holder that wants an ending has one of its own.

## Trying it

Needs [Rulealize.Cli](https://github.com/reny-develop/Rulealize.Cli):
`dotnet tool install -g Rulealize.Cli`.

```sh
rulealize restore src/Rulealize.RuleSet.Quota/ruleset/quota.json
rulealize play    src/Rulealize.RuleSet.Quota/ruleset/quota.json --state state/example.json
```

Take `consume` twice and it is gone, leaving `reset` alone. Run it without `--state` and there
is no legal input at all — the same document saying nobody has told it what it allows.

## Building the package

```sh
dotnet pack src/Rulealize.RuleSet.Quota -c Release
```

A package with no `lib` folder, holding this one document under `ruleset/`.
[What each property in the project file is for](https://github.com/reny-develop/Rulealize.Registry/blob/main/doc/publish.md#a-rule-set).

## License

Apache-2.0, and it covers the document as much as everything else here.

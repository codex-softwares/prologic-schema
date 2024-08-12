# Prologic templates, Json dialect guide

Prologic aims to help decision taking by dynamically interacting with a user using questions.

Questionnaires are based on templates. Those templates describe both the questions the user will use and the logic making it dynamic.

There is 2 categories of elements in Prologic's templates:
- The inputs: they are declared entry points which can receive user's actions and external data.
- The variables: they are rules, either numeric formulas or boolean which constitute the logic of the questionnaire.

## Inputs

Inputs are declared in the input list:

```json
{
  "inputs": []
}
```

### Questions

Questions are of 3 types: 
- context selector
- multiple choices
- numeric options (`"type": "NUMERIC_OPTIONS"`)

They are all represented as checkbox form.

The `type`  
All the questions come with a `title` and a text describing it more in `details`.

#### Data probing (`"type": "DATA"`)

This type of input allows to connect to external data sources and to map probes onto discret options.

Here is an example:
```json
{
  "type": "DATA",
  "universe": "factory1",
  "title": "Facility",
  "details": "Select the facility",
  "sources": {
    "pressure": {
      "title": "Pressure inside the tank",
      "probe": "tank-pressure"
    },
    "temp": {
      "title": "Temperature in the tank",
      "probe": "tank-temp"
    },
    "mixer-watt": {
      "title": "Power consumption of the mixer",
      "probe": "mixer-watt",
      "effects": [
        {
          "title": "Off",
          "when": {
            "<": 50
          }
        },
        {
          "title": "On",
          "when": {
            ">=": 50
          },
          "set": {
            "mixer-ON": true
          }
        }
      ]
    }
  }
}
```

Each source with effects is then displayed as a numerically multiple choice mapped question.

The `universe` references an external data set, see the chapter about the data-sets.

#### Multiple choices (`"type": "MULTIPLE_CHOICES"`)

This is the most basic way of interaction for user. It involves a set items represented by checkboxes for which effects on variables can be attached to.
It is also possible de define more complex rules also triggering effects on variables.

Here is an example:
```json
{
  "type": "MULTIPLE_CHOICES",
  "id": "b",
  "title": "Reason for stop",
  "minAnswers": 1,
  "maxAnswers": 1,
  "details": "Please select the reason for the machine to stop",
  "options": [
    {
      "id": "b0",
      "title": "Planned stop"
    },
    {
      "id": "b1",
      "title": "Power outage",
      "set": {
        "power-involved": true
      }
    },
    {
      "id": "b2",
      "title": "Emergency",
      "add": {
        "risk-score": 2
      }
    }
  ],
  "rules": [
    {
      "ifAnswers": {
        "contains": 1,
        "of": [
          "b2",
          "b1"
        ]
      },
      "then": {
        "set": {
          "failure": true
        }
      }
    },
    {
      "ifStateIs": "Answered",
      "then": {
        "set": {
          "stopping-reason-known": true
        }
      }
    }
  ]
}
```

- The `id` is required to reference the question in other rules.
- The `minAnswers` property determines the amount of selected items for the question to be considered answered.
- The `maxAnswers` property limits the user's ability to select too much items.
- The `options` property defines the list of items.
- The `rules` property defines complex logic involving potentially more than on single item.

The items can have *effects* declared using the `whenSelect` property. They can be of 2 kinds:
- `add` a quantity `to` a declared *counter*.
- `set` a *fact* to a given boolean value. 

#### Numeric options (`"type": "NUMERIC_OPTIONS"`)

This type of question is similar to the *multiple choises* questions. Though, items are intrinsically associated with a numeric value.</br>
Only one single item can be selected at a time.

Here is an example:

```json
{
  "type": "NUMERIC_OPTIONS",
  "id": "T",
  "title": "Temperature of the liquid in the tank",
  "details": "If no automatic measure is available go and check the thermometer.",
  "mappedOn": "temp",
  "options": [
    {
      "id": "T1",
      "title": "< 50°C",
      "<": 50
    },
    {
      "id": "T2",
      "title": "< 95°C",
      ">=": 50,
      "<": 98
    },
    {
      "id": "T4",
      "title": ">= 98°C",
      ">=": 98,
      "set": {
        "overheat": true
      }
    }
  ]
}
```

- The `id` is required to reference the question in other rules.
- The `options` property defines the list of items. Items contains range definitions (`"<"`, `"<="`, `">"`, `">="`) which can be mixed.
- The `mappedOn` property is optional and can be use to automatically preselect an item based on either a probe or a variable.

The items can have *effects* declared using the `whenSelect` property. They can be of 2 kinds:
- `add` a quantity `to` a declared *counter*.
- `set` a *fact* to a given boolean value. 

## Variables

What makes forms dynamic is the set of variables that can be declared in the `vars` section,
There is mainly 2 kind of variables, facts and counters.

### Facts

Facts has pseudo-boolean values. They can be `True`, `Talse` or `Undefined` by default.


Many combination of condition can be used among which:
- `and`: (logical and of other conditions).
- `or`: (logical or of other conditions).
- `all`: (logical and of facts).
- `any`: (logical or of facts).
- `inputs` - `are`: all the referenced inputs have the target state.
- `valueOf` - `<`/`<=`/`>`/`>=`: a numeric value correspond to the numerical criterie.

Here are examples:

```json
{
  "vars": {
    "explosion-risk": {
      "default": "False",
      "trueIf": {
        "all": {
          "overheat": true,
          "overpressure": true
        }
      }
    },
    "dump-required": {
      "default": "False",
      "trueIf": {
        "any": {
          "overfilling": true,
          "overpressure": true
        }
      }
    },
    "all-done": {
      "inputs": [
        "a",
        "b",
        "c"
      ],
      "are": "Answered"
    },
    "score-threshold": {
      "valueOf": "score",
      ">=": 0,
      "<": 5
    }
  }
}
```

### Counters

Counters are variables acting as sums of values comming for input *effects".

Here is an example:

```json
{
  "vars": {
    "score": {
      "numberType": "Integer",
      "initialValue": 0,
      "title": "Malinas",
      "details": "This is a counter"
    }
  }
}
```

The `initialValue` allows to start from another value than 0.
The `numberType` will serve in a future version.


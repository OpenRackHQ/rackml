# RackML

**An open XML schema for describing datacenter racks, appliances, ports, slots, and connections with dimensional precision.**

RackML (Rack Markup Language) is a human-readable, XML-based format for modeling physical datacenter infrastructure. It captures rack geometry, equipment placement, port layouts, and cable connections — all with real-world dimensional accuracy down to the millimeter.

## Why RackML?

Datacenter planning today is fragmented across 2D diagrams, spreadsheets, and proprietary vendor tools that don't talk to each other. RackML provides a vendor-neutral, open standard that:

- **Describes physical reality** — rack post geometry, appliance dimensions, port positions on specific faces
- **Models connections** — network, power, and fiber cabling with cable type metadata
- **Supports modular hardware** — slot/component hierarchy for line cards, PSUs, and expansion modules
- **Uses real units** — millimeters, meters, and rack units (e.g. 1U = 44.45mm per EIA-310)
- **Is plain XML** — version-controllable, diffable, parseable by any language

## Quickstart

A minimal valid RackML document:

```xml
<rack name="my-rack" height="42">
    <appliance name="server1" y="1u" type="server" />
</rack>
```

This creates a standard 42U rack with a single server at position 1U.

## Element Hierarchy

```
rack
├── appliance
│   ├── slot
│   │   └── component
│   │       └── port
│   ├── port
│   └── annotation
├── connection
└── annotation
```

## Dimensional Units

All size and position attributes accept values with explicit unit suffixes:

| Unit | Suffix | Description | Example |
|------|--------|-------------|---------|
| Millimeters | `mm` | Absolute measurement | `size_x="440mm"` |
| Meters | `m` | Absolute measurement | `D="1m"` |
| Rack Units | `u` | 1U = 44.45mm (EIA-310) | `y="10u"`, `size_y="2u"` |

Bare numbers without a suffix are interpreted as millimeters (e.g., `"440"` is equivalent to `"440mm"`).

## Common Attributes

Most elements inherit these attributes for identification, sizing, and positioning.

### Identification & Metadata

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | No | Document-unique identifier (XML ID type) |
| `name` | string | No | Human-readable label |
| `sku` | string | No | Part number or stock-keeping unit |
| `weight` | number | No | Weight in kilograms |

### Size & Position

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `size_x` | dimension | No | Width |
| `size_y` | dimension | No | Height |
| `size_z` | dimension | No | Depth |
| `x` | dimension | No | X position relative to parent |
| `y` | dimension | No | Y position relative to parent |
| `z` | dimension | No | Z position relative to parent |

### Bezel

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `Bd` | dimension | No | Bezel depth |
| `Bw` | dimension | No | Bezel width |

## Element Reference

### `<rack>` — Root Element

The rack enclosure. All other elements are children of `<rack>`.

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `version` | string | No | RackML schema version (e.g., `"1.0.0"`) |
| `height` | positiveInteger | **Yes** | Number of rack units (e.g., `"42"`) |
| `A` | dimension | No | Horizontal distance between inner edges of front posts |
| `B` | dimension | No | Horizontal distance between center of mounting holes |
| `C` | dimension | No | Horizontal distance between outer edges of front posts |
| `D` | dimension | No | Rack depth |
| `U` | dimension | No | Physical size of one rack unit (default: `"44.45mm"`) |

**Child elements:** `<appliance>`, `<connection>`, `<annotation>`

```xml
<rack name="datacenter-rack-01" height="42" 
      A="450mm" B="465mm" C="483.4mm" D="1m" U="44.45mm">
    <!-- appliances, connections, etc. -->
</rack>
```

> **Note:** The `A`, `B`, `C`, `D` geometry attributes follow the EIA-310-E standard for 19-inch rack specifications. You may define your own `A`, `B`, `C`, and `D` values.

### `<appliance>` — Equipment

Servers, switches, storage arrays, PDUs, UPS units, and any other rack-mounted equipment.

When used as a standalone document root (without a parent `<rack>`), an `<appliance>` element defines a reusable appliance library entry that can be imported into any rack.

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `type` | string | No | — | Appliance classification (e.g., `"server"`, `"switch"`, `"pdu"`) |
| `version` | string | No | — | Schema version for this appliance definition |
| `anchor` | string | No | `"front"` | Mounting reference point (`front` / `rear`) |
| `position` | string | No | `"mounted"` | Installation position type |
| `rot_x` | decimal | No | `0` | Rotation around X-axis in degrees |
| `rot_y` | decimal | No | `0` | Rotation around Y-axis in degrees |
| `rot_z` | decimal | No | `0` | Rotation around Z-axis in degrees |
| `_manufacturer` | string | No | — | User-defined manufacturer (e.g., `"Dell"`, `"Cisco"`) |
| `_model` | string | No | — | User-defined model (e.g., `"R740"`, `"C9300-48T"`) |
| `_description` | string | No | — | User-defined notes |

> **Custom attributes:** Any attribute prefixed with `_` is a user-defined extension field. The schema reserves `_manufacturer`, `_model`, and `_description` as conventions, but you can add any `_`-prefixed attribute. The XSD also allows attributes from other XML namespaces on all elements via `xs:anyAttribute`.

**Child elements:** `<slot>`, `<port>`, `<annotation>`

```xml
<appliance name="web-server-01" y="1u" type="server"
           _manufacturer="Dell" _model="R740"
           size_x="440mm" size_y="2u" size_z="800mm">
    <port name="eth1" type="rj45" x="50mm" y="0" face="front" />
    <port name="power1" type="c14" x="10mm" y="0" face="rear" />
</appliance>
```

### `<slot>` — Expansion Bay

Defines a modular bay within an appliance that accepts components (PSU bays, line card slots, drive bays, PCIe slots).

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `accepts` | string | **Yes** | Component type this slot accepts (e.g., `"psu"`, `"linecard"`, `"pcie"`) |
| `face` | string | **Yes** | Which face of the appliance (`front` / `rear`) |
| `rotation` | decimal | No | Rotation in degrees around Z-axis |

**Child elements:** `<component>` (0 or 1)

```xml
<slot name="psu-slot-1" accepts="psu" face="rear"
      x="10mm" y="10mm" size_x="100mm" size_y="50mm">
    <component type="psu" name="750w-psu">
        <port name="power-in" type="c14" x="10mm" y="10mm" />
    </component>
</slot>
```

### `<component>` — Installed Module

A modular component installed in a slot — power supplies, line cards, drive modules, etc.

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | **Yes** | Component type classification |

**Child elements:** `<port>`

```xml
<component type="linecard" name="48-port-ethernet">
    <port name="eth1" type="rj45" x="10mm" y="10mm" />
    <port name="eth2" type="rj45" x="20mm" y="10mm" />
</component>
```

### `<port>` — Connection Point

A physical port on an appliance or component — network, power, console, USB, etc.

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `type` | string | **Yes** | — | Port type identifier |
| `face` | string | No | — | Which face the port is on (`front` / `rear`) |
| `rotation` | decimal | No | `0` | Rotation in degrees around Z-axis |

**Common port types:**

| Type | Description |
|------|-------------|
| `rj45` | Ethernet (copper) |
| `rj45_f` | Ethernet (female) |
| `sfp` | 1G fiber / copper SFP |
| `sfp+` | 10G fiber / DAC |
| `qsfp` | 40G/100G fiber |
| `fiber` | Generic fiber port |
| `c14` | IEC C14 power inlet (equipment side) |
| `c13` | IEC C13 power outlet (PDU side) |
| `c20` | IEC C20 high-power inlet |
| `console` | Serial console port |
| `serial` | RS-232 serial port |
| `usb` | USB port |
| `hdmi` | HDMI video output |
| `displayport` | DisplayPort video output |
| `vga` | VGA video output |

Implementations should accept additional type values to support new connector standards without a schema revision.

```xml
<port name="mgmt-port" type="rj45" x="380mm" y="0" face="front" />
<port name="power-in-1" type="c14" x="50mm" y="0" face="rear" />
```

### `<connection>` — Cable / Link

Defines a cable or logical connection between two ports. Port references use slash-delimited path notation: `appliance-name/port-name`.

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `from` | string | No | Source port path (e.g., `"server-01/eth1"`) |
| `to` | string | No | Destination port path (e.g., `"switch-01/port24"`) |
| `cable-type` | string | No | Cable medium (e.g., `"Cat6"`, `"OM4"`, `"DAC"`) |

```xml
<connection name="uplink-1" from="server-01/eth1" to="switch-01/port24" cable-type="Cat6" />
```

### `<annotation>` — Notes

User-authored notes attached to a rack or appliance. Can be placed as a child of `<rack>` or `<appliance>`.

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | **Yes** | Annotation body (plain text) |
| `author` | string | No | Who wrote it |
| `date` | string | No | ISO 8601 timestamp (e.g., `"2026-03-15T14:30:00Z"`) |

```xml
<rack name="prod-rack-01" height="42">
    <annotation text="Primary production rack" author="jsmith" date="2024-01-15T10:30:00Z" />
    <appliance name="web-server" y="1u" type="server">
        <annotation text="Scheduled for replacement Q2 2024" author="ops-team" />
    </appliance>
</rack>
```

## Design Principles

1. **Dimensional accuracy** — Every position and size maps to real physical measurements. No abstract grid cells.
2. **Front/rear awareness** — Ports and slots specify which face they're on, enabling accurate front-of-rack and rear-of-rack views.
3. **Connection completeness** — Both network and power cabling are first-class citizens with cable type metadata.
4. **Extensibility** — User-defined `_`-prefixed attributes let you attach any metadata without schema changes.
5. **Human readability** — Plain XML that engineers can read, write, and diff in version control.

## File Extensions

| Extension | Usage |
|-----------|-------|
| `.rml` | RackML document; Preferred |
| `.xml` | Also valid; Use only when required by constraints |

## Contributing

Contributions are welcome. Please open an issue to discuss proposed schema changes before submitting a pull request.

## License

Licensed under the [Apache License 2.0](LICENSE).

```
Copyright 2024 Logistic Support Alliance

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## Trademark

"RackML" is a trademark of Logistic Support Alliance. You may use the name to refer to the schema format, but you may not use it in a way that implies official endorsement of your product or service without written permission.

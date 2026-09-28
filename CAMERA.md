# MH3U (Wii U) – Smooth Vertical Camera

A Cemu cheat for **Monster Hunter 3 Ultimate (Wii U)** that replaces the stepped on-land vertical camera with the smooth, free tilt the game already uses underwater.

By default, pushing the camera up or down on land jumps between a few fixed height levels. Underwater, the camera tilts smoothly for as long as you hold the input. This cheat gives the land camera the underwater behaviour, with configurable tilt limits.

## Requirements

- Monster Hunter 3 Ultimate **EU**, **update v32**. The addresses are specific to that executable; other regions or versions need the addresses found again.
- A Cemu build with Gateway-style cheat support, such as [Cemu-Enhanced](https://github.com/jM5557/Cemu-Enhanced).

## Installation

1. Right-click the game in Cemu and open **Cheats** (or use **Tools → Cheats** while the game is running).
2. Paste the cheat below and tick it.
3. To go back to the original camera without restarting, untick the cheat and tick the OFF cheat once.

## The cheat

Default limits: **88° up everywhere**, **55° down on land**, **88° down underwater** (the game's original values).

```
[Smooth vertical camera (underwater style)]
{Land camera uses the underwater free-tilt. Land: 88 up / 55 down. Underwater: unchanged.}
*cemu_enabled
022868D4 60000000
0228686C 60000000
01BFEA00 9421FFF0
01BFEA04 7C0802A6
01BFEA08 90010014
01BFEA0C 80B900A4
01BFEA10 81990098
01BFEA14 7C8C2A14
01BFEA18 38610030
01BFEA1C 4845EBD9
01BFEA20 80010014
01BFEA24 7C0803A6
01BFEA28 38210010
01BFEA2C 8099009C
01BFEA30 4E800020
0228A6D4 4B97432D
01BFEA40 819C0E30
01BFEA44 898C003D
01BFEA48 2C0C0003
01BFEA4C 3980C16C
01BFEA50 41820008
01BFEA54 3980D8E4
01BFEA58 7C006000
01BFEA5C 4E800020
02286A40 4B978001
02286A48 7D976378
```

### OFF (restores the original camera)

```
[Smooth vertical camera (underwater style) OFF]
022868D4 4082033C
0228686C 418204CC
0228A6D4 8099009C
022869F8 2C0B3E94
02286A0C 3AE03E94
02286A40 2C00C16C
02286A48 3AE0C16C
```

## What to expect

- Holding camera up or down tilts the camera smoothly, speeding up gently like it does underwater.
- The camera height stays at the middle level. The tilt replaces the old stepped heights.
- Small tilts drift back to level when you let go (the game's own auto-return). Larger tilts stay where you leave them.
- At extreme angles the camera can clip into terrain.

## Changing the tilt limits

The game stores angles as 16-bit values where **65536 = 360°**.

- **Up value** = angle × 65536 ÷ 360, rounded, written in hex
- **Down value** = `10000` − up value (hex)

"Up" means the direction you get by pressing camera **up**. With inverted camera controls, "up" and "down" below swap.

### Angle table

| Angle | Up value | Down value |
|------:|:--------:|:----------:|
| 30° | `1555` | `EAAB` |
| 35° | `18E4` | `E71C` |
| 40° | `1C72` | `E38E` |
| 45° | `2000` | `E000` |
| 50° | `238E` | `DC72` |
| 55° | `271C` | `D8E4` |
| 60° | `2AAB` | `D555` |
| 65° | `2E39` | `D1C7` |
| 70° | `31C7` | `CE39` |
| 75° | `3555` | `CAAB` |
| 80° | `38E4` | `C71C` |
| 85° | `3C72` | `C38E` |
| 88° (game default) | `3E94` | `C16C` |

### Down limit on land

Edit the **last four digits** of this line, using a **down value** from the table:

```
01BFEA54 3980D8E4
```

| Land down limit | Line |
|---|---|
| 45° | `01BFEA54 3980E000` |
| 50° | `01BFEA54 3980DC72` |
| 55° (default) | `01BFEA54 3980D8E4` |
| 60° | `01BFEA54 3980D555` |

> **Why not 88° on land?** The land camera already sits above the hunter looking down, and the tilt is added on top of that. Once the starting angle plus the tilt passes 90°, the camera goes over the top of the hunter and the view flips. If you see the flip, lower this value; if the camera stops too early, raise it.

### Down limit underwater

Edit the last four digits of this line, using a **down value**:

```
01BFEA4C 3980C16C
```

Example (60° underwater): `01BFEA4C 3980D555`

### Up limit (land and underwater)

The up limit is shared by land and underwater. Add these two lines to the cheat, putting the same **up value** in both:

```
022869F8 2C0B____
02286A0C 3AE0____
```

Examples:

| Up limit | Lines |
|---|---|
| 60° | `022869F8 2C0B2AAB` / `02286A0C 3AE02AAB` |
| 75° | `022869F8 2C0B3555` / `02286A0C 3AE03555` |
| 88° (default) | `022869F8 2C0B3E94` / `02286A0C 3AE03E94` |

### Examples of full limit sets

**75° up, 45° down on land, underwater down unchanged:**
```
022869F8 2C0B3555
02286A0C 3AE03555
01BFEA54 3980E000
```

**60° both ways on land, 60° down underwater:**
```
022869F8 2C0B2AAB
02286A0C 3AE02AAB
01BFEA54 3980D555
01BFEA4C 3980D555
```

When you change a line that is already in the cheat (`01BFEA54`, `01BFEA4C`), edit it in place rather than adding a second copy.

## How it works

The camera routine has two branches: land and underwater (the game checks the hunter's state byte, where `3` = swimming).

| Lines | What they do |
|---|---|
| `022868D4`, `0228686C` | Make the land camera run the underwater tilt code instead of the stepped-level code, and remove an input check that stopped it running on land. |
| `01BFEA00`–`01BFEA30`, `0228A6D4` | The land branch only rotated the camera offset by yaw. This adds the pitch rotation the underwater branch does, using the game's own rotate-around-X function (`0205D5F4`), so the tilt value is applied. |
| `01BFEA40`–`01BFEA5C`, `02286A40`, `02286A48` | Replace the fixed down clamp with one that uses `01BFEA54` on land and `01BFEA4C` underwater. |
| `022869F8`, `02286A0C` (optional) | The up clamp: compare value and clamp value. |

Code-cave memory used: `01BFEA00`–`01BFEA5F`. Avoid it in other cheats or graphic packs.

## Credits

Reverse engineered on the MH3U EU v32 executable with Cemu's debugger.

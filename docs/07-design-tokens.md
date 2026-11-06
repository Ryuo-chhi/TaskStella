# 07 Design Tokens

Design tokens are the named values behind the design: colors, text styles, spacing, radii and animation durations. Defining them once in `lib/ui/theme/` keeps every screen consistent and makes purchasable themes possible.

Values come from `TaskStella-Design-Spec.md` sections 1.1 to 1.4. The Figma variables are exported in `tokens/taskstella.variables.json` if you want to compare names.

## Rule

Widgets never contain hex colors, font sizes or pixel values directly. They read them from the theme. If a value is missing, add it to the theme first.

## Colors: name by role, not by color

Name colors by what they do (`primary`, `textSecondary`), not what they look like (`sageGreen`). When the user equips a different theme, the role stays the same and only the value changes.

| Role token     | Cream Meadow value | Use                                              |
| -------------- | ------------------ | ------------------------------------------------ |
| primary        | `#3F6B54`          | Primary buttons, active tab, active date, links  |
| primaryPressed | `#2F5742`          | Pressed primary                                  |
| background     | `#F2F6F3`          | Screen background                                |
| surface        | `#FFFFFF`          | Inputs, sheets, bottom nav, stat cards           |
| textPrimary    | `#25332C`          | Titles, body, text on tint cards                 |
| textSecondary  | `#5B6760`          | Hints, captions (13 sp or larger only)           |
| divider        | `#E8E1D5`          | Hairlines, empty progress track                  |
| tintSage       | `#DCE9D6`          | Hero card, low priority, completed               |
| tintLavender   | `#E6E0F4`          | Focus cards, medium priority, upcoming           |
| tintPeach      | `#F9DFD2`          | Task cards, high priority, overdue               |
| tintButter     | `#FBEBC2`          | Notes, tips, price pills                         |
| accent         | `#6B5FA3`          | Focus accents, selected sound chip               |
| accentOnTint   | `#5E5296`          | Accent text on lavender tint                     |
| alert          | `#A8452F`          | Overdue, high priority, errors, destructive text |
| warning        | `#8A5A00`          | Medium priority text                             |
| reward         | `#F2B84B`          | Coin fill, streak flame. Fill only, never text.  |
| illustration   | `#A9C6A5`          | Hills, leaves                                    |

Each purchasable theme provides the same set of role tokens with its own values, and must pass the contrast table in the spec (section 1.2) before it ships.

## Text styles (Nunito)

| Token        | Size (sp) | Weight                   | Color             |
| ------------ | --------- | ------------------------ | ----------------- |
| hero         | 28        | SemiBold                 | textPrimary       |
| bigNumber    | 56        | Bold                     | textPrimary       |
| sectionTitle | 18        | SemiBold                 | textPrimary       |
| body         | 15        | Regular, line height 1.5 | textPrimary       |
| caption      | 13        | Regular                  | textSecondary     |
| button       | 16        | SemiBold                 | depends on button |
| link         | 13        | SemiBold                 | primary           |

Nothing smaller than 12 sp. Map these onto Flutter's `TextTheme` so widgets can use `Theme.of(context).textTheme`.

## Spacing, radii, motion

| Token                     | Value                                  |
| ------------------------- | -------------------------------------- |
| space4, 8, 12, 16, 24, 32 | 8-point scale                          |
| screenPadding             | 20                                     |
| cardPadding               | 20                                     |
| sectionGap                | 28                                     |
| radiusInput               | 20                                     |
| radiusCard                | 28                                     |
| radiusHero, radiusSheet   | 32                                     |
| radiusPill                | fully rounded                          |
| durationMotion            | 300 ms, ease-in-out (range 250 to 350) |
| minTouchTarget            | 48 dp                                  |

## Theme switching at runtime

1. Each theme in the catalog maps to a full set of role tokens.
2. The root app widget holds the equipped theme id in its `State`.
3. When ShopService equips a new theme, the screen calls a callback that reaches the root, which calls `setState` and rebuilds `MaterialApp` with the new `ThemeData`.
4. Because widgets only read role tokens, every screen updates without changes.

Color values that do not fit `ThemeData` (tints, reward, illustration) can live in a small custom class registered as a theme extension, or in a class passed down with the theme. Pick one approach and document it here.

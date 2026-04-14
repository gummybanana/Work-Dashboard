# PetLog Design System

## Colors

| Token | Hex | Usage |
|---|---|---|
| teal-600 | `#0D9488` | Primary buttons, active nav, links |
| teal-700 | `#0F766E` | Primary hover |
| teal-50 | `#F0FDFA` | Selected/active backgrounds |
| orange-500 | `#F97316` | Warnings, overdue badges |
| orange-50 | `#FFF7ED` | Warning backgrounds |
| red-500 | `#EF4444` | Destructive buttons, error text |
| red-50 | `#FEF2F2` | Error backgrounds |
| green-500 | `#22C55E` | Success, up-to-date badges |
| green-50 | `#F0FDF4` | Success backgrounds |
| gray-900 | `#111827` | Headings |
| gray-700 | `#374151` | Body text |
| gray-500 | `#6B7280` | Secondary text, placeholders |
| gray-200 | `#E5E7EB` | Borders, dividers |
| gray-50 | `#F9FAFB` | Page background |
| white | `#FFFFFF` | Card backgrounds |

## Typography

- **Font:** `"Inter", sans-serif`
- **H1:** 24px, bold, gray-900
- **H2:** 18px, semibold, gray-900
- **H3:** 16px, semibold, gray-900
- **Body:** 14px, regular, gray-700
- **Small:** 12px, regular, gray-500
- **Mono:** `"JetBrains Mono", monospace` — used for weights and dates in tables

## Spacing

4px base. Use: 4, 8, 12, 16, 24, 32, 48px.

## Border Radius

- Buttons/inputs: 6px
- Cards: 8px
- Badges: 9999px (pill)
- Avatars: 9999px (circle)

## Shadows

- Cards: `0 1px 3px rgba(0,0,0,0.1)`
- Modals: `0 10px 25px rgba(0,0,0,0.15)`

---

## Components

### Button
| Variant | BG | Text | Hover BG | Border |
|---|---|---|---|---|
| Primary | teal-600 | white | teal-700 | none |
| Secondary | white | gray-700 | gray-50 | 1px gray-200 |
| Destructive | red-500 | white | `#DC2626` | none |
| Ghost | transparent | gray-700 | gray-50 | none |

Height: 36px. Padding: 12px horizontal. Font: 14px medium. Radius: 6px.
Disabled: opacity 0.5, no pointer events.

### Input
Height: 36px. Border: 1px gray-200. Radius: 6px. Padding: 8px 12px.
Focus: border teal-600. Error: border red-500 + error message below in Small red-500.
Label: 14px medium gray-700, 4px gap above input.

### Card
Background: white. Border: 1px gray-200. Radius: 8px. Shadow: card shadow. Padding: 16px.

### Badge
Height: 22px. Padding: 2px 10px. Radius: pill. Font: 12px medium.
Variants: green (green-50 bg, green-700 text), orange (orange-50 bg, orange-700 text), red (red-50 bg, red-700 text), gray (gray-100 bg, gray-600 text).

### Modal
Overlay: `rgba(0,0,0,0.4)`. Container: white, 8px radius, modal shadow, max-width 480px, padding 24px.
Header: H2 + close button (X icon, Ghost). Footer: right-aligned buttons, 8px gap.

### Tab Bar
Horizontal tabs. Inactive: gray-500 text, no border. Active: teal-600 text, 2px teal-600 bottom border. Height: 40px. Font: 14px medium.

### Pet Avatar
Circle, 48px (list) or 80px (detail). Background: teal-50. Text: teal-600, 20px bold (initials of pet name, first 2 letters uppercase).

### Empty State
Centered in parent. Icon: 48px, gray-400. Heading: H3, gray-500. Description: Body, gray-500. CTA button below if specified.

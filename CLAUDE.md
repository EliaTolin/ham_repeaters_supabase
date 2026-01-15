# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HamQRG is a Supabase backend for a ham radio repeater database. It stores repeater stations with their frequencies, access modes (analog, DMR, C4FM, D-STAR, etc.), geographic locations, and user feedback.

## Common Commands

```bash
# Start local Supabase development environment
supabase start

# Stop local environment
supabase stop

# Apply migrations to local database
supabase db reset

# Create a new migration
supabase migration new <migration_name>

# Push migrations to remote (production)
supabase db push

# Generate TypeScript types from schema
supabase gen types typescript --local > types/supabase.ts
```

## Architecture

### Database Schema

The schema uses PostgreSQL with PostGIS for geographic queries.

**Core Tables:**
- `repeaters` - Radio repeater stations with frequency, location (lat/lon or Maidenhead locator), and mode (Analog/Digital/Mixed)
- `repeater_access` - Access configurations per repeater (CTCSS tones, DCS codes, DMR color codes, network affiliations)
- `networks` - Named networks (DMR, C4FM, D-STAR, VoIP) that repeaters can belong to
- `repeater_feedback` - User likes/down reports with geographic position

**Key Enums:**
- `repeater_mode`: Analog, Digital, Mixed
- `access_mode`: ANALOG, DMR, C4FM, DSTAR, ECHOLINK, SVX, APRS, BEACON, ATV
- `network_kind`: dmr, c4fm, dstar, voip, mixed, other

### Geographic Functions

Two main spatial query functions are exposed via the API:

- `repeaters_nearby(lat, lon, radius_km, limit, access_modes)` - Find repeaters within radius, returns distance
- `repeaters_in_bounds(lat1, lon1, lat2, lon2, access_modes)` - Find repeaters in bounding box

Both functions return repeater data with aggregated `accesses` JSONB containing all access configurations.

### Row Level Security

All tables use RLS. Only authenticated users can read data. Users can only modify their own feedback entries.

## Conventions

- Migrations are timestamped SQL files in `supabase/migrations/`
- The `geom` column is auto-generated from lat/lon coordinates
- Maidenhead locators are automatically converted to coordinates if lat/lon not provided
- Frequencies are stored in Hz (`frequency_hz`), shifts in Hz (`shift_hz`)
- CTCSS tones stored as numeric(6,1) in Hz

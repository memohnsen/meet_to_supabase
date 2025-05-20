# Meet to Supabase

This project automatically syncs weightlifting meets data from USA Weightlifting to a Supabase database. It runs daily as a GitHub Action, fetching the latest meet data from the USA Weightlifting API and inserting new meets into your Supabase database.

## Features

- Scrapes meet data from USA Weightlifting API
- Checks if meets already exist in Supabase before inserting
- Intelligently parses complex address formats and date information
- Maps US states to appropriate time zones
- Handles escaped JSON characters in the API response
- Includes retry mechanism with exponential backoff
- Runs daily via GitHub Actions

## Setup

### Prerequisites

- Node.js 14+ installed
- A Supabase account with a project set up
- A GitHub account for Actions

### Local Development

1. Clone this repository:
   ```
   git clone <your-repo-url>
   cd meet_to_supabase
   ```

2. Create a `.env` file with your Supabase credentials:
   ```
   SUPABASE_URL=your_supabase_url
   SUPABASE_KEY=your_supabase_service_role_key
   ```
   
   Note: You need a service role key with write permissions to the `meets` table.

3. Install dependencies:
   ```
   npm install
   ```

### GitHub Actions Setup

1. Push this repository to GitHub
2. Go to your GitHub repository Settings > Secrets and variables > Actions
3. Add the following repository secrets:
   - `SUPABASE_URL`: Your Supabase project URL
   - `SUPABASE_KEY`: Your Supabase service role key

## Usage

### Running Locally

```
npm run sync
```

### GitHub Actions

The script will automatically run daily at 2:00 AM UTC via GitHub Actions. You can also trigger the workflow manually:

1. Go to the Actions tab in your GitHub repository
2. Select the "Daily Meets Sync" workflow
3. Click "Run workflow"

## Supabase Schema

The script is designed to work with the following Supabase table schema:

```sql
create table public.meets (
  id uuid not null default gen_random_uuid (),
  name text not null,
  venue_name text not null,
  venue_street text not null,
  venue_city text not null,
  venue_state text not null,
  venue_zip text not null,
  time_zone text not null,
  start_date date not null,
  end_date date not null,
  status public.meet_status not null default 'upcoming'::meet_status,
  created_at timestamp with time zone null default now(),
  updated_at timestamp with time zone null default now(),
  constraint meets_pkey primary key (id),
  constraint meets_name_key unique (name)
) TABLESPACE pg_default;

-- This enum type must be created first
CREATE TYPE public.meet_status AS ENUM ('upcoming', 'in_progress', 'completed', 'cancelled');

-- This trigger updates the updated_at column automatically
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
   NEW.updated_at = now();
   RETURN NEW;
END;
$$ language 'plpgsql';
```

## Troubleshooting

- **API Response Format Changes**: If the USA Weightlifting API changes its response format, you may need to update the parsing logic in `scripts/sync-meets.js`.
- **GitHub Actions Not Running**: Check the Actions tab for any error logs and make sure your secrets are correctly set up.
- **Database Errors**: Verify your Supabase credentials and that the `meets` table exists with the correct schema.
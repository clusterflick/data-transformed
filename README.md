# data-transformed

This repository contains the automated workflow for transforming raw cinema data
into a standardized format for the Clusterflick project.

## Purpose

The transformation workflow processes raw cinema event data retrieved from
various cinema websites and converts it into a unified, standardized cinema data
format. This includes:

- **Data normalization**: Converting venue-specific data formats into a
  consistent schema
- **Movie matching**: Attempting to match each event to a movie in The Movie
  Database (TMDB)
- **Categorization**: When movie matching fails, events are categorized
  appropriately
- **Quality assurance**: Ensuring data integrity and consistency across all
  venues

## How It Works

Each job downloads the latest raw data from the `clusterflick/data-retrieved`
repository's most recent release, then executes the transform command for each
venue:

```bash
npx clusterflick/scripts transform <venue-identifier>
```

This command:

- Reads the raw data for the specified venue
- Normalizes the data structure
- Attempts to match events to movies in TMDB using the Movie Database API
- Categorizes events that cannot be matched to movies
- Generates standardized output files

## Schedule

The workflow is automatically triggered when the
[data retrieval workflow](https://github.com/clusterflick/data-retrieved)
completes successfully. It can also be triggered manually via workflow dispatch
if needed.

## Workflow Structure

Transform commands are organized into parallel job groups to:

- Minimize total wall clock time
- Prevent cascading failures (one command failure won't stop others)
- Enable individual job restarts when issues occur

## Maintenance

### Adding New Venues

When a new venue is added to the system, this workflow **must be updated** to
include the transformation step. To add a venue:

1. Identify the appropriate job group (or create a new one if needed)
2. Add a new step to run the transform command for the venue:
   ```yaml
   - name: venue-identifier
     run: npx clusterflick/scripts transform venue-identifier
   ```
3. Ensure the job downloads the correct raw data files from the retrieval
   repository

### Dependencies

The workflow requires several API keys configured as GitHub secrets:

- `MOVIEDB_API_KEY` - For matching events to movies in The Movie Database
- `GEMINI_API_KEY` - For AI-powered categorization when movie matching fails
- `PAT` - Personal Access Token for triggering downstream workflows

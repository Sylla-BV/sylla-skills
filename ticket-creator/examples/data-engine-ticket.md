# Example: Data-Engine Ticket (Harvester Implementation)

> Based on real ticket S-1149 (Base Harvester Actor Class), enhanced with example API response and field mapping. Shown as a source-specific harvester to demonstrate the copy-adapt pattern.

---

## Summary

Implement the Pressbooks harvester actor that fetches open textbook metadata from the Pressbooks Algolia index, downloads PDFs/EPUBs, and ingests them into the Sylla platform via the base harvester pattern.

## Context

Pressbooks is one of four OER sources that Sylla harvests. This harvester follows the base harvester pattern established in S-1149 and uses Algolia as the search API (similar to how OAPEN uses their API).

**Project:** [Automated Book Reharvesting Tools](https://linear.app/sylla/project/automated-book-reharvesting-tools)

**Dependencies:**
- Blocked by: S-1149 (Base Harvester Actor Class — provides `BaseHarvester` to extend)
- Blocked by: S-1148 (Shared Harvesting Models — provides `HarvestedBook` model)

---

## Requirements

- Extend `BaseHarvester` for the Pressbooks source
- Fetch book metadata from Pressbooks Algolia index
- Map Algolia response fields to `HarvestedBook` model
- Handle pagination through Algolia search results
- Download PDFs and EPUBs when available
- Use delta detection to skip already-harvested books
- Produce structured outcome logs in S3

---

## Technical Context

**Repo:** data-engine

**Existing pattern to follow:**
- Base class: `ingestion-actors/worker/harvesting/base_harvester.py` (`BaseHarvester`) — extend this class, implement `_get_source_id()` and the fetch method
- Similar harvester: `ingestion-actors/worker/harvesting/oapen_harvester.py` — follow this as the closest reference for Algolia-based fetching

**Systems involved:** Algolia (Pressbooks index), S3, SQS/Dramatiq, NextJS API (book insertion)

**New/modified files:**
- `ingestion-actors/worker/harvesting/pressbooks_harvester.py` — Harvester implementation (NEW)
- `ingestion-actors/worker/harvesting/tests/test_pressbooks_harvester.py` — Tests (NEW)

---

## Data-Engine Specification

**Actor type:** Harvester (extends `BaseHarvester`)

**Base pattern:** `ingestion-actors/worker/harvesting/base_harvester.py`

**Source URL / API endpoint:**
- Algolia App ID: `PRESSBOOKSDIR`
- Algolia Index: `pressbooks_directory`
- API: `https://PRESSBOOKSDIR-dsn.algolia.net/1/indexes/pressbooks_directory`

**Example API response:**

```json
{
  "hits": [
    {
      "objectID": "12345",
      "title": "Introduction to Psychology",
      "author": "Dr. Jane Smith",
      "publisher": "Pressbooks",
      "publicationDate": "2024-01-15",
      "isbn": "978-1-234567-89-0",
      "doi": null,
      "description": "A comprehensive introduction to psychology for undergraduate students.",
      "license": "CC BY 4.0",
      "language": "en",
      "subjects": ["Psychology", "Social Sciences"],
      "coverImage": "https://pressbooks.directory/covers/12345.jpg",
      "urls": {
        "pdf": "https://open.umn.edu/opentextbooks/files/intro-psychology.pdf",
        "epub": "https://open.umn.edu/opentextbooks/files/intro-psychology.epub",
        "webbook": "https://pressbooks.pub/intro-psychology"
      },
      "lastUpdated": "2024-06-01T00:00:00Z"
    }
  ],
  "nbHits": 1523,
  "page": 0,
  "nbPages": 76,
  "hitsPerPage": 20
}
```

**Fields to extract:**

| Source Field | Maps To | Notes |
|-------------|---------|-------|
| `hit.objectID` | `source_id` | Used for delta detection |
| `hit.title` | `HarvestedBook.title` | |
| `hit.author` | `HarvestedBook.author` | May be string or array |
| `hit.publisher` | `HarvestedBook.publisher` | |
| `hit.isbn` | `HarvestedBook.isbns[]` | Parse into list, may be null |
| `hit.doi` | `HarvestedBook.doi` | May be null |
| `hit.description` | `HarvestedBook.description` | |
| `hit.license` | `HarvestedBook.license` | |
| `hit.language` | `HarvestedBook.language` | |
| `hit.subjects` | `HarvestedBook.subjects[]` | Array of strings |
| `hit.coverImage` | `HarvestedBook.image_url` | Download via base class |
| `hit.urls.pdf` | `HarvestedBook.pdf_url` | Download via base class |
| `hit.urls.epub` | `HarvestedBook.epub_url` | Download via base class |
| `hit.lastUpdated` | `HarvestedBook.last_updated` | Parse ISO datetime |

**External API documentation:**
- Algolia Search API: https://www.algolia.com/doc/rest-api/search/#search-index-post
- Use `algoliasearch` Python client v3

**Library/version:** `algoliasearch>=3.0.0`

---

## Acceptance Criteria

- [ ] `PressbooksHarvester` extends `BaseHarvester`
- [ ] Fetches all books from Pressbooks Algolia index with pagination
- [ ] Maps all fields from Algolia response to `HarvestedBook` model
- [ ] Handles missing optional fields gracefully (null isbn, null doi, null epub)
- [ ] `_get_source_id()` returns `objectID`
- [ ] Downloads PDFs, EPUBs, and cover images via base class methods
- [ ] Delta detection skips already-harvested books
- [ ] Outcome files written to S3
- [ ] Per-book failures don't crash the entire harvest
- [ ] `mypy` passes

---

## Verification

- [ ] Unit test: field mapping converts Algolia hit to `HarvestedBook` correctly (use example response above)
- [ ] Unit test: handles missing optional fields (null isbn, null epub_url)
- [ ] `mypy` passes

---

## Out of Scope

- TOC extraction from Pressbooks PDFs (S-1153)
- PDF validation (S-1156)
- Handling Pressbooks API rate limits (their Algolia tier is generous)

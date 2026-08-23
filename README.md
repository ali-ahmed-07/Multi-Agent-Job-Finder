# Resume → Job Matcher

An AI-powered multi-agent job discovery system that analyzes a candidate's resume, extracts relevant skills and keywords, generates optimized job-search queries, discovers relevant job openings through Tavily, extracts job information from public job pages, and presents the results in a structured format.

The system is designed around a resume-first workflow, meaning the user does not need to provide a separate job description. Candidate skills, experience, role, and search criteria are derived directly from the uploaded resume.

---

## Overview

The Resume → Job Matcher follows a multi-stage pipeline:

```text
Resume Upload
      ↓
Resume Text Extraction
      ↓
Resume Analysis
      ↓
Skill & Keyword Extraction
      ↓
Search Query Generation
      ↓
Web Job Search
      ↓
Job URL Filtering
      ↓
Job Page Scraping
      ↓
Job Data Extraction
      ↓
Job Data Processing
      ↓
Final Job Results
```

The project uses Google Gemini for intelligent resume analysis and extraction, Tavily for web search, and deterministic Python processing for normalization, deduplication, sorting, filtering, and result preparation.

The architecture is designed to minimize unnecessary LLM/API usage by using AI only where semantic understanding is required and using Python-based processing for deterministic operations.

---

## Key Features

* Upload resumes in PDF or DOCX format
* Extract resume text automatically
* Analyze candidate experience and technical skills
* Identify the candidate's most suitable job role
* Extract relevant technical and professional keywords
* Extract programming languages, frameworks, libraries, databases, and tools
* Generate optimized job-search queries from extracted resume information
* Search the web using Tavily
* Discover real job posting URLs
* Focus scraping on relevant job pages
* Scrape publicly accessible job pages
* Extract:

  * Job title
  * Company
  * Location
  * Employment type
  * Short job description
  * Posting date
  * Job URL
* Normalize extracted job information
* Remove duplicate job postings
* Sort jobs by reliable posting date
* Handle unavailable posting dates safely
* Filter irrelevant or incomplete results
* Limit the number of jobs processed
* Prepare structured results for Streamlit
* Minimize unnecessary LLM token usage
* Reduce unnecessary scraping and processing
* Separate AI-based tasks from deterministic Python processing

---

## Resume Analyzer & Keyword Extraction

The Resume Analyzer is one of the most important stages of the system.

A resume normally contains unstructured information such as work experience, technical skills, projects, education, tools, and job responsibilities. Instead of passing the complete resume directly into every stage, the Analyzer converts the resume into a structured candidate profile.

The Analyzer identifies information such as:

* Primary job role
* Technical skills
* Programming languages
* Frameworks
* Libraries
* Databases
* Development tools
* Cloud technologies
* Years of experience
* Industry/domain experience
* Education
* Professional experience
* Relevant job-search keywords

The extracted information is then used to generate focused search queries.

```text
Resume
   │
   ▼
Resume Text
   │
   ▼
Analyzer
   │
   ├── Job Role
   ├── Technical Skills
   ├── Experience
   ├── Frameworks
   ├── Databases
   ├── Tools
   └── Relevant Keywords
            │
            ▼
     Search Query Generation
            │
            ▼
        Tavily Search
            │
            ▼
       Job Posting URLs
```

This approach makes the search and scraping process more targeted because the system does not need to repeatedly process the complete resume.

---

## Keyword Extraction Process

The keyword extraction process converts important resume information into searchable terms.

For example, if a candidate's resume contains:

```text
Python
FastAPI
Django
MySQL
REST API
Docker
Git
```

and the identified role is:

```text
Python Developer
```

the system can generate focused search queries such as:

```text
Python Developer FastAPI
Python Developer Django
Python Developer REST API
Python Developer FastAPI MySQL
Python Developer Docker
```

These queries are then used to discover relevant job postings.

The important point is that the system does not blindly search the entire resume. It first identifies the information that is useful for job discovery.

---

## Why Keyword Extraction Improves Scraping

Keyword extraction improves the overall job discovery and scraping workflow in several ways.

### 1. More Relevant Search Results

The extracted role and technical keywords allow Tavily to search for jobs that are more closely related to the candidate's profile.

Instead of searching broadly for:

```text
developer jobs
```

the system can search more specifically for:

```text
Python Developer FastAPI Django
```

This increases the probability of discovering relevant job postings.

### 2. Less Unnecessary Scraping

The system first discovers job URLs through targeted search queries.

Only the discovered and relevant URLs are passed to the scraping stage.

```text
Resume
   ↓
Keyword Extraction
   ↓
Targeted Search
   ↓
Relevant Job URLs
   ↓
Scraping
```

This prevents the scraper from unnecessarily processing unrelated pages.

### 3. Lower Processing Cost

Because the search is focused before scraping begins, fewer pages need to be processed.

This reduces:

* Unnecessary HTTP requests
* Unnecessary page processing
* Unnecessary LLM calls
* Processing time
* API usage

### 4. Lower LLM Token Usage

The system uses Google Gemini primarily for tasks that require semantic understanding.

After the AI extracts the relevant information, Python handles deterministic operations such as:

* Filtering
* Normalization
* Deduplication
* Sorting
* Date handling
* Result limiting

This prevents the LLM from being used for repetitive operations that can be handled more efficiently with Python.

---

## AI vs Python Processing

The project intentionally separates intelligent AI operations from deterministic Python operations.

### Google Gemini

Gemini is used where language understanding is required:

* Resume analysis
* Candidate profile generation
* Skill extraction
* Keyword extraction
* Job role identification
* Search query generation
* Intelligent job information extraction when required

### Python

Python is used for deterministic and repetitive operations:

* Resume file handling
* URL processing
* Data normalization
* Duplicate removal
* Date processing
* Job filtering
* Result sorting
* Result limiting
* Data validation
* Structured result preparation

This separation helps reduce unnecessary API usage and improves the efficiency of the overall pipeline.

---

## Architecture

```text
                         Resume
                           │
                           ▼
                 ┌──────────────────┐
                 │ Resume Reader    │
                 │ PDF / DOCX       │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Analyzer Chain   │
                 │ Google Gemini    │
                 └────────┬─────────┘
                          │
                  Candidate Profile
                          │
              ┌───────────┼───────────┐
              │           │           │
          Job Role     Skills      Keywords
              │           │           │
              └───────────┼───────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Query Generator  │
                 │ Targeted Search  │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Job Search Agent │
                 │ Tavily           │
                 └────────┬─────────┘
                          │
                     Job URLs
                          │
                          ▼
                 ┌──────────────────┐
                 │ Job Scraper      │
                 │ Agent + Tool     │
                 └────────┬─────────┘
                          │
                   Structured Jobs
                          │
                          ▼
               ┌──────────────────────┐
               │ Job Result Processor │
               │ Python               │
               └──────────┬───────────┘
                          │
             ┌────────────┼────────────┐
             │            │            │
          Normalize   Deduplicate    Filter
             │            │            │
             └────────────┼────────────┘
                          │
                          ▼
                     Sort Results
                          │
                          ▼
                    Final Results
                          │
                          ▼
                    Streamlit UI
```

---

## Complete Workflow

### Step 1: Resume Upload

The user uploads a resume through the Streamlit interface.

Supported formats:

* PDF
* DOCX

The system does not require the user to manually enter a job description.

---

### Step 2: Resume Text Extraction

The Resume Reader extracts readable text from the uploaded document.

The extracted text becomes the input for the Analyzer Chain.

```text
PDF / DOCX
    ↓
Resume Reader
    ↓
Plain Resume Text
```

---

### Step 3: Resume Analysis

The Analyzer Chain uses Google Gemini to understand the resume.

It identifies:

```text
Candidate Role
Technical Skills
Experience
Tools
Frameworks
Databases
Education
Relevant Keywords
```

The result is converted into a structured candidate profile.

---

### Step 4: Keyword Extraction

The system extracts the most useful terms for job discovery.

For example:

```text
Role:
Python Developer

Skills:
Python
FastAPI
Django
MySQL
REST API
Docker
Git
```

These keywords become the foundation of the search queries.

---

### Step 5: Search Query Generation

The extracted candidate information is converted into optimized search queries.

Example:

```text
Python Developer FastAPI
Python Developer Django
Python Developer REST API
Python Developer MySQL
Python Developer Docker
```

The goal is to generate queries that are specific enough to discover relevant opportunities without creating unnecessary search requests.

---

### Step 6: Web Job Search

The generated queries are sent to Tavily.

Tavily searches publicly available web content and returns potentially relevant results.

The system extracts job posting URLs from those search results.

```text
Search Query
     ↓
Tavily
     ↓
Search Results
     ↓
Job URLs
```

---

### Step 7: Job URL Filtering

Before scraping, the system can filter and validate discovered URLs.

This helps avoid processing:

* Duplicate URLs
* Invalid URLs
* Non-job pages
* Unrelated search results
* Already processed pages

This stage helps reduce unnecessary scraping.

---

### Step 8: Job Page Scraping

The scraper processes publicly accessible job pages.

The goal is to extract structured job information from the page.

Typical fields include:

```text
Job Title
Company
Location
Employment Type
Description
Posting Date
URL
```

The scraping stage is intentionally limited to relevant URLs discovered through the search process.

---

### Step 9: Job Data Processing

After scraping, the raw job results are passed to the Job Result Processor.

Python handles deterministic processing such as:

```text
Raw Jobs
   ↓
Normalize
   ↓
Validate
   ↓
Deduplicate
   ↓
Filter
   ↓
Sort
   ↓
Limit Results
```

This stage does not require an LLM for normal operations.

---

## Deduplication

The system removes duplicate job postings so that the user does not receive the same opportunity multiple times.

Duplicate detection can be based on normalized information such as:

* Job URL
* Job title
* Company
* Location

This helps produce a cleaner final result set.

---

## Date Processing

Job posting dates are handled carefully.

If a reliable posting date is available, the system can use it for sorting and filtering.

If the date is unavailable or cannot be parsed safely, the system handles it without causing the entire pipeline to fail.

This allows incomplete job pages to remain usable instead of breaking the complete workflow.

---

## Result Limiting

The system limits the number of jobs processed and returned.

This provides several benefits:

* Faster processing
* Lower API usage
* Less scraping
* Smaller result sets
* Better Streamlit performance
* Lower unnecessary token usage

---

## LLM/API Optimization

A major goal of the project is to minimize unnecessary AI/API usage.

The system follows an AI-first-where-needed approach rather than using Gemini for every operation.

```text
                AI Tasks
                   │
       ┌───────────┴───────────┐
       │                       │
 Resume Understanding     Job Understanding
       │                       │
       └───────────┬───────────┘
                   │
                   ▼
            Structured Data
                   │
                   ▼
          Python Processing
                   │
       ┌───────────┼───────────┐
       │           │           │
    Filter      Sort      Deduplicate
       │           │           │
       └───────────┼───────────┘
                   │
                   ▼
             Final Results
```

This architecture avoids using an LLM for simple operations that can be completed deterministically.

---

## Project Structure

```text
resume-job-matcher/
│
├── README.md
├── requirements.txt
├── requirements-dev.txt
├── .env
├── .env.example
├── .gitignore
│
├── data/
│   └── sample_resumes/
│
├── src/
│   └── resume_job_matcher/
│       │
│       ├── __init__.py
│       ├── config.py
│       ├── pipeline.py
│       │
│       ├── tools/
│       │   ├── __init__.py
│       │   ├── resume_reader.py
│       │   └── scrape_url.py
│       │
│       ├── chains/
│       │   ├── __init__.py
│       │   └── analyzer.py
│       │
│       ├── agents/
│       │   ├── __init__.py
│       │   ├── job_search_agent.py
│       │   └── job_scraper_agent.py
│       │
│       └── processors/
│           ├── __init__.py
│           └── job_result_processor.py
│
├── app.py
│
└── tests/
    ├── __init__.py
    └── tools/
        ├── __init__.py
        └── test_resume_reader.py
```

---

## Main Components

### Resume Reader

Responsible for:

* Reading PDF files
* Reading DOCX files
* Extracting resume text
* Providing clean text to the Analyzer

### Analyzer Chain

Responsible for:

* Understanding resume content
* Identifying candidate role
* Extracting skills
* Extracting keywords
* Identifying experience
* Creating a structured candidate profile

### Job Search Agent

Responsible for:

* Creating search queries
* Sending queries to Tavily
* Discovering relevant job opportunities
* Collecting job posting URLs

### Job Scraper Agent

Responsible for:

* Accessing publicly available job pages
* Extracting job information
* Converting raw page information into structured data

### Job Result Processor

Responsible for:

* Normalization
* Validation
* Deduplication
* Filtering
* Date processing
* Sorting
* Result limiting

### Streamlit UI

Responsible for:

* Resume upload
* Running the job discovery pipeline
* Displaying candidate analysis
* Displaying discovered jobs
* Presenting structured job results

---

## Technologies Used

* Python
* Streamlit
* Google Gemini
* LangChain
* Tavily
* PDF/DOCX processing libraries
* Web scraping utilities
* Deterministic Python data processing

---

## Environment Variables

Create a `.env` file based on `.env.example`.

Example:

```env
GOOGLE_API_KEY=your_google_api_key
TAVILY_API_KEY=your_tavily_api_key
```

Never commit real API keys to GitHub.

The `.env` file should remain excluded through `.gitignore`.

---

## Installation

Clone the repository:

```bash
git clone https://github.com/ali-ahmed-07/Multi-Agent-Job-Finder.git
cd Multi-Agent-Job-Finder
```

Create a virtual environment:

```bash
python -m venv venv
```

Activate the environment on Windows:

```powershell
.\venv\Scripts\Activate.ps1
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Create the environment configuration:

```text
.env
```

Add the required API keys.

---

## Running the Application

Run the Streamlit application:

```bash
streamlit run app.py
```

The application will open the Streamlit interface where the user can upload a resume and start the job discovery workflow.

---

## Testing

Run the available tests using:

```bash
pytest
```

For the resume reader specifically:

```bash
pytest tests/tools/test_resume_reader.py
```

---

## Data Flow

The complete data flow can be summarized as:

```text
User
 │
 │ Upload Resume
 ▼
Resume Reader
 │
 │ Extracted Text
 ▼
Analyzer
 │
 ├── Candidate Role
 ├── Skills
 ├── Experience
 └── Keywords
 │
 ▼
Query Generator
 │
 │ Optimized Queries
 ▼
Tavily
 │
 │ Search Results
 ▼
Job URL Filtering
 │
 │ Relevant URLs
 ▼
Job Scraper
 │
 │ Raw Job Data
 ▼
Job Result Processor
 │
 ├── Normalize
 ├── Validate
 ├── Deduplicate
 ├── Filter
 ├── Sort
 └── Limit
 │
 ▼
Streamlit
 │
 ▼
Final Job Results
```

---

## Design Goals

The project is designed around the following goals:

### Resume-First Search

The candidate does not need to provide a separate job description. The resume itself becomes the source of search criteria.

### Targeted Search

Skills, role information, and extracted keywords are used to generate focused search queries.

### Efficient Scraping

Only relevant job URLs discovered through the targeted search process are passed to the scraper.

### Minimal LLM Usage

Gemini is used for semantic tasks while deterministic operations are handled by Python.

### Structured Results

Raw search and scraping results are transformed into consistent structured job records.

### Reliable Processing

The pipeline is designed to handle duplicate URLs, missing dates, incomplete information, and invalid results without unnecessarily breaking the complete workflow.

---

## Error Handling

The pipeline is designed to handle common problems such as:

* Invalid resume files
* Unsupported file formats
* Empty resume text
* Missing API keys
* Search failures
* Invalid URLs
* Inaccessible job pages
* Missing job fields
* Missing posting dates
* Duplicate job postings
* Scraping failures

Where possible, individual failures are handled without stopping the entire job discovery process.

---

## API Usage Optimization Strategy

The project specifically focuses on reducing unnecessary LLM/API usage.

The optimization strategy includes:

1. Analyze the resume once.
2. Extract reusable skills and keywords.
3. Generate targeted search queries.
4. Avoid sending the complete resume repeatedly.
5. Limit the number of search queries.
6. Limit the number of job URLs processed.
7. Scrape only relevant discovered pages.
8. Use Python for filtering and normalization.
9. Use Python for deduplication and sorting.
10. Avoid LLM calls for deterministic operations.

This approach allows the system to maintain intelligent resume understanding while reducing unnecessary token consumption and API dependency.

---

## Example Workflow

A candidate uploads a resume containing:

```text
Python Developer

Skills:
Python
FastAPI
Django
MySQL
Docker
Git
REST API

Experience:
2+ years
```

The Analyzer generates a structured profile:

```text
Role:
Python Developer

Skills:
Python
FastAPI
Django
MySQL
Docker
Git
REST API

Experience:
2+ years
```

The system then generates targeted queries:

```text
Python Developer FastAPI
Python Developer Django
Python Developer MySQL
Python Developer REST API
Python Developer Docker
```

Tavily discovers relevant job URLs.

The scraper processes those URLs and extracts structured job information.

The Job Result Processor then:

```text
Raw Results
    ↓
Normalize
    ↓
Remove Duplicates
    ↓
Filter Invalid Results
    ↓
Process Dates
    ↓
Sort
    ↓
Limit
    ↓
Final Jobs
```

The final structured jobs are displayed through Streamlit.

---

## Privacy & Security

The application should not expose sensitive resume information unnecessarily.

API credentials must be stored in environment variables and must not be committed to the repository.

Recommended files:

```text
.env
.env.example
.gitignore
```

The `.env` file should contain real credentials locally, while `.env.example` should contain only placeholder values.

---

## Future Improvements

Potential future improvements include:

* Advanced resume-to-job similarity scoring
* Semantic job ranking
* More job-source integrations
* Better date extraction
* Additional document formats
* Job application tracking
* Saved searches
* User preference filtering
* Location-based filtering
* Salary-based filtering
* Experience-level matching
* Automated job alerts
* More advanced keyword weighting

---

## Conclusion

Resume → Job Matcher provides a resume-first approach to automated job discovery.

The system combines:

```text
Resume Understanding
        +
Keyword Extraction
        +
Targeted Web Search
        +
Relevant URL Discovery
        +
Focused Job Scraping
        +
Python-Based Processing
        +
Structured Results
```

The main optimization principle is to use AI where semantic understanding is required and deterministic Python processing everywhere else.

By extracting the candidate's role, skills, experience, and relevant keywords before searching, the system can perform more targeted searches and pass fewer irrelevant URLs to the scraping stage.

This results in a more efficient workflow with reduced scraping overhead, reduced unnecessary API usage, lower LLM token consumption, and cleaner final job results.

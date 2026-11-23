# JobFinder - Modern Job Search Application

A visually stunning, production-grade job search application built with Vue 3 and Vite, powered by the JSearch API.

## Features

- 🎨 **Premium UI/UX**: Clean, modern design with glassmorphism and smooth animations
- 🔍 **Advanced Search**: Search by job title, keywords, and location
- 🎯 **Smart Filters**: Filter by date posted, employment type, and remote work
- 📱 **Fully Responsive**: Mobile-first design with adaptive layouts
- ⚡ **Fast Performance**: Optimized with Vite and lazy loading
- 💾 **Persistent Search**: Remembers your last search using localStorage
- 🎭 **Loading States**: Beautiful skeleton loaders and empty states

## Tech Stack

- **Frontend**: Vue 3 (Composition API), Vite, Tailwind CSS
- **Backend**: Vercel Serverless Functions
- **API**: JSearch API (RapidAPI)

## Setup

### Prerequisites

- Node.js 16+
- npm or yarn
- RapidAPI account with JSearch API access

### Installation

1. Clone the repository:
```bash
git clone <your-repo-url>
cd Jsearch
```

2. Install dependencies:
```bash
npm install
```

3. Create a `.env` file in the root directory:
```bash
cp .env.example .env
```

4. Add your RapidAPI key to `.env`:
```
RAPIDAPI_KEY=your_rapidapi_key_here
```

### Development

Run the development server:
```bash
npm run dev
```

The app will be available at `http://localhost:5173`

### Build

Build for production:
```bash
npm run build
```

Preview the production build:
```bash
npm run preview
```

## Deployment

See **[DEPLOYMENT.md](DEPLOYMENT.md)** for Vercel setup, env vars, and rollback.

## Deployment to Vercel

1. Install Vercel CLI:
```bash
npm install -g vercel
```

2. Deploy:
```bash
vercel
```

3. Set environment variable in Vercel dashboard:
   - Go to your project settings
   - Add `RAPIDAPI_KEY` with your RapidAPI key value

## Environment Variables

- `RAPIDAPI_KEY`: Your RapidAPI key for JSearch API access

## API Configuration

The app uses the following default settings:
- **Country**: South Africa (ZA)
- **Language**: English (en)
- **Default Search**: "software developer jobs in South Africa"

You can modify these in `src/App.vue` if needed.

## Project Structure

```
Jsearch/
├── api/
│   └── jobs.js              # Vercel serverless function
├── src/
│   ├── components/
│   │   ├── SearchBar.vue    # Search input component
│   │   ├── Filters.vue      # Filter sidebar/drawer
│   │   ├── JobCard.vue      # Individual job card
│   │   ├── SkeletonLoader.vue # Loading state
│   │   └── EmptyState.vue   # Empty results state
│   ├── App.vue              # Main application
│   └── style.css            # Global styles
├── tailwind.config.js       # Tailwind configuration
├── vercel.json              # Vercel configuration
└── README.md
```

## Features Breakdown

### Search
- Debounced search input (300ms)
- Job title/keywords search
- Location-based search
- Preserved last search state

### Filters
- Date posted (any time, 24h, 3 days, week, month)
- Employment type (full-time, part-time, contract, internship)
- Remote work toggle
- Collapsible sections
- Mobile drawer for filters

### Job Cards
- Company logo with fallback
- Job title and company name
- Location and employment type tags
- Remote badge
- Job description preview
- Posted date (relative)
- Salary range (when available)
- Direct apply link

### UI/UX
- Glassmorphism hero section
- Smooth transitions and animations
- Hover effects and visual feedback
- Skeleton loaders during fetch
- Empty state with suggestions
- Error handling with retry
- Load more pagination
- Responsive design (mobile-first)

## License

MIT

## Support

For issues or questions, please open an issue on GitHub.

## Auto-apply (CVs → job applications and email alerts)

`npm run autoapply` checks every PDF in `CVs/` against fresh JSearch listings. It uses only free services: the JSearch free plan, Gmail and GitHub Actions.

1. Each CV needs an entry in `CVs/people.json`, keyed by the PDF file name:
   - `enabled` (optional, default `true`): set to `false` to stop searching and applying for this CV without deleting its entry
   - `name`, `email` (where their job alerts go), `phone` (optional, printed in the cover letter)
   - `roles`: the job titles to search for
   - `keywords` (optional): a job's title must contain at least one of these (e.g. "Nurse")
   - `exclude` (optional): a job whose title contains any of these is skipped (e.g. "Senior")
   - `skills` (optional): skills from their CV, most important first, for `{matchedSkills}` (below). `"Spring Boot|Springboot"` matches either spelling and is written as "Spring Boot".
   - `coverLetter` (optional): a cover letter template written inline; a file in `CoverLetters/` (below) takes priority
1. Put each person's cover letter in `CoverLetters/` with the same file name as their CV:
   - **Template** (`CVs/jane_cv.pdf` → `CoverLetters/jane_cv.txt`): plain text, filled in from each job post and made into a PDF for every application. It can use these fields:

     | Field | Filled with |
     |---|---|
     | `{title}`, `{company}` | the job title and employer ("your company" if missing) |
     | `{location}` | the job's city and province |
     | `{employmentType}` | full-time, part-time, contract, internship or temporary |
     | `{matchedSkills}` | up to 4 of the person's `skills` that the job post mentions, e.g. "Java, Spring Boot and SQL" |
     | `{date}` | today's date, e.g. "12 November 2026" |
     | `{name}`, `{phone}`, `{email}` | from `people.json` |

     A sentence using a field the job post doesn't provide is left out, so put optional fields in their own sentence (e.g. "The role's focus on {matchedSkills} is a close match for my experience.").
   - **Finished PDF** (`CoverLetters/jane_cv.pdf`): attached unchanged to every application, so keep it general. Used only when there's no `.txt` template.
   - A CV with neither (and no `coverLetter`) gets a short generic letter. A letter file that doesn't match any CV's name is reported at the start of each run.
   - Applications are emailed with the CV and `Cover Letter - <name>.pdf` attached, and a short note naming the job as the email body.
   - `npm run autoapply -- --preview` searches as usual but sends nothing, and writes sample letters for up to 3 new jobs per CV to `data/previews/` so you can check how they read.
2. Each run searches all of South Africa for jobs posted in the last 3 days, one search per source per CV, cycling through that CV's roles. Sources: JSearch (`RAPIDAPI_KEY`), plus Adzuna (`ADZUNA_APP_ID` + `ADZUNA_APP_KEY`) and Jooble (`JOOBLE_API_KEY`) when their keys are set. A source that fails is skipped for that run, and a job listed on several sources (same title and employer) is only handled once.
3. If a listing asks for applications by email, the CV is emailed to that address with the cover letter (at most 5 per CV per run).
4. The person then gets one email listing the jobs applied for and the other new jobs, with links to apply themselves (at most 10 per run; the rest wait for the next run).

Jobs already handled are remembered in `data/state.json`, so nobody gets the same job twice.
Use `npm run autoapply -- --dry-run` to see what would happen without sending anything.
Use `npm run autoapply -- --demo-to=you@example.com` (or the "Demo" box when starting the workflow on GitHub) to receive one real application per CV yourself, including disabled CVs, instead of it going to employers.
Locally, settings go in `.env` (see `.env.example`). On GitHub, the workflow in `.github/workflows/autoapply.yml` runs it at 10:00, 14:00 and 17:00 (South Africa time) using repository secrets. You can also start it from the Actions tab, with an optional dry run.

The JSearch free plan allows about 200 searches a month. Two CVs × 3 runs a day uses about 186, so adding a third CV means running less often (edit the `cron` line in the workflow).

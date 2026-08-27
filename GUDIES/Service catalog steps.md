
Backstage's current recommended approach is to use the GitHub catalog provider to crawl your GitHub organisation and find repositories containing catalog-info.yaml. (Backstage)
The architecture
Think of it like this:
                   GitHub
                      │
                      │ API
                      ▼
              GitHub Integration
                      │
                      ▼
             GitHub Discovery
                  Provider
                      │
                      ▼
            Backstage Catalog DB
                      │
                      ▼
              ┌───────────────┐
              │ /catalog      │
              │               │
              │ App A         │
              │ App B         │
              │ App C         │
              │ App D         │
              └───────────────┘

The important thing is:
GitHub repository ≠ Backstage application automatically.
Backstage needs to know what each repository represents. Normally you put a catalog-info.yaml in each repository.

1. Install the GitHub Catalog provider
From your Backstage root:
yarn --cwd packages/backend add @backstage/plugin-catalog-backend-module-github

Then in:
packages/backend/src/index.ts

add:
backend.add(import('@backstage/plugin-catalog-backend'));
backend.add(import('@backstage/plugin-catalog-backend-module-github'));

The GitHub discovery provider isn't installed by default, so this package is required. (Backstage)

2. Connect Backstage to GitHub
Note: 
Kindly create a free GitHub organisation in your GitHub account first if you dont have one.
You may add GitHub accounts to your newly created organisation members
This link worked for me…
https://github.com/account/organizations/new?plan=free&ref_cta=Create%2520a%2520free%2520organization&ref_loc=cards&ref_page=%2Forganizations%2Fplan 
In your root:
app-config.yaml

add:
integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}

Then create the environment variable:
export GITHUB_TOKEN=your_token_here

For a GitHub Personal Access Token, Backstage's documentation says reading software components requires the repo scope; organization-data access may additionally require read:org, read:user, and user:email. (Backstage)


For Local Desktop Personal Demo 
 For Demo purposes and local set up, create a Personal access token in Github with this link and permissions 
https://github.com/settings/tokens/new 
✅ Minimum permissions for repo CRUD
1. Core permission (REQUIRED)
✅ repo → Full control of private repositories
This single checkbox already includes:
Create repo
Clone / read repo
Push (update code)
Manage branches
Delete contents
👉 If you’re working with private repos, this is enough for 90% of use cases.

2. If you also want to delete repos
✅ delete_repo
👉 Needed only if you want to delete entire repositories (not just files inside them).

3. If you use GitHub Actions (optional)
✅ workflow
👉 Only tick this if your repo uses CI/CD (e.g. Backstage scaffolder creating workflows).


For a company GitHub organisation (Ignore this section if you are doing a personal demo)
consider a GitHub App instead of a personal token.
Backstage supports GitHub Apps, and they provide higher rate limits and a clearer permission model. (Backstage)
For simply reading software components, the GitHub App needs at least:
Contents: Read-only
Commit statuses: Read-only
and if you want organisation information:
Members: Read-only




3. Tell Backstage which GitHub organisation to scan
This is the important part.
In app-config.yaml:
catalog:
  providers:
    github:
      myGithubProvider:
        organization: my-github-org # create a new org and put the name of the org here
        catalogPath: '/catalog-info.yaml'
        filters:
          branch: 'main'
          repository: '.*'
        schedule:
          frequency: { minutes: 30 }
          timeout: { minutes: 3 }

Change:
organization: my-github-org
to your actual GitHub organisation.
For example:
organization: pipelines-of-code

The provider will then look through the organisation's repositories for:
/catalog-info.yaml

and ingest the entities it finds.

4. Put catalog-info.yaml in each application repository
For example, suppose GitHub contains:
my-company/
│
├── customer-api/
├── payments-api/
├── frontend/
├── identity-service/
└── reporting-service/

You could have:
customer-api/
└── catalog-info.yaml

payments-api/
└── catalog-info.yaml

frontend/
└── catalog-info.yaml

identity-service/
└── catalog-info.yaml

reporting-service/
└── catalog-info.yaml

A simple catalog-info.yaml could be:
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: customer-api
  description: Customer API
  annotations:
    github.com/project-slug: my-company/customer-api

spec:
  type: service
  lifecycle: production
  owner: team-platform

Another repository:
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: payments-api
  description: Payments API
  annotations:
    github.com/project-slug: my-company/payments-api

spec:
  type: service
  lifecycle: production
  owner: team-payments

Now Backstage knows:
GitHub Repository
       ↓
catalog-info.yaml
       ↓
Backstage Component
       ↓
Software Catalog


5. Start Backstage
Run:
yarn dev or yarn start

Then go to:
http://localhost:3000/catalog

Backstage's Software Catalog is exposed at /catalog. (Backstage)
You should eventually see:
Software Catalog

Customer API
Payments API
Frontend
Identity Service
Reporting Service

The provider periodically scans GitHub according to your schedule.

The REALLY important distinction
You previously asked me about GitHub authentication.
There are actually two separate GitHub connections you can have in Backstage:
A. GitHub Login
User
 ↓
GitHub
 ↓
Backstage Login

This is configured under:
auth:
  providers:
    github:

That's for users signing into Backstage with GitHub. (Backstage)
B. GitHub Catalog Integration
Backstage Backend
       ↓
     GitHub
       ↓
   repositories
       ↓
catalog-info.yaml
       ↓
Backstage Catalog

This is configured under:
integrations:
  github:

and:
catalog:
  providers:
    github:

That's what you need for automatically picking up your applications.

If you DON'T want catalog-info.yaml in every repo
That's a different requirement.
If what you actually mean is:
“I have 100 GitHub repositories and I simply want Backstage to list all of them as applications, without adding catalog-info.yaml to each repository.”
we can do that too, but I'd recommend deciding what you want Backstage to consider an application.
For example:
GitHub
│
├── app-one       → Component
├── app-two       → Component
├── app-three     → Component
├── library-one   → Library
└── infrastructure → Resource

Backstage's catalog is designed around entities, not simply a list of GitHub repositories. The catalog-info.yaml tells Backstage the entity's kind, owner, lifecycle, type, relationships, etc. (Backstage)
So for a proper internal developer portal, I'd use:
GitHub → Discovery Provider → catalog-info.yaml → Software Catalog.
If you tell me your GitHub organisation name and whether you're using GitHub.com or GitHub Enterprise, I can give you the exact app-config.yaml, backend setup, GitHub token/App setup, and catalog-info.yaml for your Backstage project.


Tested local walkthrough for this repo
This section records the exact working path followed in the local Backstage PoC.

Context used in the test:
GitHub organisation: dev-santhus
Backstage branch: template-playground
Example existing app repo: dev-santhus/student-record
Python skeleton repo: dev-santhus/python-boilerplate-skeleton
Generated test repo: dev-santhus/demo-python-app

1. Create or choose a GitHub organisation
We used:
dev-santhus

Why:
The GitHub catalog provider scans an organisation, not just the local Backstage repository.

2. Install the GitHub catalog provider package
From the Backstage repo root:

corepack yarn --cwd packages/backend add @backstage/plugin-catalog-backend-module-github

What this does:
Adds the GitHub catalog discovery backend module to packages/backend/package.json and updates yarn.lock.

Why:
Backstage does not load GitHub organisation discovery by default.

3. Register the GitHub catalog module in the backend
In packages/backend/src/index.ts, add the GitHub module near the catalog plugin:

// catalog plugin
backend.add(import('@backstage/plugin-catalog-backend'));
backend.add(import('@backstage/plugin-catalog-backend-module-github'));
backend.add(
  import('@backstage/plugin-catalog-backend-module-scaffolder-entity-model'),
);

What this does:
Loads the GitHub catalog provider when the Backstage backend starts.

4. Configure GitHub integration in app-config.yaml
The app already had this:

integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}

What this does:
Tells Backstage to read the GitHub token from the GITHUB_TOKEN environment variable.

5. Configure GitHub organisation discovery
In app-config.yaml, under catalog:, add:

catalog:
  providers:
    github:
      githubOrgDiscovery:
        organization: dev-santhus
        catalogPath: /catalog-info.yaml
        filters:
          branch: main
          repository: .*
        schedule:
          frequency: { minutes: 30 }
          timeout: { minutes: 3 }

What this does:
Backstage scans all repos in the dev-santhus organisation on the main branch and imports repos that contain /catalog-info.yaml.

6. Allow Template entities in catalog rules
Make sure app-config.yaml allows Template entities:

catalog:
  rules:
    - allow: [Component, System, API, Resource, Location, Template]

Why:
Without Template in the allow list, Backstage can reject scaffolder templates discovered through catalog locations.

7. Add a catalog-info.yaml to each app repo that should appear in the catalog
For dev-santhus/student-record, add this file at the root:

apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: student-record
  description: Student record application
  annotations:
    github.com/project-slug: dev-santhus/student-record
spec:
  type: service
  lifecycle: experimental
  owner: user:default/backstage-guest

What this does:
Makes the GitHub repository importable as a Backstage Component.

8. Set the GitHub token in PowerShell
For this local Windows setup, this worked:

$env:GITHUB_TOKEN="your_github_token_here"

What this does:
Sets GITHUB_TOKEN for the current PowerShell session only.

Important:
Do not paste real tokens into notes, chat, commits, or screenshots. If a token is exposed, revoke or regenerate it.

9. Fine-grained token permissions that worked
For catalog read-only discovery:

Resource owner:
dev-santhus

Repository access:
All repositories, or select every repo Backstage must read.

Repository permissions:
Metadata: Read-only
Contents: Read-only
Commit statuses: Read-only

For scaffolder templates that create new GitHub repos:

Resource owner:
dev-santhus

Repository access:
All repositories

Repository permissions:
Metadata: Read-only
Contents: Read and write
Commit statuses: Read-only
Administration: Read and write

Why:
publish:github uses GitHub's create organisation repository API. Fine-grained tokens need Administration: Read and write to create repos in an organisation, and Contents: Read and write to push generated files.

10. Start Backstage
From the Backstage repo root:

corepack yarn start

What this does:
Starts the local Backstage frontend and backend.

11. Verify the catalog
Open:

http://localhost:3000/catalog

Expected result after discovery:
student-record appears as a Component.

Note about local SQLite:
This project currently uses better-sqlite3 with connection ':memory:'. That means catalog data resets every time Backstage restarts. If GitHub discovery or the token is broken after restart, previously imported entities disappear until discovery succeeds again.

12. Common errors seen during this implementation
Error:
401 Unauthorized while reading GitHub

Meaning:
The token was missing, not loaded, expired, or did not have access to the repo being read.

Fix:
Set GITHUB_TOKEN again, restart Backstage, and confirm the fine-grained token includes all required repos.

Error:
Resource not accessible by personal access token while creating dev-santhus/demo-python-app

Meaning:
The token could read repos but could not create an organisation repo.

Fix:
Add Administration: Read and write and Contents: Read and write to the fine-grained token.

Error:
Catalog item disappeared after restart

Meaning:
The in-memory SQLite catalog reset.

Fix:
Make sure GitHub discovery can run again, or move local development to a persistent database/file-backed config later.


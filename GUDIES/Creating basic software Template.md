
Creating Basic Software Templates for Node.Js, Java, Next JS, Go and Python 
Note: The default examples/ folder in the Backstage standalone and the reference in the app-config.yaml show how a sample workflow can create a repository in your GitHub repo. 
To get started, the following steps show how Backstage expects templates to be wired. 
You can simply replicate that pattern per language and tweak the template.yaml + content/ for Java (Maven), Go, Python, Next.js 👇

Note: You must have your GITHUB_TOKEN set in your environment variables 
OR run this command in your terminal or command line 

export GITHUB_TOKEN=your_token_here
yarn start


🧱 1. Understand your current working template
You already have:
examples/template/
  ├── template.yaml
  └── content/
      ├── catalog-info.yaml
      ├── index.js
      └── package.json
👉 This is a Node.js template
So your job = clone this structure per stack

🧭 2. Create new template folders
From your examples/ directory:
mkdir java-maven-service
mkdir go-service
mkdir python-service
mkdir nextjs-service
Each should look like:
examples/<service>/
  ├── template.yaml
  └── content/

☕ 3. Java Maven Template
📁 Structure
examples/java-maven-service/
  ├── template.yaml
  └── content/
      ├── pom.xml
      ├── src/main/java/App.java
      └── catalog-info.yaml


📄 src/main/java/App.java
public class App {

    public static void main(String[] args) {
        System.out.println("Hello from {{name}}");
    }
}



🧾 template.yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: java-maven-service
  title: Java Maven Service
  description: Create a Java Maven microservice
spec:
  owner: user:guest
  type: service

  parameters:
    - title: Service Info
      required:
        - name
      properties:
        name:
          type: string
          title: Name of the service

  steps:
    - id: fetch-base
      name: Fetch Template
      action: fetch:template
      input:
        url: ./content
        values:
          name: parameters.name-id:publishname:PublishtoGitHubaction:publish:githubinput:repoUrl:github.com?repo={{ parameters.name }}&owner=olumuyiwa-org

    - id: register
      name: Register in Catalog
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: '/catalog-info.yaml'

📦 pom.xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>{{name}}</artifactId>
  <version>1.0.0</version>
</project>


 
🐹 4. Go Template
📁 Structure
go-service/
  ├── template.yaml
  └── content/
      ├── main.go
      ├── go.mod
      └── catalog-info.yaml







🧾 go.mod

module example.com/go-service

go 1.22


🧾 main.go
package main

import "fmt"

func main() {
    fmt.Println("Hello from {{name}}")
}

🐍 5. Python Template
📁 Structure
python-service/
  ├── template.yaml
  └── content/
      ├── app.py
      ├── requirements.txt (For a simple template, this file can be empty)
      └── catalog-info.yaml
🧾 app.py
def main():
    print("Hello from {{name}}")

if __name__ == "__main__":
    main()

⚛️ 6. Next.js Template
📁 Structure
nextjs-service/
  ├── template.yaml
  └── content/
      ├── package.json
      ├── pages/index.js
      └── catalog-info.yaml
🧾 pages/index.js
export default function Home() {
  return <h1>Hello from {{name}}</h1>;
}

🧩 7. catalog-info.yaml (same pattern for all)
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: {{name}}
spec:
  type: service
  owner: user:guest
  lifecycle: experimental

🔗 8. Wire templates into Backstage (you already started)
Update your app-config.yaml:
catalog:
  locations:
    - type: file
      target: ../../examples/java-maven-service/template.yaml
      rules:
        - allow: [Template]

    - type: file
      target: ../../examples/go-service/template.yaml
      rules:
        - allow: [Template]

    - type: file
      target: ../../examples/python-service/template.yaml
      rules:
        - allow: [Template]

    - type: file
      target: ../../examples/nextjs-service/template.yaml
      rules:
        - allow: [Template]

🚀 9. Run Backstage
yarn dev
Then go to:
👉 http://localhost:3000/create
You’ll see:
Java Maven Service
Go Service
Python Service
Next.js Service

⚠️ Common mistakes 
❌ url: ./content wrong path
❌ missing catalog-info.yaml
❌ GitHub token not set
❌ Github repo owner incorrect (e.g olumuyiwa-org)
❌ template not picked → restart Backstage

💡 Advanced  move (for your platform vision)
Once this works, next step is:
👉 Turn these into Golden Paths
add Dockerfile
add CI/CD
add lint/test setup
enforce naming conventions



Tested Python template walkthrough for this repo
This section records the exact working path followed for issue #17:
"Create a template in Backstage for users to create a Python application".

Acceptance criteria from the issue:
1. Create a repo called python-boilerplate-skeleton.
2. Add essential files and folders in the repo.
3. Create a template that uses python-boilerplate-skeleton to create a Python project in Backstage.

1. Create the skeleton repository
In GitHub, create:

dev-santhus/python-boilerplate-skeleton

Recommended settings:
Owner: dev-santhus
Repository name: python-boilerplate-skeleton
Visibility: Public
Add README: Yes
Add .gitignore: Python
License: No license
Description: Python boilerplate skeleton for Backstage scaffolder templates

What this repo does:
It is the source skeleton that Backstage copies when a developer creates a new Python app.

2. Add app.py to the skeleton repo
Create app.py at the root:

def main():
    print("Hello from ${{ values.name }}")


if __name__ == "__main__":
    main()

What this does:
Creates a simple Python entrypoint. Backstage replaces ${{ values.name }} with the project name entered in the template form.

3. Add requirements.txt
Create requirements.txt at the root:

pytest==8.3.4
ruff==0.8.4

What this does:
Adds a basic test runner and linter to the generated project.

4. Add catalog-info.yaml to the skeleton repo
Create catalog-info.yaml at the root:

apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: ${{ values.name }}
  description: ${{ values.description }}
  annotations:
    github.com/project-slug: ${{ values.owner }}/${{ values.repo }}
spec:
  type: service
  lifecycle: experimental
  owner: ${{ values.ownerEntity }}

What this does:
Every generated repo gets its own Backstage catalog descriptor, so it can be registered automatically after publishing.

5. Add a basic test
Create tests/test_app.py:

from app import main


def test_main_runs(capsys):
    main()

    captured = capsys.readouterr()

    assert "Hello from" in captured.out

What this does:
Adds a simple pytest test to prove the generated app runs.

6. Create the Backstage template folder locally
In the Backstage repo, create:

frontend-templates/staging/python-project/template.yaml

What this does:
Adds a scaffolder template definition for the Python project.

7. Add the working Python template.yaml
Use this content:

apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: python-project
  title: Python Project
  description: Create a Python application from the standard boilerplate skeleton
spec:
  owner: user:default/backstage-guest
  type: service

  parameters:
    - title: Project details
      required:
        - name
        - repo
      properties:
        name:
          title: Project Name
          type: string
          description: Name of the Python application
          pattern: '^[a-z0-9-]+$'
        description:
          title: Description
          type: string
          description: Short description of the Python application
        ownerEntity:
          title: Owner
          type: string
          default: user:default/backstage-guest
        repo:
          title: Repository Name
          type: string
          description: GitHub repository name to create
          pattern: '^[a-z0-9-]+$'

  steps:
    - id: fetch-base
      name: Fetch Python Boilerplate
      action: fetch:template
      input:
        url: https://github.com/dev-santhus/python-boilerplate-skeleton/tree/main
        values:
          name: ${{ parameters.name }}
          description: ${{ parameters.description }}
          owner: dev-santhus
          repo: ${{ parameters.repo }}
          ownerEntity: ${{ parameters.ownerEntity }}

    - id: publish
      name: Publish To GitHub
      action: publish:github
      input:
        repoUrl: github.com?owner=dev-santhus&repo=${{ parameters.repo }}
        description: ${{ parameters.description }}
        defaultBranch: main
        repoVisibility: public

    - id: register
      name: Register In Catalog
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: /catalog-info.yaml

  output:
    links:
      - title: Repository
        url: ${{ steps.publish.output.remoteUrl }}
      - title: Open In Catalog
        icon: catalog
        entityRef: ${{ steps.register.output.entityRef }}

Important version-specific note:
For this repo's Backstage version, this worked:

url: https://github.com/dev-santhus/python-boilerplate-skeleton/tree/main

These did not work during testing:

url: https://github.com/dev-santhus/python-boilerplate-skeleton/archive/refs/heads/main.zip

and:

url: https://github.com/dev-santhus/python-boilerplate-skeleton
ref: main

The zip URL produced a GitHub API 404. The separate ref field produced a fetch:template validation error because this action version did not allow an additional ref property.

8. Register the Python template in template-locations.yaml
Update:

frontend-templates/staging/template-locations.yaml

Add:

apiVersion: backstage.io/v1alpha1
kind: Location
metadata:
  name: templates
  description: All templates
spec:
  targets:
    - ./backstage-user/template.yaml
    - ./maven-project/template.yaml
    - ./reactvite-project/template.yaml
    - ./python-project/template.yaml

What this does:
Adds Python Project to the group of templates Backstage loads.

9. Wire template-locations.yaml into local app-config.yaml
In app-config.yaml, under catalog.locations, add:

    - type: file
      target: ../../frontend-templates/staging/template-locations.yaml
      rules:
        - allow: [Location]

What this does:
Makes the local Backstage backend load the template location file from frontend-templates/staging.

10. Make sure Template is allowed in catalog rules
In app-config.yaml:

catalog:
  rules:
    - allow: [Component, System, API, Resource, Location, Template]

What this does:
Allows the catalog to ingest Template entities. Without this, the Python template may not appear on the Create page.

11. Set the GitHub token for local testing
In PowerShell:

$env:GITHUB_TOKEN="your_github_token_here"

What this does:
Sets the GitHub token for the current terminal session.

Important:
If a token is pasted into chat, committed, or exposed in a screenshot, revoke or regenerate it.

12. Token permissions needed for the Python template
For reading templates and catalog files:

Metadata: Read-only
Contents: Read-only
Commit statuses: Read-only

For creating new repos with publish:github:

Metadata: Read-only
Contents: Read and write
Commit statuses: Read-only
Administration: Read and write

Use:
Resource owner: dev-santhus
Repository access: All repositories

Why:
Backstage must read the skeleton repo, create a new organisation repo, push files, and register catalog-info.yaml.

13. Restart Backstage
From the repo root:

corepack yarn start

What this does:
Starts Backstage and loads the latest template/catalog configuration.

14. Verify the template appears
Open:

http://localhost:3000/create

Expected result:
Python Project appears on the Create page.

15. Test project creation
Use test values:

Project Name: demo-python-app
Description: Demo Python app created from Backstage
Owner: user:default/backstage-guest
Repository Name: demo-python-app

Expected result:
Backstage fetches the skeleton, creates dev-santhus/demo-python-app, pushes the files, and registers the generated catalog-info.yaml.

16. Verify the generated repo
Open the dev-santhus organisation in GitHub.

Expected result:
demo-python-app appears as a new repository.

17. Verify the generated component in Backstage
Open:

http://localhost:3000/catalog

Expected result:
demo-python-app appears as a Component.

18. Errors seen and fixes
Error:
The Python template does not appear on the Create page.

Fix:
Add the template to frontend-templates/staging/template-locations.yaml, add the Location entry in app-config.yaml, make sure Template is allowed in catalog rules, then restart Backstage.

Error:
Request failed for /archive/refs/heads/main.zip, 404 Not Found.

Fix:
Use the tree/main URL in fetch:template:
https://github.com/dev-santhus/python-boilerplate-skeleton/tree/main

Error:
Invalid input passed to action fetch:template, additional property "ref".

Fix:
Do not use ref: main with this Backstage version. Put the branch in the URL with /tree/main.

Error:
401 Unauthorized while fetching the skeleton repo.

Fix:
Make sure GITHUB_TOKEN is set in the terminal session, the token has access to python-boilerplate-skeleton, and Backstage has been restarted.

Error:
Resource not accessible by personal access token while creating the organisation repo.

Fix:
Give the fine-grained token Administration: Read and write and Contents: Read and write.

19. Local database note
This Backstage project currently uses an in-memory SQLite database:

backend:
  database:
    client: better-sqlite3
    connection: ':memory:'

That means catalog entries reset after every restart. GitHub discovery must run successfully after restart for GitHub-imported entities to reappear.


🚀 Advanced Software Templates (Golden Path)

⚠️ Prerequisite: Start with basic templates first to understand the flow 
This stage focuses on building production-grade boilerplates for:
Node.js
Java (Maven)
Go
Python
Next.js / React
🔑 Environment Setup (Backstage)
Before running templates locally:
export GITHUB_TOKEN=your_token_here
yarn start


🧱 1. Golden Rule (Applies to ALL Stacks)
Every boilerplate must follow the same mental model:
<stack>-boilerplate/
├── src/ or cmd/        # Application code
├── tests/              # Unit / integration tests
├── .github/workflows   # OR .gitlab-ci.yml
├── .gitignore
├── Dockerfile
├── Makefile ✅
├── README.md
├── catalog-info.yaml   # Backstage registration
├── <language-specific files>
└── sonar-project.properties (optional)
🎯 Goal
Zero confusion when switching stacks
Faster onboarding
Standardised Developer experience across teams

🛠️ 2. What a Makefile Actually Does 
A Makefile is your universal command interface.
Instead of remembering 10+ commands across tools, you run:
make build
make test
make run
make lint
💡 Why this matters
Without Makefile:
npm install
npm run build
npm run test
docker build -t app .
With Makefile:
make all
✅ Benefits
One command style across ALL languages
Simplifies onboarding (especially juniors)
Standard entry point for CI/CD
Removes “how do I run this project?” confusion

🔥 Example Makefile (works across most stacks)
.PHONY: install build test lint run docker-build

install:
	npm install || pip install -r requirements.txt || go mod tidy || mvn install

build:
	npm run build || go build ./... || mvn package
test:
	npm test || pytest || go test ./... || mvn test
lint:
	npm run lint || flake8 || golangci-lint run || mvn checkstyle:check
run:
	npm start || python src/main.py || go run cmd/app/main.go || mvn spring-boot:run
docker-build:
	docker build -t app .
all: install build test
👉 This is the Golden Path interface across all services.

🐹 3. Go Boilerplate (Refined)
go-boilerplate/
├── cmd/app/main.go
├── pkg/helloworld/
├── tests/
├── .github/workflows/  OR .gitlab-ci.yaml 
├── Dockerfile
├── Makefile ✅
├── go.mod
├── go.sum
├── README.md
├── catalog-info.yaml
└── sonar-project.properties

☕ 4. Java (Maven) Boilerplate
maven-boilerplate/
├── src/main/java/
├── src/test/java/
├── .mvn/wrapper/
├── .github/workflows/  OR .gitlab-ci.yaml 
├── Dockerfile
├── Makefile ✅
├── pom.xml
├── mvnw
├── mvnw.cmd
├── README.md
├── catalog-info.yaml
└── sonar-project.properties


🟩 5. Node.js Boilerplate
nodejs-boilerplate/
├── src/
├── tests/
├── .github/workflows/  OR .gitlab-ci.yaml 
├── Dockerfile
├── Makefile ✅
├── package.json
├── yarn.lock
├── tsconfig.json
├── eslint.config.js
├── vitest.config.ts
├── README.md
└── catalog-info.yaml

⚛️ 6. Next.js Boilerplate
nextjs-boilerplate/
├── app/
├── public/
├── tests/
├── .github/workflows/  OR .gitlab-ci.yaml 
├── Dockerfile
├── Makefile ✅
├── package.json
├── next.config.ts
├── tsconfig.json
├── tailwind.config.ts
├── README.md
└── catalog-info.yaml

🐍 7. Python Boilerplate
python-boilerplate/
├── src/
├── tests/
├── .github/workflows/  OR .gitlab-ci.yaml 
├── Dockerfile
├── Makefile ✅
├── pyproject.toml
├── requirements.txt
├── README.md
├── catalog-info.yaml
└── sonar-project.properties

⚛️ 8. React Boilerplate
react-boilerplate/
├── public/
├── src/
├── tests/
├── .github/workflows/  OR .gitlab-ci.yaml 
├── Dockerfile
├── Makefile ✅
├── package.json
├── vite.config.ts
├── tsconfig.json
├── eslint.config.js
├── README.md
└── catalog-info.yaml


🧠 9. Backstage Scaffolder Alignment
Each template must follow:
steps:
  - id: fetch-base
    name: Fetch Template
    action: fetch:template
    input:
      url: ./content
Folder Structure
examples/<service>/
├── template.yaml
└── content/
    └── <stack>-boilerplate/
👉 Rule: Everything inside content/ becomes the generated repo.




🚀 10. Golden Path = Enterprise Ready
To move from “template” → “platform standard”, every service should include:
✅ Required Capabilities
Makefile → standard commands
Dockerfile → containerisation
CI/CD pipeline → build + test + deploy
Health endpoint → /health
Logging setup → structured logs
Linting + testing preconfigured
README with run instructions

🧩 Final Insight (What You're Really Building)
You’re not just creating templates—you’re building:
A Developer Platform Contract
Every service:
Builds the same way
Runs the same way
Deploys the same way
That’s what makes Backstage powerful.




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




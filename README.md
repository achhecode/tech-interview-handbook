# Tech Interview Handbook

Java, Spring Boot, Microservices, System Design and coding interview questions with answers.
Built with [Jekyll](https://jekyllrb.com) and [Just the Docs](https://just-the-docs.github.io/just-the-docs/).

**Live site:** https://achhecode.github.io/tech-interview-handbook

## Structure

```
.
├── _config.yml
├── index.md                  # Home
├── how-to-add.md             # How to add questions
├── _templates/               # Copy-paste templates (not published)
├── introduction/             # 1  Introduction and Preparation
├── core-java/                # 2  Core Java
├── modern-java/              # 3  Modern Java (8 to 25)
├── spring-framework/         # 4  Spring Framework
├── spring-boot/              # 5  Spring Boot
├── database/                 # 6  SQL, JPA and Hibernate
├── microservices/            # 7  Microservices
├── design-patterns/          # 8  Design Patterns and LLD
├── system-design/            # 9  System Design
├── devops-and-cloud/         # 10 DevOps and Cloud
├── coding/                   # 11 Coding Problems
└── scenario-based/           # 12 Scenario-Based Questions
```

Each folder has an `index.md` (the section) and one `.md` file per topic.

## Add content

Open a topic page and copy the block from `_templates/question.md` above the comment at the bottom of the file.
Full instructions: see `how-to-add.md`.

## Run locally

```bash
bundle install
bundle exec jekyll serve --livereload
```

Open http://localhost:4000/tech-interview-handbook/ (restart after editing `_config.yml`).

## Deploy

Settings → Pages → Source → **GitHub Actions**, then push to `main`.

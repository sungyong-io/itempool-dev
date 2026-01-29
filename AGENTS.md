# Agent Guidelines for Item Pool System

This repository contains the documentation and source code for the Item Pool System (문제은행시스템).
Agents operating in this repository must adhere to the following guidelines, derived from the project documentation.

## 1. Build, Lint, and Test Commands

Since the project uses a split Frontend (Vue 3) and Backend (Spring Boot) architecture, commands depend on the directory.

### Frontend (Vue 3 + Vite)
*Expected directory: `frontend/` or root if separate repo.*

- **Install Dependencies:** `npm install`
- **Development Server:** `npm run dev`
- **Build:** `npm run build` (uses `vite build`)
- **Lint:** `npm run lint` (ESLint + Prettier)
- **Run Unit Tests:** `npm run test:unit` (Vitest)
  - Run single test: `npx vitest run src/path/to/test.spec.js`
- **Run E2E Tests:** `npm run test:e2e` (Playwright/Cypress)

### Backend (Spring Boot 3 + JDK 21)
*Expected directory: `backend/` or root if separate repo.*

- **Build:** `./gradlew build` (or `./mvnw package`)
- **Run:** `./gradlew bootRun` (or `./mvnw spring-boot:run`)
- **Test:** `./gradlew test` (or `./mvnw test`)
  - Run single test (Gradle): `./gradlew test --tests "kr.co.iosys.content.quiz.MyTest"`
  - Run single test (Maven): `./mvnw -Dtest=MyTest test`

## 2. Code Style & Conventions

### General Rules
- **Encoding:** UTF-8, LF line endings.
- **Indentation:**
  - Frontend: **Tab** (size 4).
  - Backend: **4 Spaces**.
- **Line Length:** 120 characters recommended.
- **Blocks:** Always use curly braces `{}` for `if`, `for`, `while`, `try`.
- **Logging:** No `console.log` or `System.out.println`. Use `SLF4J` (Java) or proper loggers.

### Naming Conventions
- **Classes/Components:** PascalCase (e.g., `QuizListView`, `TokenService`)
- **Variables/Methods:** camelCase (e.g., `accessToken`, `fetchQuizList`)
- **Constants:** UPPER_SNAKE_CASE (e.g., `TOKEN_HEADER`)
- **Database Mapping:**
  - Table columns (snake_case) map to Java fields (camelCase).
  - Example: `USER_ID` -> `userId`, `QUIZ_HISTORY_SEQ` -> `quizHistorySeq`.
- **Code Fields:**
  - Code value: `...Cd` (e.g., `userPermCd`)
  - Code name: `...Nm` (e.g., `userPermNm`)
- **Suffixes:**
  - Date: `...At`, `...Dt`, `...Datetime`
  - Boolean: `...Yn` (DB), `is...` (JS)
  - Count: `...Cnt`, Number: `...No`

### Frontend Architecture (Vue 3)
- **Framework:** Vue 3 (Composition API with `<script setup>`).
- **State:** Pinia for global state.
- **API:** Use `src/api/` modules. Do not call Axios directly in components.
- **Directory Structure:**
  - `src/pages/`: Routing views
  - `src/components/`: Shared components
  - `src/features/`: Domain features (e.g., `content/quiz`)
  - `src/stores/`: Pinia stores
- **Template:**
  - Keep tags single-line if short.
  - Attribute order: `v-if/for` > `ref/key` > `class/style` > props > events.

### Backend Architecture (Spring Boot)
- **Package Root:** `kr.co.iosys`
- **Layered Architecture:** `Controller` -> `Service` -> `Mapper (MyBatis)` -> `DB`
- **Standard Packages (per domain):**
  - `controller`, `service`, `mapper`, `dto`, `domain`
  - Example: `kr.co.iosys.content.quiz.controller`
- **API Response Wrapper:**
  ```json
  {
    "result": "success|fail",
    "message": "User message",
    "errorMessage": "Debug message",
    "data": { ... }
  }
  ```
- **Controller Rules:**
  - No business logic. Delegate to Service.
  - Use DTOs for request/response bodies.
- **Auth:**
  - Stateless Token (Access + Refresh).
  - Refresh Token Rotation required.

### Database (MyBatis)
- **Keywords:** UPPERCASE (`SELECT`, `FROM`, `WHERE`).
- **Format:** Indent for readability.
- **CamelCase:** Use `AS` or configuration to map snake_case columns to camelCase fields.
- **Performance:** Avoid N+1 issues. Use Paging for lists.

## 3. Technology Stack Summary

- **Frontend:** Vue 3, Vite, Axios, Pinia, Tailwind CSS (or SCSS).
- **Backend:** Spring Boot 3.x, Spring MVC, MyBatis, MariaDB.
- **Java:** JDK 21 (LTS).
- **DB:** MariaDB (utf8mb4).

## 4. Documentation References

Refer to `docs/` for detailed specifications:
- `문제은행시스템_코딩스타일가이드_v1.0.md`: Detailed coding standards.
- `문제은행시스템_기술스택_취합.md`: Architecture and stack details.
- `문제은행시스템_ERD_초안.md`: Database schema.

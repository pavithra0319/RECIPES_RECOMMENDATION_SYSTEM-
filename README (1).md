# 🍽️ ML-Based Recipe Recommendation System

A full-stack recipe recommendation web application that helps users discover similar recipes by name and find dishes based on ingredients available in their kitchen. The project combines machine learning with a React frontend, a Java Spring Boot backend, a MySQL database, and a Python FastAPI recommendation service.

> **Project status:** Core recipe-name recommendation flow and backend integration have been tested. Ingredient-search UI, image display, and any remaining pages should be verified before presenting them as complete.

## ✨ Features

- **Recipe-name recommendations** using TF-IDF and K-Nearest Neighbors (KNN).
- **Fuzzy title matching** using RapidFuzz to handle minor spelling differences.
- **Ingredient-based recommendations** using ingredient text similarity and additional ingredient rules.
- **User registration and login**.
- **Password hashing** with BCrypt.
- **JWT-based authentication** for protected endpoints.
- **Favourites and search history** support through the backend.
- Recipe metadata such as cuisine, course, diet, source, and image filename.
- REST API integration between the Java backend and Python ML service.

## 🧰 Technology Stack

| Layer | Technologies |
|---|---|
| Frontend | React, Vite, JavaScript, Axios, React Router |
| Backend API | Java, Spring Boot, Spring Security |
| Authentication | JWT, BCrypt |
| Database | MySQL, Spring Data JPA / Hibernate |
| ML service | Python, FastAPI, Pandas, Scikit-learn, RapidFuzz, Joblib |
| Recommendation techniques | TF-IDF vectorization, KNN, cosine distance/similarity |
| Testing tools | Postman, FastAPI Swagger UI |

## 🏗️ Architecture

```text
┌────────────────────────────┐
│       React + Vite         │
│  Login / Search / Results  │
└──────────────┬─────────────┘
               │ HTTP / Axios
               ▼
┌────────────────────────────┐
│    Java Spring Boot API    │
│ Auth / Favorites / History │
└──────────┬─────────┬───────┘
           │         │
           │         └────────────────┐
           ▼                          ▼
┌──────────────────┐       ┌────────────────────┐
│      MySQL       │       │ Python FastAPI ML   │
│ Users, favorites │       │ TF-IDF + KNN +      │
│ and search data  │       │ title matching      │
└──────────────────┘       └────────────────────┘
```

## 🧠 How Recommendations Work

### 1. Recipe-name recommendations

1. The user enters a recipe name.
2. RapidFuzz compares the input with available recipe titles and selects the best match above the configured threshold.
3. TF-IDF transforms the selected recipe's text into a numerical vector.
4. KNN retrieves nearby recipe vectors using the configured distance metric.
5. The API returns the selected recipe and the requested number of recommendations.

### 2. Ingredient-based recommendations

1. The user supplies ingredients available in their kitchen.
2. Ingredient text is normalized and transformed into features.
3. KNN retrieves candidate recipes with similar ingredient text.
4. Additional ingredient rules can filter candidates that require unavailable major ingredients.
5. Suitable candidates are ranked and returned.

**Note:** A displayed similarity score represents text/vector similarity, not the probability that a user will like a recipe. The exact behaviour depends on the model and preprocessing configuration.

## 📊 Dataset

The prepared combined dataset was reported to contain **20,689 recipes**:

| Source | Records after cleaning |
|---|---:|
| Epicurious dataset | 13,475 |
| Indian recipe dataset | 7,214 |
| **Total** | **20,689** |

The dataset includes fields such as recipe title, ingredients/text used by the model, cuisine, course, diet, source, recipe ID, and image filename. Actual available columns may vary by dataset version.

Please follow the original dataset licences and attribution requirements. Do not upload datasets unless their licences permit redistribution.

## 📁 Project Structure

The application is split into separate projects. Your local folder names and package names may differ.

```text
recipe-recommendation-system/
├── recipe-frontend/          # React + Vite frontend
│   └── src/
│       ├── pages/             # Login, Register, Home, etc.
│       └── services/          # Axios API configuration
├── recipebackend/             # Java Spring Boot backend
│   └── src/main/java/
│       └── com/example/recipebackend/
│           ├── config/
│           ├── controller/
│           ├── service/
│           ├── repository/
│           ├── entity/
│           └── security/
└── recipe-ml-api/              # Python FastAPI ML service
    ├── app.py
    ├── models/
    │   ├── tfidf_vectorizer.pkl
    │   ├── knn_model.pkl
    │   └── recipe_data.pkl
    └── requirements.txt
```

The structure above is illustrative. Keep each actual project's existing structure when adding this README.

## ⚙️ Prerequisites

Install the following before running the application:

- Python 3.10 or another version supported by the ML dependencies
- Java version compatible with the Spring Boot project
- Maven (or use the project's Maven wrapper)
- Node.js and npm
- MySQL Server and MySQL Workbench
- Git

Check your installed versions:

```bash
python --version
java -version
node --version
npm --version
git --version
```

## 🚀 Setup and Run

Run each service in a **separate terminal**. Replace paths and filenames with the actual names in your repository.

### 1. Start the Python ML API

Open a terminal in the ML API folder:

```powershell
python -m venv .venv
```

Activate the environment on Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install dependencies:

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Ensure the trained model files exist at the paths expected by the application. The model files are loaded when the API starts; they must be generated or obtained from a permitted source before launching the service.

Start FastAPI (if the entry-point file is `app.py` and the FastAPI instance is named `app`):

```powershell
python -m uvicorn app:app --reload --host 127.0.0.1 --port 8000
```

Open the interactive API documentation:

- `http://127.0.0.1:8000/docs`

Useful endpoint:

- `GET http://127.0.0.1:8000/`
- `POST http://127.0.0.1:8000/recommend`

Example request for the recipe-name endpoint:

```json
{
  "recipe_name": "Masala Dosa",
  "limit": 5
}
```

The ingredient-based endpoint and request fields depend on the implementation in your ML API. Check `/docs` for its exact route and schema.

### 2. Configure and start MySQL

1. Start MySQL Server.
2. Create a database using the database name configured by the Spring Boot application.
3. Set the database username and password in local configuration or environment variables.
4. Ensure the MySQL JDBC driver and required Spring Data JPA dependencies are present.

Example SQL (change the database name if your configuration uses another one):

```sql
CREATE DATABASE recipe_recommendation_db;
```

Do not commit real database passwords or production credentials.

### 3. Start the Spring Boot backend

Open a terminal in the backend folder and configure the database connection and ML API base URL in `application.properties` or `application.yml`.

Typical local configuration keys may include:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/recipe_recommendation_db
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

Set `DB_USERNAME` and `DB_PASSWORD` in your local environment. Add any project-specific settings required by your code. If the backend calls FastAPI at `http://127.0.0.1:8000`, ensure that the URL matches the running ML service.

Run with Maven:

```powershell
.\mvnw.cmd spring-boot:run
```

If the project has no Maven wrapper, use:

```powershell
mvn spring-boot:run
```

The backend has been configured to use port `8080` in the development setup. If that port is already occupied, stop the other application or configure a different port.

### 4. Start the React frontend

Open a terminal in the frontend folder:

```powershell
npm install
npm run dev
```

Vite normally serves the app at:

- `http://localhost:5173`

Verify that the Axios base URL points to the Spring Boot backend and that Spring Security CORS settings allow the frontend's development origin. Do not use permissive CORS settings for a production deployment.

## 🔌 API Flow

The intended request flow is:

```text
React
  → Spring Boot REST endpoint
      → Python FastAPI recommendation endpoint
      ← ML recommendation JSON
  ← Spring Boot response
← Rendered results in React
```

Example response fields from the recipe-name endpoint:

- `searched_recipe`
- `search_match_score`
- `selected_recipe`
- `recommendations`
- Recipe metadata such as `recipe_id`, `title`, `similarity`, `cuisine`, `course`, and `diet`

The exact Spring Boot route and response format should be checked against the current code.

## 🧪 Testing

Suggested manual checks:

- [ ] ML API starts and `/docs` loads.
- [ ] `POST /recommend` returns a selected recipe and recommendations for a known title.
- [ ] Unknown recipe names return a clear error.
- [ ] Spring Boot starts and connects to MySQL.
- [ ] Registration creates a user and stores a hashed password.
- [ ] Login returns the expected authentication response.
- [ ] Protected endpoints reject unauthenticated requests.
- [ ] React registration and login work.
- [ ] Recipe-name search displays the API response correctly.
- [ ] Favourites and search history work end-to-end.
- [ ] Ingredient search and recipe images are verified in the frontend.

## 🔐 Security and Repository Hygiene

Before publishing to GitHub:

- Never commit database passwords, JWT signing secrets, API keys, or personal data.
- Add `.env`, `.venv/`, `venv/`, `node_modules/`, `target/`, `__pycache__/`, and other generated files to `.gitignore`.
- Review whether the dataset and serialized model files (`.pkl`) may be redistributed. Large model files may be stored separately and documented with download/setup instructions.
- Use a specific allowed-origin list for production CORS configuration.
- Keep passwords hashed and validate authentication on protected endpoints.
- Treat downloaded serialized model files as trusted-only inputs; loading an untrusted pickle can execute malicious code.

## 🛣️ Future Enhancements

- Finish and verify the ingredient-search page in React.
- Display recipe photos using valid image URLs or a reliable local image mapping.
- Add cuisine, course, diet, preparation-time, and calorie filters.
- Improve ingredient synonym handling and recipe ranking.
- Add automated unit and integration tests.
- Deploy the frontend, backend, ML API, and database with environment-based configuration.

## 👩‍💻 Contributors

Add the names of the project team members and faculty guide here.

## 📄 License

Choose a license for your own code before publishing. Ensure dataset licences and third-party assets are respected. Do not imply that third-party datasets are owned by this project.

---

**Built as an academic full-stack machine learning project** combining recipe recommendations, REST APIs, authentication, and database integration.

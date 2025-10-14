# 1. Введение
## 1.1. Краткое описание возможностей приложения
**Voider.dev** - приложения для обучения начинающих программистов при помощи коротких уроков в игровом формате

# 2. Стек технологий
## 2.1. Серверная часть приложения
- Node.js
- Express.js
- PostgreSQL
## 2.2. Клиентская часть приложения
- Flutter(dart)
- Dio - для запросов к API

# 3. Структура приложения
## 3.1. Архитектура приложения
### 3.1.1 Иерархия обучения

Пользователь выбирает курс, разделенный на модули. В модуле есть уроки, в каждом из которых будут задания. 
```mermaid
graph LR;
A[course] --> B[modules]
B --> C[lessons]
C --> D[exercises];
```
Упражнения не привязаны к урокам, и могут повторятся от одного урока к другому. Но упражнения привязаны к модулю. Упражнение может повторно появиться в модуле на "уровень" больше, но не наоборот.  Уроки, по сути, являются "чекпонитами", постепенно проходя которые пользователь проходит модуль. Модули идут последовательно, один за другим, в заранее прописанном порядке. Пройдя все модули, пользователь закрывает курс.
## 3.2. Архитектура БД
### 3.2.1. Список таблиц

**Информация о пользователях**
```sql
CREATE TABLE users (
	id SERIAL PRIMARY KEY,
	username VARCHAR(50) UNIQUE NOT NULL,
	email VARCHAR(255) UNIQUE NOT NULL,
	password_hash VARCHAR(255) NOT NULL,
	exp_points INTEGER DEFAULT 0,
	level INTEGER DEFAULT 0,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	last_active_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	days_streak INTEGER DEFAULT 0,
	avatar_url VARCHAR(500),
);
```

**Список курсов**
```sql
CREATE TABLE courses (
	id SERIAL PRIMARY KEY,
	title VARCHAR(100) NOT NULL,
	description TEXT
);
```

**Список модулей**
```sql
CREATE TABLE modules (
	id SERIAL PRIMARY KEY,
	course_id INTEGER REFERENCES courses(id) ON DELETE CASCADE,
	title VARCHAR(100),
	order_index INTEGER NOT NULL, -- тут хранится порятдок модулей
	lesson_amount INTEGER NOT NULL -- кол-во уроков в модуле
);
```

**Список уроков**
```sql
CREATE TABLE lessons (
	id SERIAL PRIMARY KEY,
	module_id INTEGER REFERENCES modules(id) ON DELETE CASCADE,
	order_index INTEGER NOT NULL,
	repetition_required INTEGER NOT NULL DEFAULT 3,
	repetition_done INTEGER NOT NULL DEFAULT 0,
	is_complete BOOLEAN NOT NULL DEFAULT FALSE,
	exercises_per_repetition INTEGER NOT NULL DEFAULT 5
	);
```

**Список всех заданий**
```sql
CREATE TABLE exercises (
	id SERIAL PRIMARY KEY,
	module_id INTEGER REFERENCES modules(id) ON DELETE CASCADE,
	exercise_type VARCHAR(20) ,
	description VARCHAR(500),
	correct_answer INTEGER REFERENCES answers(id) ON DELETE CASCADE,
	exp_points INTEGER DEFAULT 10
);
```

**Список возможных ответов**
```sql
CREATE TABLE answers (
	id SERIAL PRIMARY KEY,
	answer VARCHAR(50) NOT NULL
);
```

**Список заданий в модуле**
```sql
CREATE TABLE module_exercises (
	id SERIAL PRIMARY KEY,
	module_id INTEGER REFERENCES modules(id) ON DELETE CASCADE,
	exercise_id INTEGER REFERENCES exercises(id) ON DELETE CASCADE,
	UNIQUE(module_id, exercise_id)
);
```

**Статистика по урокам**
```sql
CREATE TABLE lesson_attempts (
	id SERIAL PRIMARY KEY,
	user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
	lesson_id INTEGER REFERENCES lessons(id) ON DELETE CASCADE,
	attempt_amount INTEGER NOT NULL,
	is_completed BOOLEAN DEFAULT FALSE,
	is_available BOOLEAN DEFAULT FALSE,
	UNIQUE (user_id, lesson_id, attempt_amount)
);
```

**Курсы которые проходят пользователи**
```sql
CREATE TABLE user_courses (
	id SERIAL PRIMARY KEY,
	user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
	course_id INTEGER REFERENCES courses(id) ON DELETE CASCADE,
	started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	completed_at TIMESTAMP,
	current_module_id INTEGER REFERENCES modules(id),
	progress_percentage INTEGER DEFAULT 0,
	UNIQUE(user_id, course_id)
);
```

**Список достижений**
```sql
CREATE TABLE achivements (
	id SERIAL PRIMARY KEY,
	title VARCHAR(100) NOT NULL,
	description TEXT,
	icon_url VARCHAR(500)
);
```

**Достижения, полученные пользователями**
```sql
CREATE TABLE user_achivements (
	id SERIAL PRIMARY KEY,
	user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
	achivement_id INTEGER REFERENCES achivements(id) ON DELETE CASCADE,
	earned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	UNIQUE(user_id, achivement_id)
);
```

**Список друзей**
```sql
CREATE TABLE friendships (
	id SERIAL PRIMARY KEY,
	user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
	friend_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	is_request BOOLEAN DEFAULT TRUE, -- сначала запрос в друзья, при подтверждении становится FALSE
	UNIQUE(user_id, friend_id),
	CHECK(user_id != friend_id)
);
```

### 3.2.2 Диаграмма связей таблиц
```mermaid
erDiagram
    users {
        SERIAL id PK
        VARCHAR(50) username UK
        VARCHAR(255) email UK
        VARCHAR(255) password_hash
        INTEGER exp_points
        INTEGER level
        TIMESTAMP created_at
        TIMESTAMP last_active_at
        INTEGER days_streak
        VARCHAR(500) avatar_url
    }

    courses {
        SERIAL id PK
        VARCHAR(100) title
        TEXT description
        VARCHAR(50) language
    }

    user_courses {
        SERIAL id PK
        INTEGER user_id FK
        INTEGER course_id FK
        TIMESTAMP started_at
        TIMESTAMP completed_at
        INTEGER current_module_id FK
        INTEGER progress_percentage
    }

    modules {
        SERIAL id PK
        INTEGER course_id FK
        VARCHAR(100) title
        INTEGER order_index
        INTEGER lesson_amount
    }

    lessons {
        SERIAL id PK
        INTEGER module_id FK
        INTEGER order_index
        INTEGER repetition_required
        INTEGER exercises_per_repetition
    }

    answers {
        SERIAL id PK
        VARCHAR(50) answer
    }

    exercises {
        SERIAL id PK
        INTEGER module_id FK
        VARCHAR(20) exercise_type
        VARCHAR(500) description
        INTEGER correct_answer FK
        INTEGER exp_points
    }

    module_exercises {
        SERIAL id PK
        INTEGER module_id FK
        INTEGER exercise_id FK
    }

    lesson_attempts {
        SERIAL id PK
        INTEGER user_id FK
        INTEGER lesson_id FK
        INTEGER attempt_amount
        BOOLEAN is_completed
        BOOLEAN is_available
    }

    achivements {
        SERIAL id PK
        VARCHAR(100) title
        TEXT description
        VARCHAR(500) icon_url
    }

    user_achivements {
        SERIAL id PK
        INTEGER user_id FK
        INTEGER achivement_id FK
        TIMESTAMP earned_at
    }

    friendships {
        SERIAL id PK
        INTEGER user_id FK
        INTEGER friend_id FK
        TIMESTAMP created_at
        BOOLEAN is_request
    }

    users ||--o{ user_courses : "enrolls_in"
    users ||--o{ lesson_attempts : "makes"
    users ||--o{ user_achivements : "earns"
    users ||--o{ friendships : "has"
    courses ||--o{ user_courses : "has_enrollments"
    courses ||--o{ modules : "contains"
    user_courses }o--|| modules : "current_module"
    modules ||--o{ lessons : "contains"
    modules ||--o{ exercises : "has"
    modules ||--o{ module_exercises : "includes"
    lessons ||--o{ lesson_attempts : "has_attempts"
    exercises }|--|| answers : "references"
    exercises ||--o{ module_exercises : "included_in"
    achivements ||--o{ user_achivements : "awarded_to"
```
## 3.2. Архитектура API
### 3.2.1. Регистрация/вход

**Регистрация нового пользователя**
`POST api/auth/register`
```json
{
	"username": string,
	"email": string,
	"password": string
}
```

`201 Created`
```json
{
	"message": "user registered succsessfully",
	"user": {
		"id": number,
		"username": string,
		"email": string,
		"created_at": timestamp,
		"exp_points": 0,
		"level": 0,
		"days_streak": 0
	}
}
```

**Авторизация пользователя**
`POST api/auth/login`
```json
{
	"email": string,
	"password": string
}
```

`200 OK`
```json
{
	"message": "login successfull",
	"user": {
		"id": number,
		"username": string,
		"email": string,
		"exp_points": number,
		"level": number,
		"avatar_url": string,
		"days_streak": number,
		"last_active_at": timestamp
	}
}
```
### 3.2.2. Главная страница
**Дорожная карта**
`GET api/home/roadmap?user_id=number`
```json
{
	"user_info": {
		"id": number,
		"username": string,
		"level": number,
		"exp_points": number,
		"days_streak": number,
	},
	"current_course": {
		"id": number,
		"title": string,
		"progress_percentage": number,
		"current_module":{
			"id": number,
			"title": string,
			"order_index": number
			"lessons": {
				"id": number,
				"order_index": number,
				"repetition_required": number,
				"repetition_done": number,
				"is_complele": boolean
			}
		}
	}
}
```

**Прохождение урока**
`GET api/home/lessos?lesson_id=number`
```json
{
	"id": number,
	"exercises_amount": number,
	"current_repetition": number,
	"exercises": [
		"exersice_0": {
			"id": number,
			"exercise_type": string,
			"exp_points": number,
			"description": string,
			"answers": [
				"answer_0": number,
				"answer_1": number,
				"answer_2": number,
				"answer_x": number,
				],
			"correct_answer": number,
		},
		"exersice_1": {
			"id": number,
			"exercise_type": string,
			"exp_points": number,
			"description": string,
			"answers": [
				"answer_0": number,
				"answer_1": number,
				"answer_2": number,
				"answer_x": number,
				],
			"correct_answer": number,
		},
		"exersice_x": {
			"id": number,
			"exercise_type": string,
			"exp_points": number,
			"description": string,
			"answers": [
				"answer_0": number,
				"answer_1": number,
				"answer_2": number,
				"answer_x": number,
			],
			"correct_answer": number,
		},
	],
}
```

`POST api/home/lesson`
```json
{
	"id": number,
	"is_repetition_complete": boolean,
	"current_repetition": number,
	"exp_earned": number
}
```

`200 Ok`
```json
{
	"id": number,
	"is_complete": number,
	"exp_earned": number,
}
```

### 3.2.3 Профиль пользователя
**Профиль пользователя
`GET api/profile?user_id=number`
```json
{
	"user_info": {
		"id": number,
		"username": string,
		"level": number,
		"exp_points": number,
		"days_streak": number,
		"avatar_url": string,
	},
}
```

`POST api/profile/update`
```json
{
	"user_info": {
		"id": number,
		"username": string,
		"level": number,
		"exp_points": number,
		"days_streak": number,
		"avatar_url": string,
	},
}
```

`200 ok`
```json
{
	"user_info": {
		"id": number,
		"username": string,
		"level": number,
		"exp_points": number,
		"days_streak": number,
		"avatar_url": string,
	},
}
```

**Достижения пользователя**
`GET api/profile/achivements?user_id=number`
```json
{
	"id": number,
	"achemevents": [
		number: [ // number - id достижения
			"title": string,
			"dexription": string,
			"icon_url": string,
		]
	]
}
```

`POST api/profile/achevements`
```json
{
	"id": number,
	"user_id": number,
	"is_earned": boolean,
}
```

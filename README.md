# Multi-Container Application
This project is part of [roadmap.sh](https://roadmap.sh/projects/multi-container-service) DevOps projects.


## Test the Project Locally

### Endpoints

- `GET /todos` — Get all todos
- `POST /todos` — Create a new todo
- `GET /todos/:id` — Get a todo by ID
- `PUT /todos/:id` — Update a todo by ID
- `DELETE /todos/:id` — Delete a todo by ID

### Setup
1. Install dependencies:

```bash
npm install
```

2. Set up a local mongodb database
I've used docker image.

```bash
docker run -d -p 27017:27017 mongo
```

3. Start the server:
```bash
npm start
```
Go to [localhost:3000/todos](http://localhost:3000/todos)

4. Add some data to database

```bash
curl -X POST http://localhost:3000/todos \
     -H "Content-Type: application/json" \
     -d '{"title": "Buy milk"}'
```


Hello students! Your task is to containerize a multi-container application using Docker Compose.

You've been given a project repository containing a Node.js backend application and a PostgreSQL database initialization script. Your goal is to write a `docker-compose.yaml` file from scratch that builds and runs this application stack.

---

## 📋 Assignment: Building a Multi-Container App with Docker Compose

### Objective

To write a `docker-compose.yaml` file that defines, builds, and links a backend application service and a database service, enabling them to communicate and persist data.

### Provided Materials

You will need to clone this repository. Inside, you'll find:
* `backend-app/`: Contains the Node.js application source code and its `Dockerfile`.
* `database/`: Contains an `init.sql` script to create and populate the database table.

### Requirements

Your final `docker-compose.yaml` file must satisfy the following conditions:

1.  **Two Services:** It must define two services, which you can call `backend` and `db`.
2.  **Backend Service:** The `backend` service must be built from the `Dockerfile` located in the `backend-app` directory.
3.  **Database Service:** The `db` service must use the official `postgres:15-alpine` image from Docker Hub.
4.  **Port Mapping:** The `backend` service must be accessible from the host machine on port `3001`.
5.  **Service Communication:** The `backend` service must be able to connect to the `db` service using the hostname `db`.
6.  **Database Initialization:** The `db` service must automatically run the `init.sql` script on its first launch to create the `users` table ans insert some records (feel free to check its contents).
7.  **Data Persistence:** All data stored in the PostgreSQL database must be saved in a **named volume** so it persists even if the database container is removed and recreated.
8.  **Startup Order:** The `backend` service must be configured to start *only after* the `db` service has started.
9.  **Environment Variables:** You must set all necessary environment variables for both the `backend` and `db` services to connect and authenticate properly.

---

### 📝 Step-by-Step Instructions

Follow these steps to build your `docker-compose.yaml` file. I recommend you create a new file named `my-compose.yaml` in the root of the cloned directory to avoid confusion.

#### Step 1: Analyze the Application

Before writing any YAML, you *must* understand how the application works.

1.  Look inside `backend-app/index.ts`.
2.  Find the `Database connection` configuration. Notice the environment variables it uses to connect to the database:
    * `DB_USER`
    * `DB_PASSWORD`
    * `DB_HOST`
    * `DB_PORT`
    * `DB_DATABASE`

This tells you *exactly* what environment variables the `backend` service will need.

#### Step 2: Define the Compose File Structure

Start your `my-compose.yaml` file by defining the version and the `services` block.

```yaml
services:
  # Our services will go here
  ```

#### Step 3: Define the Database (`db`) Service

Let's start with the database. Add a service named `db`:

1.  **Image:** Use the `image:` key to specify `postgres:15-alpine`.
    
2.  **Environment:** Add an `environment:` block. The `postgres` image needs its _own_ set of variables to initialize the database. These variables must match what the backend expects. Let's use:
    
    -   `POSTGRES_USER=myuser`
    -   `POSTGRES_PASSWORD=mypassword`
    -   `POSTGRES_DB=mydatabase`
        
3.  **Volumes:** We need two volumes for the `db` service. Add a `volumes:` block.
    
    -   **Data Persistence:** To save the data, map a named volume (e.g., `db-data`) to the path where Postgres stores data (`/var/lib/postgresql/data`.
        
    -   **Initialization Script:** To run the `init.sql` script, you must map it into a special directory inside the container: `/docker-entrypoint-initdb.d/`. The line will look like `./database/init.sql:/docker-entrypoint-initdb.d/init.sql`

#### Step 4: Define the Backend (`backend`) Service

Now, let's define the `backend` service.

1.  **Build:** Instead of an `image`, use the `build:` key to point to the directory containing the Dockerfile: `./backend-app`.
    
2.  **Ports:** Use the `ports:` key to map your host port `3001` to the container port `3001` (which is what the Node.js app uses). This will look like `["3001:3001"]`.
    
3.  **Environment:** Add an `environment:` block. This is where you pass in the connection details for the database, based on your analysis from Step 1.
    
    -   `DB_USER=myuser` (Must match `POSTGRES_USER`)
    -   `DB_PASSWORD=mypassword` (Must match `POSTGRES_PASSWORD`)
    -   `DB_DATABASE=mydatabase` (Must match `POSTGRES_DB`)
    -   `DB_HOST=db` (This is the _critical_ part. The hostname `db` matches the _service name_ of your database. Docker Compose handles the networking.)
    -   `DB_PORT=5432` (The default port for Postgres)

#### Step 5: Define the Top-Level Volume

You referenced a named volume (`db-data`) in Step 3, but you haven't formally defined it.

### Verification

Once your `docker-compose.yaml` file is written, run it!

1.  Open your terminal in the project's root directory.
2.  Build and run the application
3.  You should see logs from both the `db` and `backend` services. Look for the `db` service log to show it's "ready to accept connections" and the `backend` service log to show "App listening on port 3001".
4.  Open your web browser and go to `http://localhost:3001/health` to check the health, go to `http://localhost:3001/api/users` to retrieve the list of users.

**Test the persistence:**

1.  In your terminal, press `Ctrl+C` to stop the containers.
2.  Run `docker compose down`. This stops and removes the containers.
3.  Run `docker-compose up` again.
4.  Refresh your browser at `http://localhost:3001/api/users`. **Your users should still be there!** This confirms your named volume is working.
5. Once everything is working fine then you've completed the assignment, congratulations **🎉** **🎉**

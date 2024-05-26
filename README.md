# Weducate Deploy Repository

This repository serves as a proxy to integrate the frontend and backend repositories of a full stack application using Git submodules. It also includes a `docker-compose.yml` file to manage and deploy the application stack.

## Repository Structure

/your-proxy-repo
├── frontend
│   ├── Dockerfile
│   ├── (other frontend files)
│   └── ...
├── backend
│   ├── Dockerfile
│   ├── (other backend files)
│   └── ...
├── docker-compose.yml
└── README.md

- `frontend/`: Contains the frontend application as a Git submodule.
- `backend/`: Contains the backend application as a Git submodule.
- `docker-compose.yml`: Configuration file for Docker Compose to manage and deploy the services.
- `README.md`: This file.

## Getting Started

### Prerequisites

- Git
- Docker
- Docker Compose

### Cloning the Repository

Clone this proxy repository along with its submodules:

```sh
git clone --recurse-submodules <url-of-proxy-repo>
cd <name-of-proxy-repo>

If you have already cloned the repository without submodules, you can initialize and update them with:

git submodule init
git submodule update

### Building and Running the Application

To build and run the entire application stack, use Docker Compose:

docker-compose up --build -d

This command will:

- Build the frontend and backend Docker images.
- Start the containers for the frontend, backend, and PostgreSQL database.
- Run the services in detached mode.

### Accessing the Application

- Frontend: [http://localhost](http://localhost)
- Backend: [http://localhost:8080](http://localhost:8080)
- PostgreSQL: Accessible within the Docker network at db:5432

### Updating Submodules

To update the frontend or backend repositories, use the following commands:

git submodule update --remote

### Committing Changes

If you make changes in the submodules and want to commit them, you need to commit changes in both the submodules and the proxy repository:

# Update proxy repository
cd ..
git add .
git commit -m "Update submodules"
git push origin main

### Stopping the Application

To stop and remove the running containers:

docker-compose down

## Troubleshooting

- Ensure Docker and Docker Compose are installed and running on your machine.
- Verify that the submodules are properly initialized and updated.
- Check the Docker logs for any errors:

    docker-compose logs
  

## Contributing

Please feel free to submit issues, fork the repository, and send pull requests.

## License

This project is licensed under the MIT License.
```

This README.md provides an overview of the repository, instructions on how to set up and run the application, and details on how to update and commit changes to the submodules. Adjust the content to match your specific project details and repository URLs.
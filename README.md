# docker-react

A React application with Docker integration, bootstrapped with Create React App.

## Description

This is a React application designed to be containerized with Docker. It includes a multi-stage Dockerfile for building optimized production images.

## Installation

Install the project dependencies using npm:

```bash
npm install
```

## Usage

In the project directory, you can run:

### `npm start`

Runs the development server on port 3000.
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will reload if you make edits.
You will also see any lint errors in the console.

### `npm test`

Launches the test runner in interactive watch mode.
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.
Your app is ready to be deployed!

## Docker

This project includes a multi-stage Dockerfile for building and deploying the application as a Docker container.

### Build the Docker Image

```bash
docker build -t docker-react .
```

### Run the Docker Container

```bash
docker run -p 8080:80 docker-react
```

The container exposes port 80, which is mapped to port 8080 on the host.
Open [http://localhost:8080](http://localhost:8080) to view the application.

### Using Docker Compose

You can also use Docker Compose to run the application:

```bash
docker-compose up
```

## Contributing

Contributions are welcome! Follow these steps to contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes with descriptive messages
4. Push to the branch (`git push origin feature/your-feature`)
5. Create a Pull Request

Please make sure to update tests as appropriate.

## License

This project is open source and available under the [MIT License](LICENSE).
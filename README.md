Movie Picture Pipeline
You've been brought on as the DevOps resource for a development team that manages a web application that is a catalog of Movie Picture movies. They're in dire need of automating their development workflows in hopes of accelerating their release cycle. They'd like to use Github Actions to automate testing, building and deploying their applications to an existing Kubernetes cluster.

The team's project is comprised of 2 application.

A frontend UI built written in Typescript, using the React framework
A backend API written in Python using the Flask framework.
In the starter folder, you'll find 2 folders, one named frontend and one named backend, where each application's source code is maintained. Your job is to use the team's existing documentation and create CI/CD pipelines to meet the teams' needs.

Deliverables
Frontend
A Continuous Integration workflow that:
Runs on pull_requests against the main branch,only when code in the frontend application changes.
Is able to be run on-demand (i.e. manually without needing to push code)
Runs the following jobs in parallel:
Runs a linting job that fails if the code doesn't adhere to eslint rules
Runs a test job that fails if the test suite doesn't pass
Runs a build job only if the lint and test jobs pass and successfully builds the application
A Continuous Deployment workflow that:
Runs on push against the main branch, only when code in the frontend application changes.
Is able to be run on-demand (i.e. manually without needing to push code)
Runs the same lint/test jobs as the Continuous Integration workflow
Runs a build job only when the lint and test jobs pass
The built docker image should be tagged with the git sha
Runs a deploy job that applies the Kubernetes manifests to the provided cluster.
The manifest should deploy the newly created tagged image
The tag applied to the image should be the git SHA of the commit that triggered the build
Backend
A Continuous Integration workflow that:
Runs on pull_requests against the main branch,only when code in the frontend application changes.
Is able to be run on-demand (i.e. manually without needing to push code)
Runs the following jobs in parallel:
Runs a linting job that fails if the code doesn't adhere to eslint rules
Runs a test job that fails if the test suite doesn't pass
Runs a build job only if the lint and test jobs pass and successfully builds the application
A Continuous Deployment workflow that:
Runs on push against the main branch, only when code in the frontend application changes.
Is able to be run on-demand (i.e. manually without needing to push code)
Runs the same lint/test jobs as the Continuous Integration workflow
Runs a build job only when the lint and test jobs pass
The built docker image should be tagged with the git sha
Runs a deploy job that applies the Kubernetes manifests to the provided cluster.
The manifest should deploy the newly created tagged image
The tag applied to the image should be the git SHA of the commit that triggered the build
⚠️ NOTE Once you begin work on Continuous Deployment, you'll need to first setup the AWS and Kubernetes environment. Follow the instructions below instructions only when you're ready to start testing your deployments.

Setting up Continuous Deployment environment
Only complete these steps once you've finished your Continuous Integration pipelines for the frontend and backend applications. This section is meant to create a Kubernetes environment for you to deploy the applications to and verify the deployment step.

First we need to prep the AWS account with the necessary infrastructure for deploying the frontend and backend applications. As the focus of this course is building the CI/CD pipelines, we won't be requiring you to setup all of the underlying AWS and Kubernetes infrastructure. This will be done for you with the provided Terraform and helper scripts. As there are costs associated with running this infrastucture, REMEMBER to destroy everything before stopping work. Everything can be recreated, and the pipeline work you'll be doing is all saved in this repository.

Create AWS infrastructure with Terraform
Export your AWS credentials from the Cloud Gateway
Use the commands below to run the Terraform and type yes after reviewing the expected changes
cd setup/terraform
terraform apply
Take note of the Terraform outputs. You'll need these later as you work on the project. You can always retrieve these values later with this command
cd setup/terraform
terraform output
Generate AWS access keys for Github Actions
Once everything is created, you'll need to generate AWS credentials for the IAM user account that Github Actions will use in order to interact with your AWS account.
Launch the Cloud Gateway and go to the IAM service.
Under users, you should only see the github-action-user user account
Click the account and go to Security Credentials
Under Access keys select Create access key
Select Application running outside AWS and click Next, then Create access key to finish creating the keys
On the last page, make sure to copy/paste these keys for storing in Github Secrets image
Add Github Action user to Kubernetes
Now that the cluster and all AWS resources have been created, you'll need to add the github-action-user IAM user ARN to the Kubernetes configuration that will allow that user to execute kubectl commands against the cluster.

Run the init.sh helper script in the setup folder
cd setup
./init.sh
The script will download a tool, add the IAM user ARN to the authentication configuration, indicate a Done status, then it'll remove the tool
Dependencies
We've provided the below list of dependencies to assist in the case you'd like to run any of the work locally. Local development issues, however, are not supported as we cannot control the environment as we can in the online workspace.

All of the tools below will be available in the workspace

docker - Used to build the frontend and backend applications
kubectl - Used to apply the kubernetes manifests
pipenv - Used for mananging Python version and dependencies
nvm - Used for managing NodeJS versions
tfswitch Used for managing Terraform versions
kustomize Used for building the Kubernetes manifests dynamically in the CI environment
jq for parsing JSON more easily on the command line
Frontend Development notes
Running tests
While in the frontend directory, perform the following steps:

# Use correct NodeJS version
nvm use

# Install dependencies
npm ci

# Run the tests interactively. You'll need to press `a` to run the tests
npm test

# OR simulate running the tests in a CI environment
CI=true npm test


# Expected output
PASS src/components/__tests__/MovieList.test.js
PASS src/components/__tests__/App.test.js

Test Suites: 2 passed, 2 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        1.33 s
Ran all test suites.
To simulate a failure in the test coverage, which will be needed to ensure your CI/CD pipeline fails on bad tests, set the MOVIE_HEADING variable before the command like so:

FAIL_TEST=true CI=true npm test
As the test is expecting the heading to contain a certain value, we can simulate a failure by changing it with an inline or environment variable. If you use the environment variable, make sure to unset it when you're done testing

# Expect tests to fail with this set to anything except Movie List
export FAIL_TEST=true
CI=true npm test

# Expect tests to be passing again
unset MOVIE_HEADING
CI=true npm test
# Expected failure output
FAIL src/components/__tests__/App.test.js
  ● renders Movie List heading

    TestingLibraryElementError: Unable to find an element with the text: messed_up. This could be because the text is broken up by multiple elements. In this case, you can provide a function for your text matcher to make your matcher more flexible.

    Ignored nodes: comments, script, style
    <body>
      <div>
        <div>
          <h1>
            Movie List
          </h1>
          <ul />
        </div>
      </div>
    </body>

       8 | test('renders Movie List heading', () => {
       9 |   render(<App />);
    > 10 |   const linkElement = screen.getByText(movieHeading);
         |                              ^
      11 |   expect(linkElement).toBeInTheDocument();
      12 | });
      13 |

      at Object.getElementError (node_modules/@testing-library/react/node_modules/@testing-library/dom/dist/config.js:37:19)
      at allQuery (node_modules/@testing-library/react/node_modules/@testing-library/dom/dist/query-helpers.js:76:38)
      at query (node_modules/@testing-library/react/node_modules/@testing-library/dom/dist/query-helpers.js:52:17)
      at getByText (node_modules/@testing-library/react/node_modules/@testing-library/dom/dist/query-helpers.js:95:19)
      at Object.<anonymous> (src/components/__tests__/App.test.js:10:30)

PASS src/components/__tests__/MovieList.test.js
Running linter
When there are no linting errors, the output won't return any errors

npm run lint

# Expected output
> frontend@1.0.0 lint
> eslint .
To simulate linting errors, you can run the linting command like so:

FAIL_LINT=true npm run lint

# Expected output
> frontend@1.0.0 lint
> eslint .


/home/kirby/udacity/ci-cd/project/solution/frontend/src/components/MovieDetails.js
  4:24  error  'movie' is missing in props validation     react/prop-types
  7:70  error  'movie.id' is missing in props validation  react/prop-types

✖ 2 problems (2 errors, 0 warnings)
Build and run
For local development without docker, the developers use the following commands:

cd starter/frontend

# Install dependencies
npm ci

# Run local development server with hot reloading and point to the backend default
REACT_APP_MOVIE_API_URL=http://localhost:5000 npm start
To build the frontend application for a production deployment, they use the following commands:

# Build the image
# NOTE: Make sure the image is built with the URL of the backend system.
# The URL below would be the default backend URL when running locally
docker build --build-arg=REACT_APP_MOVIE_API_URL=http://localhost:5000 --tag=mp-frontend:latest .

docker run --name mp-frontend -p 3000:3000 -d mp-frontend]

# Open the browser to localhost:3000 and you should see the list of movies,
# provided the backend is already running and available on localhost:5000
Deploy Kubernetes manifests
In order to build the Kubernetes manifests correctly, the team uses kustomize in the following way:

cd starter/frontend/k8s
# Make sure you're kubeconfig is configured for the EKS cluster, i.e.
# aws eks update-kubeconfig

# Set the image tag to the newer version
# ℹ️ Don't commit any changes to the manifests that this command introduces
kustomize edit set image frontend=<ECR_REPO_URL>:<NEW_TAG_HERE>

# Apply the manifests to the cluster
kustomize build | kubectl apply -f -
Backend Development notes
Running tests
While in the backend directory, perform the following steps:

# Install dependencies
pipenv install

# Run the tests
pipenv run test

# Expected output
================================================================== test session starts ==================================================================
platform linux -- Python 3.10.6, pytest-7.2.1, pluggy-1.0.0 -- /home/kirby/.local/share/virtualenvs/backend-AXGg_iGk/bin/python
cachedir: .pytest_cache
rootdir: /home/kirby/udacity/cd12354-build-ci-cd-pipelines-monitoring-and-logging/project/solution/backend
collected 3 items

test_app.py::test_movies_endpoint_returns_200 PASSED                                                                                              [ 33%]
test_app.py::test_movies_endpoint_returns_json PASSED                                                                                             [ 66%]
test_app.py::test_movies_endpoint_returns_valid_data PASSED                                                                                       [100%]
To simulate failing the backend tests, run the following command:

FAIL_TEST=true pipenv run test

# Expected output
==================================================================== test session starts ====================================================================
platform linux -- Python 3.10.6, pytest-7.2.1, pluggy-1.0.0 -- /home/kirby/.local/share/virtualenvs/backend-AXGg_iGk/bin/python
cachedir: .pytest_cache
rootdir: /home/kirby/udacity/ci-cd/project/solution/backend
collected 3 items

test_app.py::test_movies_endpoint_returns_200 FAILED                                                                                                  [ 33%]
test_app.py::test_movies_endpoint_returns_json PASSED                                                                                                 [ 66%]
test_app.py::test_movies_endpoint_returns_valid_data PASSED                                                                                           [100%]

========================================================================= FAILURES ==========================================================================
_____________________________________________________________ test_movies_endpoint_returns_200 ______________________________________________________________

    def test_movies_endpoint_returns_200():
        with app.test_client() as client:
            status_code = os.getenv("FAIL_TEST", 200)
            response = client.get("/movies/")
>           assert response.status_code == status_code
E           AssertionError: assert 200 == 'true'
E            +  where 200 = <WrapperTestResponse streamed [200 OK]>.status_code

test_app.py:9: AssertionError
================================================================== short test summary info ==================================================================
FAILED test_app.py::test_movies_endpoint_returns_200 - AssertionError: assert 200 == 'true'
================================================================ 1 failed, 2 passed in 0.11s ================================================================
Running linter
When there are no linting errors, there won't be any output.

pipenv run lint
# No output
To simulate linting errors, you can run the linting command below. The command overrides our lint configuration and will error if any lines are over 88 characters.

pipenv run lint-fail

# Expected output
./movies/__init__.py:7:89: E501 line too long (120 > 88 characters)
./movies/__init__.py:9:89: E501 line too long (101 > 88 characters)
./movies/movies_api.py:7:89: E501 line too long (120 > 88 characters)
./movies/movies_api.py:9:89: E501 line too long (101 > 88 characters)
./movies/resources.py:16:89: E501 line too long (117 > 88 characters)
Build and run
For local development without docker, the developers use the following commands to build and run the backend application:

cd starter/backend

# Install dependencies
pipenv install

# Run application
pipenv run serve
For production deployments, the team uses the following commands to build and run the Docker image.

cd starter/backend

# Build the image
docker build --tag mp-backend:latest .

# Run the image
docker run -p 5000:5000 --name mp-backend -d mp-backend

# Check the running application
curl http://localhost:5000/movies

# Review logs
docker logs -f mp-backend

# Expected output
{"movies":[{"id":"123","title":"Top Gun: Maverick"},{"id":"456","title":"Sonic the Hedgehog"},{"id":"789","title":"A Quiet Place"}]}

# Stop the application
docker stop
Deploy Kubernetes manifests
In order to build the Kubernetes manifests correctly, the team uses kustomize in the following way:

cd starter/backend/k8s
# Make sure you're kubeconfig is configured for the EKS cluster, i.e.
# aws eks update-kubeconfig

# Set the image tag to the newer version
# ℹ️ Don't commit any changes to the manifests that this command introduces
kustomize edit set image backend=<ECR_REPO_URL>:<NEW_TAG_HERE>

# Apply the manifests to the cluster
kustomize build | kubectl apply -f -
License
License

udacity-build-cicd-project
udacity-build-cicd-project
udacity-build-cicd-project

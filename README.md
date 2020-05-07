# FFC Docker Node

N.B. This repo has now been archived as its functionality now exists in other services

This repository contains Node parent Docker image source code for the Future Farming and Countryside programme.

Two parent images are created from this repository:

- ffc-node
- ffc-node-development

It is recommended that services use [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build) to produce production and development images, each extending the appropriate parent, from a single Dockerfile.

[Examples](./example) are provided to show how parent images can be extended for different types of services. These should be a good starting point for building Node services conforming to FFC standards.

## Versioning

Images are built from this repository by a Jenkins pipeline defined in `./Jenkinsfile`. The images are tagged according to the Dockerfile version and the version of Node on which the image is based. For example, for Dockerfile version `1.0.0` based on Node `12.16.0`, the built image would be tagged `1.0.0-node12.16.0`.

The versions are set as variables near the top of the `Jenkinsfile`. The `dockerfileVersion` should be incremented with each change to the `Dockerfile`, following Semver rules. The `nodeVersion` should be set to the desired version of Node.

## Example files

`Dockerfile.web` - This is an example web project, that requires a build step to create some static files that are used by the web front end.

`Dockerfile.service` - This is an example project that doesn't expose any external ports (a message based service). There is also no build step in this Dockerfile.

## License

THIS INFORMATION IS LICENSED UNDER THE CONDITIONS OF THE OPEN GOVERNMENT LICENCE found at:

<http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3>

The following attribution statement MUST be cited in your products and applications when using this information.

> Contains public sector information licensed under the Open Government license v3

### About the license

The Open Government Licence (OGL) was developed by the Controller of Her Majesty's Stationery Office (HMSO) to enable information providers in the public sector to license the use and re-use of their information under a common open licence.

It is designed to encourage use and re-use of information freely and flexibly, with only a few conditions.

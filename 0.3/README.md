# Nomad Unofficial Image Build

This is Nomad Image has been created by mirror the upcoming Official Consul
Docker Image. See https://github.com/hashicorp/docker-consul.

There are several pieces that are used to build this image:

* We start with an Alpine base image and add CA certificates in order to reach
  the HashiCorp releases server. These are useful to leave in the image so that
  the container can access Atlas features as well.
* Official HashiCorp builds of some base utilities are then included in the
  image by pulling a release of docker-base. This includes dumb-init and gosu.
  See https://github.com/hashicorp/docker-base for more details.
* Finally a specific Nomad build is fetched and the rest of the Nomad-specific
  configuration happens according to the Dockerfile.

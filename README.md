# Evochain Container

## Description

This repository contains a Dockerfile and a few local tarballs for generating a
Docker image containing everything needed to link together an evolving EIC
simulation and reconstruction chain.  This chain evolves from the FPaDSim chain
towards the long-term goals of Argonne EIC software efforts.

## Usage

On a machine/user that has Docker properly installed, run

$ docker build -t evochain Docker

, where the evochain tag may be replaced with the user's choice.


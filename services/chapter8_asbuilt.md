# Chapter 6 AsBuilt

## Overview
As-Built data refers to information about a work site that varies from location to location as well as varying over time. The most basic form is a two-dimensional array of heights (a height-field) describing the surface of the work site, but it can include other data such as compaction information or the number of passes a machine has made over each point.

## Behind the Scenes
As-Built Queries are a way to select and process As-Built data into a form that is useful for analysing the current state of a site or historical changes between points in time.

The As-Built Query engine takes a query in the form of a json expression tree, or AST (Abstract Syntax Tree) and produces a result, which may be an image, a point cloud, a csv, a json object etc (depending on the query).

The As-Built Query engine is implemented as a two-stage process. The first stage is an expression, which evalutes a function to produce a massive 2d array of "cells". This large 2d array of "cells" is then agreggated to produce the desired result format, which may be an image, a point cloud, histogram etc.

## Tiles & Cells

## Site Requirements

## Client Requirements

## Contributing AsBuilt Data

## Extracting AsBuilt Data in a Report

# Estimation Tool for Spatial Prediction Models

## Table of contents
- [Estimation Tool for Spatial Prediction Models](#estimation-tool-for-spatial-prediction-models)
  - [Table of contents](#table-of-contents)
  - [Authors](#authors)
  - [Abstract](#abstract)
  - [Area Of Applicability (AOA)](#area-of-applicability-aoa)
  - [Aim of the tool](#aim-of-the-tool)
  - [Target group](#target-group)
  - [How does the software work?](#how-does-the-software-work)
    - [Input](#input)
    - [Part 1: Satellite image generation (with R)](#part-1-satellite-image-generation-with-r)
      - [Generation of a Sentinel-2 satellite image for the area of interest (Sentinel Image (AOI))](#generation-of-a-sentinel-2-satellite-image-for-the-area-of-interest-sentinel-image-aoi)
      - [Generation of a Sentinel-2 satellite image for the areas where the training data is located (Sentinel Image (training area))](#generation-of-a-sentinel-2-satellite-image-for-the-areas-where-the-training-data-is-located-sentinel-image-training-area)
    - [Part 2: Model training (with R)](#part-2-model-training-with-r)
    - [Part 3: Prediction and AOA (with R)](#part-3-prediction-and-aoa-with-r)
  - [How to install and run the app](#how-to-install-and-run-the-app)
  - [How to use the app](#how-to-use-the-app)
    - [Main Tool](#main-tool)
    - [Demo](#demo)
    - [Output of the results](#output-of-the-results)
  - [How to test](#how-to-test)
  - [Dependencies](#dependencies)
    - [Frontend](#frontend)
    - [Backend](#backend)
  - [Further Documentation](#further-documentation)
    - [Frontend](#frontend-1)
    - [Backend](#backend-1)
  - [Credits](#credits)
  - [License](#license)

## Authors
Project of the course Geosoftware 2 at the [Institute of Geoinformatics](https://www.uni-muenster.de/Geoinformatics/en/) by [Jakob Danel](https://github.com/jakobdanel), 
[Fabian Schumacher](https://github.com/fab-scm), 
[Thalis Goldschmidt](https://github.com/thalisgold), 
[Henning Sander](https://github.com/Hes097) and 
[Frederick Bruch](https://github.com/fbruc03) 

## Abstract
Machine learning methods have become very popular for spatial prediction efforts such as classifying remote sensing images, especially because of their ability to learn non-linear relationships and thereby solve more complex classifications tasks. A underestimated issue is that machine learning algorithms can only provide meaningful predictions when applied to data that is similar to the data they were trained on (Meyer and Pebesma, 2021). ”Similar” here refers to the value ranges of the predictor variables (such as different bands of the remote sensing image). When applying a trained machine learning algorithm to a new geographic area, it is unclear whether or not the pixels properties in that area are similar enough to the training data to enable a reliable classification.  

## Area Of Applicability (AOA)
The [Area Of Applicability](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.13650) is a method developed by Meyer and Pebesma (2021) to delineate areas in spatial data (here remote sensing images) that can be assumed to be areas the machine learning model can reliably be applied to. The AOA provides important additional information that should be communicated when applying machine learning methods to spatial prediction tasks, especially when predicting on a large or even global scale when training data are not evenly distributed over the target area. 

## Aim of the tool
The tool combines all the steps needed to perform a land use/land cover classification (generation of satellite images, model training and prediction). In particular, it is designed to extend the previous steps by the AOA and adopt this method into the typical workflow of a remote scientist/researcher without having to deal with its concrete implementation. Besides delineating such an area of applicability (AOA), this tool can also be used to point to areas where collecting additional training data is needed to train a more applicable model. 

## Target group
Researchers and users of remote sensing methods who want to
* use machine learning for land use classifications
* work with sentinel-2 data
* know how to train and apply machine learning models, but are unable or unwilling to focus on understanding and implementing the Area of Applicability
* work with large-scale mapping/modeling applications, but lack the necessary hardware to perform machine learning and compute the AOA on large datasets

## How does the software work?
The user has the possibility to select a model to work with. He can either upload his own model via an upload button or create a new model in order to train it with a selectable machine-learning algorithm. Depending on his choice, only specific parts of the software will be executed.

### Input
* Area of interest: The area for which the land use classification and the aoa are to be calculated.
* Training data or model: If a new model should be created, training data must be uploaded. Otherwise a model is uploaded by the user.
* Machine learning algorithm and hyperparameters: The new model must be trained. For this, the user can choose between two machine-learning algorithms and, if desired, also pass hyperparameters.
* Time period: In that period, a search is made for available sentinel-2 images.
* Bands/ predictors: All bands/predictors to be included in the sentinel images.
* Resolution: Resolution of the sentinel images to be generated.
* Maximum cloud cover: The satellite imagery search is filtered by maximum cloud cover.

### Part 1: Satellite image generation (with R)
#### Generation of a Sentinel-2 satellite image for the area of interest (Sentinel Image (AOI))
* Based on the user inputs (area of interest (AOI) , time period and cloud cover), the Spatial Temporal Asset Catalog (STAC) is searched for matching Sentinel-2 satellite images.
* For each Sentinel-2 image found, all bands (except ```B10```) are available for download. We only continue to work with those that have been pre-selected by the user. 
* If many images are found, we limit ourselves to 200 for further calculation.
* All images (max 200) are now superimposed and for each pixel the median is calculated over all images for each band.
* This can be helpful to avoid the problem of cloud cover and other interfering factors. In other words, the more images that can be found, the more likely it is to get a good image for model training and LULC classification.

#### Generation of a Sentinel-2 satellite image for the areas where the training data is located (Sentinel Image (training area))
* The generation of a Sentinel-2 satellite image for the areas where the training data is located is only done if the user chose to create a new model and therefore has uploaded training data.
* It works analogously to the generation of the Sentinel-2 image for the AOI. Instead of filtering by the AOI, it filters by the geometry of the training polygons. Pixels outside the polygons are set to NA.

### Part 2: Model training (with R)
If the user selects to work with his own model, no further model training is needed. If the user selects to create a new model, some additional steps must be performed to obtain valid training data. The generated sentinel image of the training areas (consisting of all selected bands) is now combined with the information from the uploaded training data. Each pixel completely covered by a training polygon is assigned the class of the polygon. As a result we get a dataset of all overlaid pixels, their assigned class and spectral information that we can now use to generate the model. 

The user can choose whether he wants to train the model with an random forest or with a support vector machine. For both, hyperparameters can be set and the model is validated with a spatial cross validation method, omitting whole training polygons.

### Part 3: Prediction and AOA (with R)
With the help of the trained model and the generated sentinel image for the AOI, a prediction is now calculated. In order to be able to make statements about the applicability of the model especially on unknown areas, the AOA is computed. In the areas where the model is not applicable according to the AOA, random points are generated that are suggested to the user as potential new locations for generating new training data. If this data is acquired in these areas and incorporated into the model, better results could be obtained.

## How to install and run the app

To make it as simple as possible we used [Docker](https://www.docker.com) for the development. The only thing necessary to run this software, is to download this repository with 'git clone --recursive https://github.com/geo-tech-project/geotech.git' and then run `docker-compsoe up --build`in the command line interface. This installs all dependencies for the front- and backend including all [R](https://www.r-project.org) packages. As these packages are not that small, this step could take up to one hour of building time (depending on your hardware). After building, the application will start automatically and you can access the webtool at `http://localhost:8780`. If you have terminated the application and want to restart it another time you can just leave out the `--build` tag of the `docker-compose up` command to start the app again.  


## How to use the app

### Main Tool
The main tool is designed in such a way that the user can use it very easily. The user is guided step by step and can only proceed to the next step if the previous one has been carried out correctly. For each step there is an additional info button that displays important information as soon as you hover over it. When everything has been entered successfully, the calculations can be started. After the calculations have been executed and no errors have occurred, the user will be directed to the results page.
![Main Tool page](https://github.com/geo-tech-project/frontend/raw/main/src/assets/main-tool.jpg)

### Demo
The demo page is structured exactly like the actual tool. However, all inputs have already been entered with default values. The user can view these entries, but not change them. He is only able to start the calculations by clicking on the ```Run demo``` button. The user should be redirected to the results page in less than 20 seconds.
![Demo page](https://github.com/geo-tech-project/frontend/raw/main/src/assets/demo.jpg)

### Output of the results
On a new route, the following three results are visualised on a map: 
* Prediction: Land use/ land cover classification
* Area of Applicabilty (AOA)
* Further train areas

It is possible to show and hide the individual results using a checkbox and even to adjust their transparency. The underlying satellite images on which the calculations are based are not displayed on the map but can be downloaded in the same way as the other results via a download button. Please note that the sentinel image of the training areas can only be downloaded if training data has been submitted.
![Result page](https://github.com/geo-tech-project/frontend/raw/main/src/assets/results_complete.jpg)



## How to test
To test this app you can proceed as follows:    
**Backend:**  
With your CLI go into your `backend` folder and run `npm test`.  
**Frontend:**  
**R:**  
The tests are written in the R package [testthat](https://testthat.r-lib.org/). 
Requirements:
- Installation of R
- Installation all R packages used in this project
- Installation of Node.js

Proceed the following steps.
1. Make a clone of the backend repository
2. Navigate into the backend/test folder
3. Run node testR.js  

## Dependencies
The following packages are used in this project:
### Frontend

- [@angular/animations](https://ghub.io/@angular/animations): Angular - animations integration with web-animations
- [@angular/cdk](https://ghub.io/@angular/cdk): Angular Material Component Development Kit
- [@angular/common](https://ghub.io/@angular/common): Angular - commonly needed directives and services
- [@angular/compiler](https://ghub.io/@angular/compiler): Angular - the compiler library
- [@angular/core](https://ghub.io/@angular/core): Angular - the core framework
- [@angular/forms](https://ghub.io/@angular/forms): Angular - directives and services for creating forms
- [@angular/material](https://ghub.io/@angular/material): Angular Material
- [@angular/platform-browser](https://ghub.io/@angular/platform-browser): Angular - library for using Angular in a web browser
- [@angular/platform-browser-dynamic](https://ghub.io/@angular/platform-browser-dynamic): Angular - library for using Angular in a web browser with JIT compilation
- [@angular/router](https://ghub.io/@angular/router): Angular - the routing library
- [@asymmetrik/ngx-leaflet](https://ghub.io/@asymmetrik/ngx-leaflet): Angular.io components for Leaflet
- [@asymmetrik/ngx-leaflet-draw](https://ghub.io/@asymmetrik/ngx-leaflet-draw): Angular.io component for the draw plugin for Leaflet
- [@creativebulma/bulma-tooltip](https://ghub.io/@creativebulma/bulma-tooltip): Display a tooltip attached to any kind of element, in different position.
- [@fortawesome/angular-fontawesome](https://ghub.io/@fortawesome/angular-fontawesome): Angular Fontawesome, an Angular library
- [@fortawesome/fontawesome-svg-core](https://ghub.io/@fortawesome/fontawesome-svg-core): The iconic font, CSS, and SVG framework
- [@fortawesome/free-brands-svg-icons](https://ghub.io/@fortawesome/free-brands-svg-icons): The iconic font, CSS, and SVG framework
- [@fortawesome/free-regular-svg-icons](https://ghub.io/@fortawesome/free-regular-svg-icons): The iconic font, CSS, and SVG framework
- [@fortawesome/free-solid-svg-icons](https://ghub.io/@fortawesome/free-solid-svg-icons): The iconic font, CSS, and SVG framework
- [ace-builds](https://ghub.io/ace-builds): Ace (Ajax.org Cloud9 Editor)
- [better-docs](https://ghub.io/better-docs): JSdoc theme
- [bootstrap](https://ghub.io/bootstrap): The most popular front-end framework for developing responsive, mobile first projects on the web.
- [bulma](https://ghub.io/bulma): Modern CSS framework based on Flexbox
- [bulma-slider](https://ghub.io/bulma-slider): Display classic slider more sexy, in different colors, sizes, and states 
- [bulma-toast](https://ghub.io/bulma-toast): Bulma&#39;s pure JavaScript extension to display toasts
- [chroma-js](https://ghub.io/chroma-js): JavaScript library for color conversions
- [font-awesome](https://ghub.io/font-awesome): The iconic font and CSS framework
- [geoblaze](https://ghub.io/geoblaze): Blazing Fast JavaScript Raster Processing Engine
- [georaster](https://ghub.io/georaster): Wrapper around Georeferenced Rasters like GeoTIFF, NetCDF, JPG, and PNG that provides a standard interface
- [georaster-layer-for-leaflet](https://ghub.io/georaster-layer-for-leaflet): Display GeoTIFFs and soon other types of raster on your Leaflet Map
- [leaflet](https://ghub.io/leaflet): JavaScript library for mobile-friendly interactive maps
- [leaflet-draw](https://ghub.io/leaflet-draw): Vector drawing plugin for Leaflet
- [leaflet-geotiff](https://ghub.io/leaflet-geotiff): A LeafletJS plugin for displaying geoTIFF raster data.
- [leaflet-geotiff-2](https://ghub.io/leaflet-geotiff-2): A LeafletJS plugin for displaying geoTIFF raster data.
- [ng2-file-upload](https://ghub.io/ng2-file-upload): 
- [ngx-markdown](https://ghub.io/ngx-markdown): Angular library that uses marked to parse markdown to html combined with Prism.js for synthax highlights
- [rxjs](https://ghub.io/rxjs): Reactive Extensions for modern JavaScript
- [tslib](https://ghub.io/tslib): Runtime library for TypeScript helper functions
- [typedoc](https://ghub.io/typedoc): Create api documentation for TypeScript projects.
- [zone.js](https://ghub.io/zone.js): Zones for JavaScript

#### Dev Dependencies

- [@angular-devkit/build-angular](https://ghub.io/@angular-devkit/build-angular): Angular Webpack Build Facade
- [@angular/cli](https://ghub.io/@angular/cli): CLI tool for Angular
- [@angular/compiler-cli](https://ghub.io/@angular/compiler-cli): Angular - the compiler CLI for Node.js
- [@types/jasmine](https://ghub.io/@types/jasmine): TypeScript definitions for Jasmine
- [@types/leaflet](https://ghub.io/@types/leaflet): TypeScript definitions for Leaflet.js
- [@types/leaflet-draw](https://ghub.io/@types/leaflet-draw): TypeScript definitions for leaflet-draw
- [@types/node](https://ghub.io/@types/node): TypeScript definitions for Node.js
- [jasmine-core](https://ghub.io/jasmine-core): Official packaging of Jasmine&#39;s core files for use by Node.js projects.
- [karma](https://ghub.io/karma): Spectacular Test Runner for JavaScript.
- [karma-chrome-launcher](https://ghub.io/karma-chrome-launcher): A Karma plugin. Launcher for Chrome and Chrome Canary.
- [karma-coverage](https://ghub.io/karma-coverage): A Karma plugin. Generate code coverage.
- [karma-jasmine](https://ghub.io/karma-jasmine): A Karma plugin - adapter for Jasmine testing framework.
- [karma-jasmine-html-reporter](https://ghub.io/karma-jasmine-html-reporter): A Karma plugin. Dynamically displays tests results at debug.html page
- [typescript](https://ghub.io/typescript): TypeScript is a language for application scale JavaScript development

### Backend

- [axios](https://ghub.io/axios): Promise based HTTP client for the browser and node.js
- [body-parser](https://ghub.io/body-parser): Node.js body parsing middleware
- [chai](https://ghub.io/chai): BDD/TDD assertion library for node.js and the browser. Test framework agnostic.
- [cors](https://ghub.io/cors): Node.js CORS middleware
- [dotenv](https://ghub.io/dotenv): Loads environment variables from .env file
- [express](https://ghub.io/express): Fast, unopinionated, minimalist web framework
- [mocha](https://ghub.io/mocha): simple, flexible, fun test framework
- [multer](https://ghub.io/multer): Middleware for handling `multipart/form-data`.
- [ng2-file-upload](https://ghub.io/ng2-file-upload): extension for multer to upload files to the server
- [nodemon](https://ghub.io/nodemon): Simple monitor script for use during development of a node.js app.
- [r-integration](https://ghub.io/r-integration): Simple portable library used to interact with pre-installed R compiler by running commands or scripts(files)
- [supertest](https://ghub.io/supertest): SuperAgent driven library for testing HTTP servers
- [swagger-ui-express](https://ghub.io/swagger-ui-express): Swagger UI Express


### R
- [terra](https://cran.r-project.org/web/packages/terra/index.html): Spatial Data Analysis
- [rgdal](https://cran.r-project.org/web/packages/rgdal/index.html): Bindings for the 'Geospatial' Data Abstraction Library
- [rgeos](https://cran.r-project.org/web/packages/rgeos/index.html): Interface to Geometry Engine - Open Source ('GEOS')
- [rstac](https://cran.r-project.org/web/packages/rstac/index.html): Client Library for SpatioTemporal Asset Catalog
- [gdalcubes](https://cran.r-project.org/web/packages/gdalcubes/index.html): Earth Observation Data Cubes from Satellite Image Collections
- [raster](https://cran.r-project.org/web/packages/raster/index.html): Geographic Data Analysis and Modeling
- [caret](https://cran.r-project.org/web/packages/caret/index.html): Classification and Regression Training
- [CAST](https://cran.r-project.org/web/packages/CAST/index.html): 'caret' Applications for Spatial-Temporal Models
- [lattice](https://cran.r-project.org/web/packages/lattice/index.html): Trellis Graphics for R
- [Orcs](https://cran.r-project.org/web/packages/Orcs/index.html): Omnidirectional R Code Snippets
- [jsonlite](https://cran.r-project.org/web/packages/jsonlite/index.html): A Simple and Robust JSON Parser and Generator for R
- [tmap](https://cran.r-project.org/web/packages/tmap/index.html): Thematic Maps
- [latticeExtra](https://cran.r-project.org/web/packages/latticeExtra/index.html): Extra Graphical Utilities Based on Lattice
- [doParallel](https://cran.r-project.org/web/packages/doParallel/index.html): Foreach Parallel Adaptor for the 'parallel' Package
- parallel
- [sp](https://cran.r-project.org/web/packages/sp/index.html): Classes and Methods for Spatial Data
- [geojson](https://cran.r-project.org/web/packages/geojson/index.html): Classes for 'GeoJSON'
- [rjson](https://cran.r-project.org/web/packages/rjson/index.html): JSON for R
- [randomForest](https://cran.r-project.org/web/packages/randomForest/index.html): Breiman and Cutler's Random Forests for Classification and Regression


## Further Documentation

The software can be split into two essential parts. The frontend was developed with the web framework [Angular](https://angular.io).
The backend is setup as a Node.js application using the [Express](https://expressjs.com/) framework. 

### Frontend
Documentation of the frontend written in Angular, with HTML, CSS and TypeScript: [Frontend](http://35.80.3.64:8781/frontend)

### Backend
The backend can be devided into three parts. The first part are the R scripts that are used to perform the actual operations, e.g. generating the sentinel images or calculating the AOA. The second part is the API that establishes the connection between the back- and frontend. The last part is the Javascript code that sets up the API and connects to the R-part. Please note that the following links can only be used from the internet network of the University of Münster.
- [R-Scripts](http://35.80.3.64:8781/R)
- [API](http://35.80.3.64/documentation)
- [Javascript](http://35.80.3.64/js)

## Credits
Credits

## License
Add license text here



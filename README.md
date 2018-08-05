# Transparent Psi Replication Study

## Overview

This is an application that supports psychological studies with data protection, where no data can be removed or changed from the database to modify the output of the study result. Data are collected via a central data collection framework implemented in JavaScript using the node.js framework. This serves the study code to clients, collects responses, and forwards the data to a remote git repository for archival. It is built fully from open source components, and the code will be provided for review at the end of the study. For security reasons, the application updates it's own source code from the git repository's remote master branch in every 5 minutes.

- **The frontend** handles user interactions (via HTML event listeners) like: inputs, selectable answers and key controls. It gets the available languages from the backend and loads the selected one. After every user interaction it sends a push request to the backend with a user data [wrapper object](wrapper_object.md).

- **The backend** fetches the available languages from a [github repository](https://github.com/gy0p4k/transparent-psi-languages) and updates the fronted with it. It also accepts the endpoint calls from the frontend, store the user object in a local git repository and pushes it to github realtime. In github all the rows are public and stored in a [csv file](https://github.com/gy0p4k/transparent-psi-results/blob/master/results.csv).

## Technologies, resources

### Frontend

- The frontend is served with [`http-server`](https://github.com/indexzero/http-server)
- The static site is a [single html page](frontend/index.html) with a container object that contains the rendered screens of the application
- The [main script](frontend/script.js) that manage the simulation rendering, it is written in [EcmaScript 6](http://es6-features.org/)
- The [backend adapter package](frontend/backend-adapter.js) contains the methods that can communicate with the backend and
    - initialize the connection (generate the `ajax` request functions and create the user data wrapper object)
    - get available languages
    - get the selected language package

##### Deployment process:

- with `package.json` 
```shell
$ cd frontend/
$ npm i
$ npm start
```

- manually
```shell
$ cd frontend/
$ npm i -g http-server
$ http-server . 
```

- with [`pm2`](http://pm2.keymetrics.io/)
```shell
$ cd frontend/
$ pm2 start frontend.sh
```

### Backend

##### Packages

- [`random.js`](backend/src/random.js) handles randomization based on `npm` package [alea](https://www.npmjs.com/package/alea) to generate pseudo-random objects 
    -  `generate()` returns a pseudo-random value between `0` and `1`
    -  `conflip()` returns a pseudo-random boolean value
    -  `shuffle()` is a custom-implemented [Fischer-Yates algorithm](https://bost.ocks.org/mike/shuffle/) for array shuffling
- [`save.js`](backend/src/save.js) handles the saving and uploading process. The used dependencies are [`simple-git`](https://www.npmjs.com/package/simple-git) and [`csv-writer`](https://www.npmjs.com/package/csv-writer)
    - `save()` start the csv writing process and the remote repository synchronization
    - `csvProdWriter` and `csvTestWriter` are writer objects that can add new csv lines to the result file
    - `gitPush()` is create a secure `ssh` connection with the remote server and synchronize the results with it via `git`
- [`user.js`](backend/src/user.js) serve user informations to the frontend (resource urls, language packages and template user objects)
- [`shell.js`](backend/src/shell.js) start `python` shell jobs for language synchronization. It returns an EcmaScript6 Promise object

##### Endpoints

- `[GET] /` returns a new user object that contains a user id based on [`uuid`](https://www.npmjs.com/package/uuid)    
    - example response:
    
        ```
        {
            "id":"1960c7da-609d-48f3-a8d0-7e9c45cd7196",
            "random":0.8826724286191165,
            "side":"left"
        }
        ```

- `[GET] /ping/:erotic/:nonErotic` returns a new user object that contains a user id and a random number between 0 and [erotic] or [non-erotic]   
    - example response:
    
        ```
        {
            "id":"1960c7da-609d-48f3-a8d0-7e9c45cd7196",
            "picType": "nonErotic",
            "picNumber": 6
        }
        ```

- `[POST] /` receive a post request with the actual user data and start the saving job
    - example request body based on the [wrapper object](wrapper_object.md)
    - example response is the same as request, so the frontend can check if the save was successful and the  request data is untouched
- `[GET] /pic/:type` returns a randoom shuffled picture array that contains the names of the resource pictures based on the user's request
    - available types:
        - `ff`
        - `fm`
        - `mm`
        - `mf`
    - example response:

        ```
        {
            "status": "ok",
                "urls": [
                    "bern206.jpg",
                    "2635.jpg",
                    "bern205.jpg",
                    ...
                ]
        }
        ```

- `[GET] /langs` returns the available languages    
    - example response:
    
        ```
        {
            "status": "ok",
            "langs": [
                "English",
                "Magyar",
                ...
                ]
        }
        ```

- `[GET] /lang/:lang` returns the selected language package
    - example response:
    
        ```
        {
            "status": "ok",
            "lang": {
                "headerText": "Transparent Psi Project",
                "otherTexts": "example"
            }
        }
        ```

- `[GET] /checkId/:labId/:expId` returns if the given laboratory id and experimenter id is correct (is in the database)
    - example response:
    
        ```
        {
            "status" :"ok",
            "valid":
                {
                    "labId":"labIdTest",
                    "expId":"expIdTest",
                    "labScore":"2",
                    "expScore":"1"
                }
        }
        ```

##### Deployment

Run the server:
```shell
$ cd backend/
$ npm i
$ npm start
```

Run the tests:
```shell
$ cd backend/
$ npm i
$ npm test
```
